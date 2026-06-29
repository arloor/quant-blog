---
title: "tick-stream：A股 tick 异常检测与飞书告警规则整理"
date: 2026-06-29T11:00:00+08:00
lastmod: 2026-06-29T11:00:00+08:00
draft: false
categories: []
tags: ["A股", "tick", "量化监控", "飞书告警"]
description: "整理 tick-stream 项目中价格跳变、动量异常、盘口流动性异常的检测逻辑，以及候选异常如何过滤、聚合、冷却并发送飞书通知。"
keywords:
- A股
- tick
- 异常检测
- 动量Z分数
- 飞书告警
---

这篇文章整理 `tick-stream` 当前版本的异常检测规则和告警发送规则。它不是交易策略，也不会自动下单；它更像一个实时监控器：订阅标的池 tick，识别短时间内的价格、动量、盘口异常，再把真正值得看的信号合并成飞书通知。

<!--more-->

> 仅作为个人技术记录和复盘材料，不构成任何投资建议。

## 整体处理链路

实时运行时，每一条 tick 会走下面这条链路：

1. 标准化 tick 字段，包括标的代码、事件时间、最新价、成交量、成交额、盘口五档。
2. 做质量过滤，剔除无效、重复、乱序、非标的池内、忽略时段内的 tick。
3. 把通过过滤的 tick 放入每个标的自己的滚动窗口。
4. 计算特征快照，包括短窗涨跌幅、动量速度、成交活跃度、盘口失衡、撤挂压力等。
5. 分别运行价格跳变、动量异常、盘口流动性异常三个检测器。
6. 把检测出来的候选异常再过一层“可报告性”过滤，避免噪音直接发通知。
7. 对同一标的在聚合窗口内的多个异常做合并。
8. 经过冷却和反向震荡保护后，构造飞书 `post` 消息并发送。
9. 全程写 JSONL 审计日志，方便回放和复盘。

换句话说，项目里有两个层次的规则：

- 检测规则：判断“这里是否有异常候选”。
- 告警规则：判断“这个异常候选是否值得发到飞书”。

这个分层很重要。检测层可以更敏感，尽量记录现场；告警层则更克制，避免把普通盘口抖动和微小价格变化推给人。

## Tick 质量过滤

进入检测前，系统先判断 tick 是否可用。

会被拒绝或忽略的情况包括：

| 状态 | 规则 |
| --- | --- |
| malformed | 缺少标的代码、事件时间，或者最新价为空、非正数 |
| ignored | 标的不在 active watchlist 中 |
| duplicate | 同一标的事件时间等于最近已接受 tick |
| out_of_order | 同一标的事件时间早于最近已接受 tick |
| ignored session | 命中规则配置里的 `ignored_sessions` |

`ignored_sessions` 主要用来避开开盘集合竞价、临近收盘等容易产生结构性噪音的时间段。比如本地配置里常见的写法是：

```yaml
ignored_sessions:
- start: 09:25:00
  end: 09:35:00
- start: "14:55:00"
  end: "15:00:00"
```

只有 `accepted` 的 tick 才会进入后续窗口和检测器。

## 特征快照

检测器依赖一组中间特征。它们不是最终告警，而是给价格、动量、盘口判断提供上下文。

| 特征 | 含义 |
| --- | --- |
| `price_return_short_pct` | 最近 `price_window_seconds` 窗口内，从窗口首价到最新价的涨跌幅 |
| `realized_volatility_ratio` | 短窗收益波动相对基准窗口收益波动的比例 |
| `volume_burst_ratio` | 短窗成交量或成交额均值，相对基准窗口均值的比例 |
| `queue_imbalance_ratio` | 买卖盘总挂量失衡程度，公式为 `abs(total_bid - total_ask) / (total_bid + total_ask)` |
| `cancel_add_ratio` | 撤挂压力，公式为 `total_cancel / max(total_add + total_cancel, 1)` |
| `order_flow_imbalance` | 盘口净压力，买盘新增和卖盘撤单偏向向上，卖盘新增和买盘撤单偏向向下 |
| `spread_ratio` | 一档卖价减一档买价，再除以最新价 |
| `depth_collapse_ratio` | 当前盘口深度相对近期平均深度的收缩比例 |

