---
title: "量化介入股票扫描：筛选条件与图表整理"
date: 2026-05-18T22:10:00+08:00
lastmod: 2026-05-18T22:10:00+08:00
draft: false
categories: []
tags: ["A股"]
description: "整理一次 A 股量化介入扫描的筛选口径，以及 output 目录下已经生成的分时图。"
keywords:
- A股
- 量化扫描
- 盘口
- 成交量
---

这篇文章记录一下这个小项目当前使用的 A 股“量化介入”扫描口径，并把 `output` 目录下已经跑出来的分时图统一放进博客，方便后面复盘。这里不重新计算，只做整理。

<!--more-->

> 仅作为个人技术记录和复盘材料，不构成任何投资建议。

## 产出概览

本次共整理 `output` 目录下的 `96` 张分时图。

| 日期 | 分时图数量 |
|---|---:|
| 2026-05-12 | 22 |
| 2026-05-13 | 24 |
| 2026-05-14 | 16 |
| 2026-05-15 | 18 |
| 2026-05-18 | 16 |

## 图集

### 2026-05-12

#### 东安动力（SHSE.600178）

{{< imgx src="/img/quant-intrusion-scan/output/20260512/charts/SHSE.600178_20260512_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 信雅达（SHSE.600571）

{{< imgx src="/img/quant-intrusion-scan/output/20260512/charts/SHSE.600571_20260512_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 迪哲医药-U（SHSE.688192）

{{< imgx src="/img/quant-intrusion-scan/output/20260512/charts/SHSE.688192_20260512_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 泽璟制药-U（SHSE.688266）

{{< imgx src="/img/quant-intrusion-scan/output/20260512/charts/SHSE.688266_20260512_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 荣昌生物（SHSE.688331）

{{< imgx src="/img/quant-intrusion-scan/output/20260512/charts/SHSE.688331_20260512_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 益方生物-U（SHSE.688382）

{{< imgx src="/img/quant-intrusion-scan/output/20260512/charts/SHSE.688382_20260512_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 诺诚健华（SHSE.688428）

{{< imgx src="/img/quant-intrusion-scan/output/20260512/charts/SHSE.688428_20260512_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 三变科技（SZSE.002112）

{{< imgx src="/img/quant-intrusion-scan/output/20260512/charts/SZSE.002112_20260512_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 建设工业（SZSE.002265）

{{< imgx src="/img/quant-intrusion-scan/output/20260512/charts/SZSE.002265_20260512_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 万润科技（SZSE.002654）

{{< imgx src="/img/quant-intrusion-scan/output/20260512/charts/SZSE.002654_20260512_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 金时科技（SZSE.002951）

{{< imgx src="/img/quant-intrusion-scan/output/20260512/charts/SZSE.002951_20260512_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 朝阳科技（SZSE.002981）

{{< imgx src="/img/quant-intrusion-scan/output/20260512/charts/SZSE.002981_20260512_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 顺网科技（SZSE.300113）

{{< imgx src="/img/quant-intrusion-scan/output/20260512/charts/SZSE.300113_20260512_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 赢合科技（SZSE.300457）

{{< imgx src="/img/quant-intrusion-scan/output/20260512/charts/SZSE.300457_20260512_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 启迪设计（SZSE.300500）

{{< imgx src="/img/quant-intrusion-scan/output/20260512/charts/SZSE.300500_20260512_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 晨曦航空（SZSE.300581）

{{< imgx src="/img/quant-intrusion-scan/output/20260512/charts/SZSE.300581_20260512_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 必创科技（SZSE.300667）

{{< imgx src="/img/quant-intrusion-scan/output/20260512/charts/SZSE.300667_20260512_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 中科信息（SZSE.300678）

{{< imgx src="/img/quant-intrusion-scan/output/20260512/charts/SZSE.300678_20260512_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 捷强装备（SZSE.300875）

{{< imgx src="/img/quant-intrusion-scan/output/20260512/charts/SZSE.300875_20260512_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 北方长龙（SZSE.301357）

{{< imgx src="/img/quant-intrusion-scan/output/20260512/charts/SZSE.301357_20260512_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 博苑新材（SZSE.301617）

{{< imgx src="/img/quant-intrusion-scan/output/20260512/charts/SZSE.301617_20260512_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 中航成飞（SZSE.302132）

{{< imgx src="/img/quant-intrusion-scan/output/20260512/charts/SZSE.302132_20260512_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

### 2026-05-13

#### 永安期货（SHSE.600927）

{{< imgx src="/img/quant-intrusion-scan/output/20260513/charts/SHSE.600927_20260513_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 富佳股份（SHSE.603219）

{{< imgx src="/img/quant-intrusion-scan/output/20260513/charts/SHSE.603219_20260513_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 微导纳米（SHSE.688147）

{{< imgx src="/img/quant-intrusion-scan/output/20260513/charts/SHSE.688147_20260513_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 美迪西（SHSE.688202）

{{< imgx src="/img/quant-intrusion-scan/output/20260513/charts/SHSE.688202_20260513_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 浙海德曼（SHSE.688577）

{{< imgx src="/img/quant-intrusion-scan/output/20260513/charts/SHSE.688577_20260513_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 金博股份（SHSE.688598）

{{< imgx src="/img/quant-intrusion-scan/output/20260513/charts/SHSE.688598_20260513_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 中邮科技（SHSE.688648）

{{< imgx src="/img/quant-intrusion-scan/output/20260513/charts/SHSE.688648_20260513_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 福立旺（SHSE.688678）

{{< imgx src="/img/quant-intrusion-scan/output/20260513/charts/SHSE.688678_20260513_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 平潭发展（SZSE.000592）

{{< imgx src="/img/quant-intrusion-scan/output/20260513/charts/SZSE.000592_20260513_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 千味央厨（SZSE.001215）

{{< imgx src="/img/quant-intrusion-scan/output/20260513/charts/SZSE.001215_20260513_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 富岭股份（SZSE.001356）

{{< imgx src="/img/quant-intrusion-scan/output/20260513/charts/SZSE.001356_20260513_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 国轩高科（SZSE.002074）

{{< imgx src="/img/quant-intrusion-scan/output/20260513/charts/SZSE.002074_20260513_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 全聚德（SZSE.002186）

{{< imgx src="/img/quant-intrusion-scan/output/20260513/charts/SZSE.002186_20260513_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 启明信息（SZSE.002232）

{{< imgx src="/img/quant-intrusion-scan/output/20260513/charts/SZSE.002232_20260513_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 蔚蓝锂芯（SZSE.002245）

{{< imgx src="/img/quant-intrusion-scan/output/20260513/charts/SZSE.002245_20260513_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 九安医疗（SZSE.002432）

{{< imgx src="/img/quant-intrusion-scan/output/20260513/charts/SZSE.002432_20260513_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 启明星辰（SZSE.002439）

{{< imgx src="/img/quant-intrusion-scan/output/20260513/charts/SZSE.002439_20260513_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 航宇微（SZSE.300053）

{{< imgx src="/img/quant-intrusion-scan/output/20260513/charts/SZSE.300053_20260513_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 海峡创新（SZSE.300300）

{{< imgx src="/img/quant-intrusion-scan/output/20260513/charts/SZSE.300300_20260513_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 思特奇（SZSE.300608）

{{< imgx src="/img/quant-intrusion-scan/output/20260513/charts/SZSE.300608_20260513_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 广和通（SZSE.300638）

{{< imgx src="/img/quant-intrusion-scan/output/20260513/charts/SZSE.300638_20260513_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 安克创新（SZSE.300866）

{{< imgx src="/img/quant-intrusion-scan/output/20260513/charts/SZSE.300866_20260513_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 海力风电（SZSE.301155）

{{< imgx src="/img/quant-intrusion-scan/output/20260513/charts/SZSE.301155_20260513_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 壹连科技（SZSE.301631）

{{< imgx src="/img/quant-intrusion-scan/output/20260513/charts/SZSE.301631_20260513_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

### 2026-05-14

#### 小商品城（SHSE.600415）

{{< imgx src="/img/quant-intrusion-scan/output/20260514/charts/SHSE.600415_20260514_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 先导基电（SHSE.600641）

{{< imgx src="/img/quant-intrusion-scan/output/20260514/charts/SHSE.600641_20260514_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 中马传动（SHSE.603767）

{{< imgx src="/img/quant-intrusion-scan/output/20260514/charts/SHSE.603767_20260514_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 鼎阳科技（SHSE.688112）

{{< imgx src="/img/quant-intrusion-scan/output/20260514/charts/SHSE.688112_20260514_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 慧智微-U（SHSE.688512）

{{< imgx src="/img/quant-intrusion-scan/output/20260514/charts/SHSE.688512_20260514_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 高新发展（SZSE.000628）

{{< imgx src="/img/quant-intrusion-scan/output/20260514/charts/SZSE.000628_20260514_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 盈方微（SZSE.000670）

{{< imgx src="/img/quant-intrusion-scan/output/20260514/charts/SZSE.000670_20260514_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 维信诺（SZSE.002387）

{{< imgx src="/img/quant-intrusion-scan/output/20260514/charts/SZSE.002387_20260514_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 亚世光电（SZSE.002952）

{{< imgx src="/img/quant-intrusion-scan/output/20260514/charts/SZSE.002952_20260514_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 竞业达（SZSE.003005）

{{< imgx src="/img/quant-intrusion-scan/output/20260514/charts/SZSE.003005_20260514_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 旗天科技（SZSE.300061）

{{< imgx src="/img/quant-intrusion-scan/output/20260514/charts/SZSE.300061_20260514_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 昌红科技（SZSE.300151）

{{< imgx src="/img/quant-intrusion-scan/output/20260514/charts/SZSE.300151_20260514_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 劲拓股份（SZSE.300400）

{{< imgx src="/img/quant-intrusion-scan/output/20260514/charts/SZSE.300400_20260514_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 宏达电子（SZSE.300726）

{{< imgx src="/img/quant-intrusion-scan/output/20260514/charts/SZSE.300726_20260514_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 创益通（SZSE.300991）

{{< imgx src="/img/quant-intrusion-scan/output/20260514/charts/SZSE.300991_20260514_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 欧圣电气（SZSE.301187）

{{< imgx src="/img/quant-intrusion-scan/output/20260514/charts/SZSE.301187_20260514_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

### 2026-05-15

#### 迪贝电气（SHSE.603320）

{{< imgx src="/img/quant-intrusion-scan/output/20260515/charts/SHSE.603320_20260515_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 蔚蓝生物（SHSE.603739）

{{< imgx src="/img/quant-intrusion-scan/output/20260515/charts/SHSE.603739_20260515_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 能科科技（SHSE.603859）

{{< imgx src="/img/quant-intrusion-scan/output/20260515/charts/SHSE.603859_20260515_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 禾川科技（SHSE.688320）

{{< imgx src="/img/quant-intrusion-scan/output/20260515/charts/SHSE.688320_20260515_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 南芯科技（SHSE.688484）

{{< imgx src="/img/quant-intrusion-scan/output/20260515/charts/SHSE.688484_20260515_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 中控技术（SHSE.688777）

{{< imgx src="/img/quant-intrusion-scan/output/20260515/charts/SHSE.688777_20260515_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 正泰电源（SZSE.002150）

{{< imgx src="/img/quant-intrusion-scan/output/20260515/charts/SZSE.002150_20260515_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 成飞集成（SZSE.002190）

{{< imgx src="/img/quant-intrusion-scan/output/20260515/charts/SZSE.002190_20260515_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 环球印务（SZSE.002799）

{{< imgx src="/img/quant-intrusion-scan/output/20260515/charts/SZSE.002799_20260515_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 比音勒芬（SZSE.002832）

{{< imgx src="/img/quant-intrusion-scan/output/20260515/charts/SZSE.002832_20260515_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 阳光电源（SZSE.300274）

{{< imgx src="/img/quant-intrusion-scan/output/20260515/charts/SZSE.300274_20260515_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 汉宇集团（SZSE.300403）

{{< imgx src="/img/quant-intrusion-scan/output/20260515/charts/SZSE.300403_20260515_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 华辰装备（SZSE.300809）

{{< imgx src="/img/quant-intrusion-scan/output/20260515/charts/SZSE.300809_20260515_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 恒辉安防（SZSE.300952）

{{< imgx src="/img/quant-intrusion-scan/output/20260515/charts/SZSE.300952_20260515_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 鼎熔岩（SZSE.301028）

{{< imgx src="/img/quant-intrusion-scan/output/20260515/charts/SZSE.301028_20260515_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 恒工精密（SZSE.301261）

{{< imgx src="/img/quant-intrusion-scan/output/20260515/charts/SZSE.301261_20260515_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 福赛科技（SZSE.301529）

{{< imgx src="/img/quant-intrusion-scan/output/20260515/charts/SZSE.301529_20260515_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 首航新能（SZSE.301658）

{{< imgx src="/img/quant-intrusion-scan/output/20260515/charts/SZSE.301658_20260515_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

### 2026-05-18

#### 中望软件（SHSE.688083）

{{< imgx src="/img/quant-intrusion-scan/output/20260518/charts/SHSE.688083_20260518_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 南网科技（SHSE.688248）

{{< imgx src="/img/quant-intrusion-scan/output/20260518/charts/SHSE.688248_20260518_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 财富趋势（SHSE.688318）

{{< imgx src="/img/quant-intrusion-scan/output/20260518/charts/SHSE.688318_20260518_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 孚能科技（SHSE.688567）

{{< imgx src="/img/quant-intrusion-scan/output/20260518/charts/SHSE.688567_20260518_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 新致软件（SHSE.688590）

{{< imgx src="/img/quant-intrusion-scan/output/20260518/charts/SHSE.688590_20260518_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 星网宇达（SZSE.002829）

{{< imgx src="/img/quant-intrusion-scan/output/20260518/charts/SZSE.002829_20260518_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 鼎捷数智（SZSE.300378）

{{< imgx src="/img/quant-intrusion-scan/output/20260518/charts/SZSE.300378_20260518_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 丝路视觉（SZSE.300556）

{{< imgx src="/img/quant-intrusion-scan/output/20260518/charts/SZSE.300556_20260518_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 广和通（SZSE.300638）

{{< imgx src="/img/quant-intrusion-scan/output/20260518/charts/SZSE.300638_20260518_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 申昊科技（SZSE.300853）

{{< imgx src="/img/quant-intrusion-scan/output/20260518/charts/SZSE.300853_20260518_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 国安达（SZSE.300902）

{{< imgx src="/img/quant-intrusion-scan/output/20260518/charts/SZSE.300902_20260518_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 国能日新（SZSE.301162）

{{< imgx src="/img/quant-intrusion-scan/output/20260518/charts/SZSE.301162_20260518_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 软通动力（SZSE.301236）

{{< imgx src="/img/quant-intrusion-scan/output/20260518/charts/SZSE.301236_20260518_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 汉朔科技（SZSE.301275）

{{< imgx src="/img/quant-intrusion-scan/output/20260518/charts/SZSE.301275_20260518_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 蓝箭电子（SZSE.301348）

{{< imgx src="/img/quant-intrusion-scan/output/20260518/charts/SZSE.301348_20260518_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}

#### 盘古智能（SZSE.301456）

{{< imgx src="/img/quant-intrusion-scan/output/20260518/charts/SZSE.301456_20260518_intraday.png" alt="" width="900px" style="max-width: 100%;" >}}