如果 tick 没有盘口字段，价格和动量仍会继续检测，但盘口相关特征会标记为 `missing_data`。

## 数学符号约定

后面解释公式时，用下面这套符号：

| 符号 | 含义 |
| --- | --- |
| `t_i` | 第 `i` 条已接受 tick 的事件时间 |
| `p_i` | 第 `i` 条已接受 tick 的最新价 |
| `v_i` | 第 `i` 条 tick 的成交量。如果没有成交量，则用成交额 |
| `B_i` | 第 `i` 条 tick 的买盘五档挂量合计 |
| `A_i` | 第 `i` 条 tick 的卖盘五档挂量合计 |
| `Delta t_i` | 相邻 tick 的时间差，代码里用 `max(t_i - t_{i-1}, 1秒)` 防止除以 0 |
| `return(a, b)` | 从价格 `a` 到价格 `b` 的涨跌幅，单位是百分比 |

项目里的涨跌幅单位是“百分比点”，不是小数。也就是说，涨 `1%` 在计算里是 `1.0`，不是 `0.01`。

基础涨跌幅函数是：

```text
return(a, b) = (b - a) / a * 100
```

如果 `a = 10.00`，`b = 10.12`，则：

```text
return(10.00, 10.12) = (10.12 - 10.00) / 10.00 * 100 = 1.20
```

意思是涨了 `1.20%`。

## 价格跳变规则

价格跳变是最直接的一类异常。

系统取某个标的最近 `price_window_seconds` 秒内的 tick，构成短窗口：

```text
W_price = {tick_i | latest_time - t_i <= price_window_seconds}
```

要求这个窗口里的 tick 数量至少达到：

```text
len(W_price) >= min_ticks_short_window
```

如果数量不够，说明短窗样本太薄，直接不判断价格跳变。

设窗口第一条 tick 的价格为 `p_first`，最新 tick 的价格为 `p_latest`，则短窗涨跌幅为：

```text
price_return_pct = return(p_first, p_latest)
                 = (p_latest - p_first) / p_first * 100
```

当满足下面条件时生成 `price_jump` 事件：

```text
abs(price_return_pct) >= price_return_threshold_pct
```

这里取绝对值，所以向上跳变和向下跳变都会被检测。

方向由涨跌幅正负决定：

- 大于 0：`up`
- 小于 0：`down`

严重程度由 `severity_thresholds` 中的价格阈值决定：

| 等级 | 判断方式 |
| --- | --- |
| warning | 达到基础 `price_return_threshold_pct`，但未达到 high |
| high | `abs(price_return_pct)` 达到 `severity_thresholds.high.price_return_pct` |
| critical | `abs(price_return_pct)` 达到 `severity_thresholds.critical.price_return_pct` |

价格跳变一旦被检测出来，默认就是可报告事件。也就是说，它不需要额外成交量或盘口确认。

举个例子：

```text
price_window_seconds = 30
price_return_threshold_pct = 1.5
min_ticks_short_window = 3

30秒窗口内第一价 p_first = 10.00
最新价 p_latest = 10.18

price_return_pct = (10.18 - 10.00) / 10.00 * 100 = 1.80
abs(1.80) >= 1.5
```

这会触发一条向上的 `price_jump` 候选异常。

## 动量异常规则

动量异常检测的是“短时间价格变化速度相对过去是否异常”。它不只看涨跌幅，而是把短窗速度和基准窗口内的逐 tick 速度做 robust z-score 比较。

它解决的是这样一个问题：同样是 10 秒涨 `0.8%`，如果一只股票过去 3 分钟一直剧烈跳动，这不一定异常；但如果过去 3 分钟几乎横盘，这个加速度就很值得注意。

### 第一步：取脉冲窗口

系统取最近 `momentum_impulse_seconds` 秒作为脉冲窗口：

```text
W_impulse = {tick_i | latest_time - t_i <= momentum_impulse_seconds}
```

要求：

```text
len(W_impulse) >= min_ticks_short_window
```

否则不判断动量异常。

设脉冲窗口第一条价格为 `p_impulse_start`，最新价为 `p_latest`：

```text
impulse_return_pct = return(p_impulse_start, p_latest)
                   = (p_latest - p_impulse_start) / p_impulse_start * 100
```

窗口跨度秒数为：

```text
seconds = max(latest_time - impulse_start_time, 1秒)
```

脉冲速度为：

```text
velocity_pct_per_second = impulse_return_pct / seconds
```

这个速度单位是 `%/秒`。

例如：

```text
p_impulse_start = 10.00
p_latest = 10.08
seconds = 8

impulse_return_pct = (10.08 - 10.00) / 10.00 * 100 = 0.80
velocity_pct_per_second = 0.80 / 8 = 0.100
```

意思是最近 8 秒以平均 `0.100%/秒` 的速度上冲。

### 第二步：真实位移门槛

在算 z-score 前，先用 `momentum_min_return_pct` 做硬过滤：

```text
abs(impulse_return_pct) >= momentum_min_return_pct
```

这个参数是防噪音的关键。比如最近 10 秒只涨了 `0.10%`，即使历史波动极低导致 z-score 看起来很高，也不会进入动量异常。

换句话说，`momentum_min_return_pct` 回答的是：

```text
这次短窗运动本身够不够大？
```

而后面的 `momentum_z_threshold` 回答的是：

```text
这次短窗运动相对近期基准够不够异常？
```

两者必须同时看。只看 z-score，在横盘时容易被很小的价格跳动放大；只看涨跌幅，又看不出“相对最近是否突然加速”。

### 第三步：构造基准速度样本

系统取最近 `momentum_baseline_seconds` 秒作为基准窗口：

```text
W_baseline = {tick_i | latest_time - t_i <= momentum_baseline_seconds}
```

要求：

```text
len(W_baseline) >= min_ticks_baseline_window
```

然后对窗口里每一对相邻 tick 计算一个速度样本：

```text
r_i = return(p_{i-1}, p_i)
Delta t_i = max(t_i - t_{i-1}, 1秒)
baseline_velocity_i = r_i / Delta t_i
```

于是得到一个速度样本序列：

```text
S = [baseline_velocity_1, baseline_velocity_2, ..., baseline_velocity_n]
```

代码还会统计其中非零样本：

```text
S_nonzero = {x in S | abs(x) > 1e-9}
```

如果非零样本太少，说明最近行情太平，z-score 容易失真。

### 第四步：低波动保护

低波动保护规则是：

```text
如果 len(S_nonzero) < momentum_min_nonzero_baseline_samples
并且 abs(impulse_return_pct) < momentum_zero_mad_min_return_pct
则本次动量异常不成立
```

这条规则的含义是：如果基准窗口里大部分 tick 都没怎么动，那么必须看到更大的真实涨跌幅，才允许继续往下判断。

### 第五步：计算 robust z-score

普通 z-score 通常用均值和标准差：

```text
z = (value - mean) / std
```

但 tick 速度很容易有尖刺，用均值和标准差会被极端值带偏。这里改用中位数和 MAD：

```text
median = median(S)
mad = median(abs(x - median) for x in S)
momentum_z = 0.6745 * (velocity_pct_per_second - median) / mad
```

这里的 `MAD` 是 median absolute deviation，也就是“相对中位数的绝对偏差的中位数”。它比标准差更抗异常值。

`0.6745` 是一个尺度校正系数。在近似正态分布下：

```text
MAD ≈ 0.6745 * sigma
```

所以：

```text
0.6745 * (value - median) / MAD
```

可以把 MAD 尺度转换成类似普通 z-score 的尺度。

如果 `mad == 0`，说明基准窗口的速度样本相对中位数没有有效离散度。系统会走特殊逻辑：

```text
zero_mad_min_abs_value = momentum_zero_mad_min_return_pct / seconds

如果 abs(velocity_pct_per_second - median) >= zero_mad_min_abs_value
并且 velocity_pct_per_second != median
则 momentum_z = 99.0
否则 momentum_z = 0.0
```

这里的 `99.0` 不是精确统计意义上的 99 个标准差，而是一个工程上的“极端异常”标记：基准完全不动，而当前短窗已经达到足够真实位移。

### 第六步：动量异常触发

最终，当满足下面条件时生成 `momentum_spike` 候选事件：

```text
abs(momentum_z) >= momentum_z_threshold
```

严重程度同样由 `severity_thresholds` 决定：

| 等级 | 判断方式 |
| --- | --- |
| warning | 达到基础 `momentum_z_threshold`，但未达到 high |
| high | `abs(momentum_z)` 达到 `severity_thresholds.high.momentum_z` |
| critical | `abs(momentum_z)` 达到 `severity_thresholds.critical.momentum_z` |

这里的“动量Z分数=8.53”可以理解为：当前短窗价格变化速度，比近期基准速度的典型波动大约偏离了 `8.53` 个 robust 尺度。

一个可复算的简化例子：

```text
momentum_impulse_seconds = 10
momentum_min_return_pct = 0.45
momentum_z_threshold = 6.5

脉冲窗口：
p_impulse_start = 10.00
p_latest = 10.09
seconds = 9

impulse_return_pct = 0.90
velocity_pct_per_second = 0.90 / 9 = 0.1000

基准速度样本 S 的中位数 median = 0.0100
MAD = 0.0071

momentum_z = 0.6745 * (0.1000 - 0.0100) / 0.0071
           ≈ 8.55
```

因为：

```text
abs(impulse_return_pct) = 0.90 >= 0.45
abs(momentum_z) = 8.55 >= 6.5
```

所以会生成一条动量异常候选。

## 盘口流动性异常规则

盘口流动性异常关注的不是成交价本身，而是挂单结构是否出现持续压力。

系统会从 tick 中提取买一到买五、卖一到卖五。没有盘口字段时，盘口检测器不可用，但不会影响价格和动量检测。

### 第一步：计算五档总量

对每条 tick，先把五档买盘和五档卖盘数量分别加总：

```text
B_i = bid_qty_1 + bid_qty_2 + bid_qty_3 + bid_qty_4 + bid_qty_5
A_i = ask_qty_1 + ask_qty_2 + ask_qty_3 + ask_qty_4 + ask_qty_5
```

`B_i` 越大，代表买盘排队量越厚；`A_i` 越大，代表卖盘排队量越厚。

### 第二步：计算挂单变化

系统会和上一条有盘口的 tick 比较，估算买卖盘总挂量的增加和减少：

```text
bid_diff = B_i - B_{i-1}
ask_diff = A_i - A_{i-1}

bid_added = max(bid_diff, 0)
bid_cancelled = max(-bid_diff, 0)
ask_added = max(ask_diff, 0)
ask_cancelled = max(-ask_diff, 0)
```

然后合并成总新增和总撤减：

```text
total_add = bid_added + ask_added
total_cancel = bid_cancelled + ask_cancelled
```

这里不是精确逐笔委托还原，而是用五档总量变化做近似压力指标。它能捕捉盘口量突然消失、突然堆积、单侧明显倾斜等现象。

### 第三步：撤挂压力

撤挂压力用 `cancel_add_ratio` 表示：

```text
cancel_add_ratio = total_cancel / max(total_add + total_cancel, 1)
```

这个值在 `0` 到 `1` 附近：

- 接近 `0`：主要是挂单增加。
- 接近 `1`：主要是挂单减少或撤单压力。
- 中间值：新增和撤减混合。

代码里的飞书展示会乘以 `100`，例如 `0.65` 会显示成 `65.00%`。

### 第四步：买卖盘失衡

买卖盘失衡用 `queue_imbalance_ratio` 表示：

```text
queue_imbalance_ratio = abs(B_i - A_i) / (B_i + A_i)
```

它也大致在 `0` 到 `1` 之间：

- 接近 `0`：买卖盘总量比较均衡。
- 越接近 `1`：一侧挂量远大于另一侧。

这个指标只看失衡程度，不直接表示方向。方向另外用总买盘和总卖盘比较：

```text
如果 B_i >= A_i，则方向为 up
否则方向为 down
```

### 第五步：盘口净压力

告警正文里还会带一个 `order_flow_imbalance`，它的公式是：

```text
order_flow_imbalance =
    (bid_added + ask_cancelled)
  - (ask_added + bid_cancelled)
```

可以粗略理解为：

- `bid_added`：买盘增强，偏向向上。
- `ask_cancelled`：卖盘撤退，偏向向上。
- `ask_added`：卖盘增强，偏向向下。
- `bid_cancelled`：买盘撤退，偏向向下。

所以这个值为正时，盘口净压力偏向上；为负时，盘口净压力偏向下。

### 第六步：盘口信号命中

当满足任一条件时，认为当前 tick 出现盘口信号：

```text
cancel_add_ratio >= orderbook_cancel_ratio_threshold
或
queue_imbalance_ratio >= orderbook_imbalance_ratio_threshold
```

但单个 tick 的盘口变化很容易抖动，所以系统要求连续命中：

```text
连续命中次数 >= orderbook_min_consecutive_ticks
```

只要某条 tick 没命中盘口信号，连续计数就会清零。这个设计是为了过滤单条 tick 的盘口闪动。

达到连续次数后生成 `orderbook_liquidity` 事件。方向由买卖盘总量决定：

- `total_bid_quantity >= total_ask_quantity`：`up`
- 否则：`down`

严重程度由撤挂压力和买卖盘失衡两个维度共同决定：

| 等级 | 判断方式 |
| --- | --- |
| warning | 达到基础盘口阈值，但未达到 high |
| high | `cancel_add_ratio` 或 `queue_imbalance_ratio` 达到 high 对应阈值 |
| critical | `cancel_add_ratio` 或 `queue_imbalance_ratio` 达到 critical 对应阈值 |

举个例子：

```text
orderbook_cancel_ratio_threshold = 0.65
orderbook_imbalance_ratio_threshold = 0.92
orderbook_min_consecutive_ticks = 3

第1条：cancel_add_ratio = 0.70，queue_imbalance_ratio = 0.50，命中，连续计数=1
第2条：cancel_add_ratio = 0.68，queue_imbalance_ratio = 0.55，命中，连续计数=2
第3条：cancel_add_ratio = 0.72，queue_imbalance_ratio = 0.58，命中，连续计数=3
```

第三条 tick 才会生成盘口流动性异常候选。

### 成交活跃度公式

盘口和动量告警都可能用到成交活跃度确认，也就是 `volume_burst_ratio`。

系统先为每条 tick 取活动量：

```text
activity_i = volume_i if volume_i > 0 else turnover_i
```

然后计算：

```text
short_activity_mean = mean(activity_i in price short window)
baseline_activity_mean = mean(activity_i in momentum baseline window)

volume_burst_ratio = short_activity_mean / max(baseline_activity_mean, 1)
```

如果 `volume_burst_ratio = 2.5`，意思是短窗成交量或成交额均值大约是基准窗口均值的 `2.5` 倍。

## 候选异常如何变成可发送告警

检测器产出的事件还不是飞书消息。它们会先经过 reportable 过滤。

### 价格跳变

`price_jump` 永远可报告。

它是项目里最高优先级的直接信号：只要短窗涨跌幅达到阈值，就会进入告警聚合流程。

### 动量异常

`momentum_spike` 需要额外确认，满足任一条件即可报告：

| 条件 | 含义 |
| --- | --- |
| `abs(impulse_return_pct) >= momentum_notify_min_return_pct` | 脉冲窗口实际涨跌幅足够大 |
| `volume_burst_ratio >= momentum_notify_volume_burst_ratio` | 成交量或成交额明显放大 |
| 盘口压力 + 最低成交活跃度 | `queue_imbalance_ratio >= momentum_notify_orderbook_imbalance_ratio` 或 `cancel_add_ratio >= momentum_notify_cancel_add_ratio`，同时 `volume_burst_ratio >= momentum_notify_orderbook_min_volume_burst_ratio` |
| 同组存在价格跳变 | 同一个 tick 批次里有 `price_jump` 作为兄弟事件 |

这个设计的意思是：动量 Z 分数可以先捕捉异常加速度，但要发给人，最好再有真实位移、成交活跃或盘口压力做确认。

用公式写，就是：

```text
reportable(momentum) =
    abs(impulse_return_pct) >= momentum_notify_min_return_pct
 or volume_burst_ratio >= momentum_notify_volume_burst_ratio
 or (
      (
        queue_imbalance_ratio >= momentum_notify_orderbook_imbalance_ratio
        or cancel_add_ratio >= momentum_notify_cancel_add_ratio
      )
      and volume_burst_ratio >= momentum_notify_orderbook_min_volume_burst_ratio
    )
 or same_batch_has_price_jump
```

这里要注意两组动量参数的区别：

| 参数 | 所在阶段 | 含义 |
| --- | --- | --- |
| `momentum_min_return_pct` | 检测阶段 | 真实涨跌幅不够时，不生成动量候选 |
| `momentum_notify_min_return_pct` | 告警阶段 | 候选已经生成后，真实涨跌幅够大则允许直接发送 |

前者决定“算不算异常候选”，后者决定“要不要打扰人”。

### 盘口流动性异常

`orderbook_liquidity` 单独发通知更谨慎，必须先达到最低等级：

```text
event.severity >= orderbook_standalone_min_severity
```

达到最低等级后，还需要满足任一条件：

| 条件 | 含义 |
| --- | --- |
| 同组存在价格跳变 | 盘口异常和价格跳变共振 |
| 价格确认 + 成交确认 | `abs(price_return_short_pct) >= orderbook_notify_min_return_pct` 且 `volume_burst_ratio >= orderbook_notify_volume_burst_ratio` |

所以盘口异常可以被记录得比较细，但不会轻易单独推送。它更适合作为价格或动量信号的增强证据。

用公式写，就是：

```text
reportable(orderbook) =
    severity >= orderbook_standalone_min_severity
 and (
      same_batch_has_price_jump
      or (
        abs(price_return_short_pct) >= orderbook_notify_min_return_pct
        and volume_burst_ratio >= orderbook_notify_volume_burst_ratio
      )
    )
```

## 告警聚合规则

通过 reportable 过滤后，事件会进入同一标的的聚合窗口。

聚合窗口由下面参数控制：

```yaml
alert_aggregation_window_seconds: 30
```

在这个窗口内，同一标的可能连续触发价格跳变、动量异常、盘口异常。系统不会每个事件都发一条，而是合并成一条飞书消息。

合并时，同一异常类型只保留一个代表事件：

1. 优先保留严重程度更高的事件。
2. 严重程度相同时，保留触发时间更新的事件。
3. 不同异常类型按触发时间排序放入同一条消息。

比如同一只股票 30 秒内先出现动量异常，后出现价格跳变和盘口异常，最终可能合并成：

```text
类型：动量异常 + 价格跳变 + 盘口流动性异常
方向：向上
等级：高
```

这样比连续收到三条消息更适合盘中查看。

## 冷却和反向震荡保护

聚合后的告警还会经过 suppression 规则。

第一类是重复冷却：

```text
cooldown_seconds
```

同一标的、同一主异常类型、同一方向，在冷却期内再次出现时，如果新事件严重程度不高于上一次，就会被抑制。

第二类是反向震荡保护：

```text
opposite_direction_guard_seconds
```

如果刚发过向上告警，很短时间内又出现同一主异常类型的向下告警，系统会认为可能是来回抽动，先抑制这条反向消息。

注意，冷却键会以聚合后的主事件为准。主事件由最高严重程度决定，多个事件同级时取实现里的排序结果。聚合组内其他事件会共享这个 suppression key。

## 飞书消息格式

最终发送的是飞书 `im/v1/messages` 的 `post` 消息。

消息标题格式：

```text
A股异动告警 | {symbol name} | {severity}
```

正文包含：

| 字段 | 示例 |
| --- | --- |
| 类型 | `价格跳变 + 动量异常` |
| 方向 | `向上` 或 `向下` |
| 触发时间 | tick 的事件时间 |
| 最新价 | 触发 tick 的最新价 |
| 当前涨跌幅 | 如果行情字段或昨收、开盘价可用，则补充 |
| 等级 | 提醒、高、严重 |
| 每类异常测量值 | 价格涨跌幅、动量Z分数、脉冲涨跌幅、撤挂压力、买卖盘失衡等 |
| 原因 | 按异常类型生成的中文解释 |

飞书鉴权使用 `tenant_access_token`。发送规则包括：

- token 会缓存到接近过期前，提前 `token_refresh_margin_seconds` 秒刷新。
- 消息发送最多尝试 `max_attempts` 次。
- 遇到 token 类失败会强制刷新 token 后重试。
- 4xx 错误或达到最大次数后停止。
- 每次发送结果都会写入 `notification.jsonl` 审计日志。

## 关键参数速查

| 参数 | 作用 |
| --- | --- |
| `price_window_seconds` | 价格跳变短窗长度 |
| `price_return_threshold_pct` | 价格跳变基础阈值 |
| `momentum_impulse_seconds` | 动量脉冲窗口长度 |
| `momentum_baseline_seconds` | 动量基准窗口长度 |
| `momentum_z_threshold` | 动量 Z 分数基础阈值 |
| `momentum_min_return_pct` | 动量检测前置真实涨跌幅门槛 |
| `momentum_min_nonzero_baseline_samples` | 动量基准窗口中最少非零速度样本数 |
| `momentum_zero_mad_min_return_pct` | MAD 为 0 或低波动场景下的更严格位移门槛 |
| `momentum_notify_min_return_pct` | 动量告警可直接报告的真实涨跌幅门槛 |
| `momentum_notify_volume_burst_ratio` | 动量告警成交活跃度确认阈值 |
| `orderbook_min_consecutive_ticks` | 盘口异常连续命中 tick 数 |
| `orderbook_cancel_ratio_threshold` | 撤挂压力检测阈值 |
| `orderbook_imbalance_ratio_threshold` | 买卖盘失衡检测阈值 |
| `orderbook_standalone_min_severity` | 盘口异常单独报告的最低等级 |
| `orderbook_notify_min_return_pct` | 盘口告警需要的价格确认阈值 |
| `orderbook_notify_volume_burst_ratio` | 盘口告警需要的成交确认阈值 |
| `alert_aggregation_window_seconds` | 同一标的告警合并窗口 |
| `cooldown_seconds` | 同方向重复告警冷却时间 |
| `opposite_direction_guard_seconds` | 反向震荡保护时间 |

## 我的理解

这套规则的核心取向是：检测可以灵敏，通知必须克制。

价格跳变是最直接的盘面事实，所以默认可报告；动量异常容易在低波动环境里被放大，所以加了真实涨跌幅、非零样本、成交和盘口确认；盘口异常最容易抖动，所以要求连续命中，并且单独发通知时还要有价格和成交配合。

这样的结构适合盘中监控：先把候选异常完整记下来，方便事后回放校准；同时只把更有上下文的组合信号推给人，减少盯盘时的噪音。

## 附录：动量 Z 分数计算例子

假设当前规则配置是：

```text
momentum_impulse_seconds = 10
momentum_min_return_pct = 0.45
momentum_z_threshold = 6.5
```

最近 10 秒内的脉冲窗口数据是：

```text
起始价 = 10.00
最新价 = 10.09
实际用时 = 9 秒
```

先算脉冲窗口真实涨跌幅：

```text
impulse_return_pct = (10.09 - 10.00) / 10.00 * 100
                   = 0.90%
```

再换算成短窗价格变化速度：

```text
velocity = 0.90 / 9
         = 0.1000 %/秒
```

假设过去基准窗口里，相邻 tick 的历史速度样本是：

```text
[0.006, 0.008, 0.010, 0.011, 0.012, 0.014]
```

这些数字的单位同样是 `%/秒`。先算中位数：

```text
median = (0.010 + 0.011) / 2
       = 0.0105
```

然后计算每个样本相对中位数的绝对偏差：

```text
|0.006 - 0.0105| = 0.0045
|0.008 - 0.0105| = 0.0025
|0.010 - 0.0105| = 0.0005
|0.011 - 0.0105| = 0.0005
|0.012 - 0.0105| = 0.0015
|0.014 - 0.0105| = 0.0035
```

排序后得到：

```text
[0.0005, 0.0005, 0.0015, 0.0025, 0.0035, 0.0045]
```

MAD 是这些绝对偏差的中位数：

```text
MAD = (0.0015 + 0.0025) / 2
    = 0.0020
```

最后代入 robust z-score 公式：

```text
momentum_z = 0.6745 * (velocity - median) / MAD
           = 0.6745 * (0.1000 - 0.0105) / 0.0020
           = 0.6745 * 44.75
           = 30.18
```

所以这一次的结果是：

```text
动量 Z 分数 = 30.18
```

因为：

```text
impulse_return_pct = 0.90% >= momentum_min_return_pct 0.45%
momentum_z = 30.18 >= momentum_z_threshold 6.5
```

因此它会被识别为一条动量异常候选。后续是否发送飞书，还要继续看 `momentum_notify_min_return_pct`、`volume_burst_ratio`、盘口压力或同组价格跳变这些告警确认条件。
