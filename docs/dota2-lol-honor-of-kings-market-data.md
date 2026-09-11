# Dota 2 / League of Legends / 王者荣耀：玩家与观看规模数据（去估算版）

> 更新日期：2026-09-11  
> 用途：为 Dota 2 AI 陪玩/游戏助手项目判断市场基本盘、内容消费规模与游戏生命周期。  
> 原则：**只收录可追溯的官方披露、平台公开计数/指数、或统计口径明确的第三方监测数据。任何推算、模型估计、未经官方确认的“实时玩家数”均不作为事实写入。**

---

## 1. 先统一数据口径

不同游戏公开的数据口径完全不同，不能直接把数字放在一起比较。

| 指标 | 含义 | 能否与其他指标直接比较 |
|---|---|---|
| Avg. CCU / 平均同时在线 | 某段时间内，任意时刻平均有多少人在游戏中 | 只能和同口径 CCU 比 |
| Peak CCU / PCU | 某个时间点或某日最高同时在线人数 | 不能当作 DAU |
| DAU | 一天内至少活跃一次的去重用户数 | 可衡量日活，但不能和 CCU 直接比较 |
| MAU | 一个月内至少活跃一次的去重用户数 | 可衡量月活，但不能和 CCU 直接比较 |
| 直播 Avg Viewers | 一段直播时间内的平均同时观看人数 | 可以作为真实并发观看指标 |
| 直播 Peak Viewers | 某一时刻的最高同时观看人数 | 不是累计观看人数 |
| 斗鱼/虎牙“热度/人气” | 平台综合指数，通常混合在线、访问、互动、直播时长等因素 | **不是观众人数，不能换算成 CCV** |

### 本文明确不做的事情

1. 不用 Dota 2 的 CCU 和假设游戏时长去反推 DAU。
2. 不采用 ActivePlayer、MMO Population 等“实时玩家计数器”的模型估算作为事实。
3. 不把斗鱼、虎牙的“热度/人气”写成“有多少万人正在看”。
4. 不把斗鱼热度与虎牙热度直接相加，也不跨平台比较绝对数值，因为算法不同。
5. 不把 Esports Charts 等未覆盖中国平台的赛事观众数称为“全球全部观众”。
6. 没有可靠公开数据时，直接写“未公开/无法可靠获得”。

---

# 2. 三款游戏目前能确认的核心玩家数据

## 2.1 Dota 2

### 最新可核实的全球 Steam 同时在线

SteamCharts 对 Steam AppID 570（Dota 2）的记录：

| 时间 | 平均同时在线 Avg. Players | 峰值同时在线 Peak Players |
|---|---:|---:|
| 2026-08 | **617,077.90** | **981,123** |
| 2026-07 | 545,528.29 | 860,019 |
| 2026-06 | 466,001.97 | 862,639 |
| 2026-05 | 422,693.28 | 664,213 |
| 2026-04 | 425,918.58 | 695,006 |
| 2026-03 | 477,855.57 | 859,016 |
| 2026-02 | 597,009.82 | 869,734 |
| 2026-01 | 569,324.18 | 855,690 |
| 截至 2026-09-11 的 Last 30 Days 快照 | **609,320.89** | **981,123** |

来源：SteamCharts — Dota 2  
https://steamcharts.com/app/570

### 这些数字是全球，不是中国国服

Steam 的公开统计对象是 **Dota 2 AppID 570**。SteamDB 的 `Dota 2 - Perfect World (CN)` 中国完美世界包同样包含 AppID 570，因此公开的 AppID 570 玩家计数是 Dota 2 的 Steam 总盘，而不是单独的中国国服计数。

来源：SteamDB — Dota 2 - Perfect World (CN)  
https://steamdb.info/sub/140254/

**公开数据没有把中国、东南亚、俄罗斯、欧洲、美洲等地区分别拆出来。** 因此：

- 可以确认：2026 年 8 月，Dota 2 全球 Steam 平均同时在线约 **61.7 万**。
- 可以确认：2026 年 8 月峰值同时在线约 **98.1 万**。
- **不能确认：中国 Dota 2 当前有多少 DAU、MAU 或平均 CCU。**
- **不能把 61.7 万理解成“一天只有 61.7 万个玩家”。** 它是平均同时在线，不是日去重玩家数。

### 近年趋势：Dota 2 平均同时在线

下表为本文根据 SteamCharts 每个月公布的 `Avg. Players` 做的**算术平均**。这是对公开月度实测值的汇总计算，不是 Valve 公布的 DAU/MAU，也不是玩家规模估算。

| 年份 | 月均 Avg. Players 的年度算术平均 |
|---|---:|
| 2020 | 433,340 |
| 2021 | 422,208 |
| 2022 | 466,160 |
| 2023 | 429,803 |
| 2024 | 450,708 |
| 2025 | **479,271** |
| 2026 1–8 月 | **515,176** |

来源：SteamCharts 月度历史数据  
https://steamcharts.com/app/570

可以直接从实测 CCU 得出的结论只有：

- 2021 是上述近年区间中的低点之一。
- 2024、2025 连续高于前一年。
- 2026 年 1–8 月月均 CCU 的算术平均进一步上升到约 51.5 万。
- 2026 年 8 月单月平均 CCU 达 61.7 万，明显高于 2024/2025 的年度月均水平。

这说明 Dota 2 的**同时在线基本盘近两年处于回升状态**。这不等于证明 DAU/MAU 同比例增长，因为 Valve 没有公开当前 DAU/MAU。

### Dota 2 当前 DAU / MAU

**没有找到 Valve/Steam 当前公开的准确 Dota 2 DAU 或 MAU。**

因此本文不使用任何通过 CCU、游戏时长或第三方模型反推出来的 Dota 2 DAU/MAU。

---

## 2.2 League of Legends（英雄联盟）

Riot 没有像 Steam 一样提供当前全球实时玩家计数器，因此 **2026 年准确全球 CCU、DAU、MAU没有可靠公开绝对数值可引用**。

目前可确认的官方历史硬数据包括：

| 时间 | 官方披露 | 口径 |
|---|---:|---|
| 2014 | **> 6,700 万/月** | MAU |
| 2016 | **> 1 亿/月** | MAU |
| 2019-08 | **约 800 万** | Riot、腾讯、Garena 全区域的“平均每日峰值同时在线 PCU” |

Riot 2019 十周年官方文章原文说明，其约 800 万数字是：

> average daily peak concurrent users (PCU) for all Riot, Tencent, and Garena regions during August 2019

也就是说，**800 万不是 DAU，而是 2019 年 8 月各日峰值同时在线的平均水平。**

来源：Riot Games / League of Legends 官方十周年文章  
https://www.leagueoflegends.com/en-us/news/riot-games/join-us-oct-15th-to-celebrate-10-years-of-league/

历史 MAU 来源：Riot Games League of Legends Fact Sheet  
https://www.riotgames.com/darkroom/original/8e2a0ca2dbd5c484ff503513ed591f32%3Adb0dce6771e17a4e90bf6ba43a32c7ab/leagueoflegends-fact-sheet.pdf

### 2026 年趋势：有官方方向性披露，但没有绝对人数

腾讯 2026 年业绩材料披露，League of Legends 在 2026 年出现 DAU 回升；腾讯 2Q26 材料进一步写明 **DAU 同比增长（DAU increased YoY in 2Q26）**，主要受到 ARAM: Mayhem 模式推动。

但腾讯没有在该披露中给出全球/中国区的绝对 DAU 数字。

腾讯投资者关系：  
https://www.tencent.com/zh-cn/investors/financial-reports/

因此本文只保留：

- **可确认：2026 Q2 LoL DAU 同比增长。**
- **不可确认：2026 年 LoL 全球准确 DAU、MAU、平均 CCU。**
- 不采用网上常见的“当前 1.x 亿月活”“实时 xxx 万玩家”等没有 Riot/Tencent 原始披露支持的精确数值。

---

## 2.3 王者荣耀 / Honor of Kings

王者荣耀公开的用户规模口径比 LoL 更明确。

### 2025 年官方披露

王者荣耀官方在 2025 年十周年节点公布：

- **国服 DAU（日活）突破 1.39 亿**；
- **Honor of Kings 系列全球合计 MAU 突破 2.6 亿。**

王者荣耀官方微博内容的新浪镜像：  
https://www.sina.cn/news/detail/5226069252900334.html

Level Infinite 对全球数据的官方表述：  
https://www.levelinfinite.com/news/hok-plus-2-0-update/

注意口径：

- **1.39 亿 = 国服 DAU**，可以用来描述中国玩家日活基本盘；
- **2.6 亿 = Honor of Kings titles combined 的全球 MAU**，不是中国 DAU，也不是同时在线。

### 历史里程碑

2020 年王者荣耀五周年时，官方公布日活跃用户数达到 **1 亿**。

腾讯云开发者社区对该官方里程碑的记录：  
https://developer.cloud.tencent.com/article/1744759

因此能够可靠看到的公开节点是：

| 时间 | 玩家指标 |
|---|---:|
| 2020 | DAU 约 **1 亿** |
| 2025 | 国服 DAU **>1.39 亿** |
| 2025 | 系列全球 MAU **>2.6 亿** |

这里不能用 2020 和 2025 两个节点去假定中间每一年按固定速度增长；本文不插值。

---

# 3. 三款游戏玩家规模：能比较什么，不能比较什么

| 游戏 | 当前/近期最可靠公开玩家指标 | 地域 | 是否可与另外两款直接比 |
|---|---|---|---|
| Dota 2 | 2026-08 Avg. CCU **617,078**；Peak **981,123** | 全球 Steam 总盘 | **不能**直接和 DAU/MAU 比 |
| League of Legends | 2026 Q2 官方仅披露 DAU 同比增长；最新绝对 DAU/MAU未公开 | 未给绝对数 | 无法做当前绝对人数对比 |
| 王者荣耀 | 2025 国服 DAU **>1.39 亿**；全球系列 MAU **>2.6 亿** | 中国 / 全球分别有口径 | 只能与相同 DAU/MAU 口径比较 |

因此，基于公开数据可以说王者荣耀具有极大的中国日活基本盘；但**不能写成“王者是 Dota 2 的 225 倍”之类的比较**，因为一个是 DAU，一个是平均 CCU。

同理，目前也没有足够的官方绝对数据去精确写出“LoL 玩家数是 Dota 2 的多少倍”。

---

# 4. 中国直播平台：斗鱼、虎牙的数据必须按“热度”处理

## 4.1 为什么不能写成真实观看人数

斗鱼、虎牙公开页面主要展示的是“热度/人气”等综合指标，而不是标准的 Concurrent Viewers。

DoHuya 对中国平台指标的说明指出：这些热度指标会综合实时活跃观众、访问、直播时长、内容量、互动等变量；不同平台算法也不同，因此：

- **热度不是实际同时观看人数；**
- **斗鱼热度和虎牙热度不能直接做绝对值比较；**
- 可以用来观察同一平台内不同游戏的相对位置、变化方向和内容生态强弱。

来源：DoHuya 方法说明  
https://dohuya.com/zh/about

## 4.2 近期斗鱼/虎牙游戏分类热度

以下数值全部是**平台热度指数，不是观众人数**。

### 斗鱼：最近 7 天分类表现（查询快照）

| 游戏 | 平均热度 | 峰值热度 | 分类排名 |
|---|---:|---:|---:|
| League of Legends | 40,009,925 | 75,045,467 | #4 |
| 王者荣耀 | 26,968,510 | 76,111,716 | #3 |
| Dota 2 | **18,249,366** | **41,104,196** | **#9** |

来源：DoHuya — DouYu Games  
https://dohuya.com/zh/games?platform=douyu

可以确认的是：**Dota 2 在斗鱼仍有可见的头部游戏分类位置；最近 7 天进入该统计的 Top 10。**

不能确认的是：“斗鱼有 1,825 万人在同时看 Dota 2”。这是错误解释。

### 虎牙：最近 7 天分类表现（查询快照）

| 游戏 | 平均热度 | 峰值热度 | 分类排名 |
|---|---:|---:|---:|
| 王者荣耀 | **49,812,701** | 118,737,264 | #1 |
| League of Legends | **33,619,536** | 79,665,691 | #4 |
| Dota 2 | — | — | 当前该榜单 Top 20 未出现 |

来源：DoHuya — Huya Games  
https://dohuya.com/zh/games?platform=huya

这只能说明在当前 DoHuya 所抓取的虎牙 Top 20 分类中 Dota 2 没有进入榜单，**不能据此推断虎牙 Dota 2 观众为 0**。

### 整个平台规模作为背景

斗鱼官方 2026 Q2 财报披露，斗鱼直播业务平均 MAU 为 **4,540 万**，付费用户约 **230 万**。

来源：DouYu Investor Relations — 2026 Q2  
https://ir.douyu.com/Press-Releases/6a86c32bef90a235e9d8f471

这个 MAU 是**整个斗鱼平台**，不是 Dota 2 用户数，不能用于反推 Dota 2 观众规模。

虎牙从 2026 年起不再按过去方式定期披露 MAU；其 2025 Q4 曾披露总平均 MAU **1.60 亿**，但该指标自 2025 年中起已经扩大到国内外多平台、服务和设备，不能与早年的“国内移动端 MAU”直接同比。

来源：HUYA 2025 Q4 / FY2025 Results  
https://ir.huya.com/2026-03-17-HUYA-Inc-Reports-Fourth-Quarter-and-Fiscal-Year-2025-Unaudited-Financial-Results-and-Announces-Cash-Dividend

---

# 5. 可直接统计真实并发观众的国际直播平台：Twitch

Twitch 的分类观看数据可以用标准 Avg Viewers / Peak Viewers 表示，因此比中国平台“热度”更容易理解。但它**只代表 Twitch 生态**，尤其不能代表王者荣耀在中国的观看规模。

## 2026-07

| 游戏 | Twitch 平均同时观众 | 峰值同时观众 | Hours Watched |
|---|---:|---:|---:|
| League of Legends | **111,141** | 558,203 | 80,150,260 |
| Dota 2 | **44,870** | 229,224 | 32,357,693 |
| Honor of Kings | **77** | 405 | 54,158 |

## 2026-08

| 游戏 | Twitch 平均同时观众 | 峰值同时观众 | Hours Watched |
|---|---:|---:|---:|
| League of Legends | **103,246** | 291,827 | 75,093,787 |
| Dota 2 | **93,748** | **1,034,021** | 68,185,055 |
| Honor of Kings | **83** | 2,574 | 58,627 |

来源：Streams Charts  
Dota 2：https://streamscharts.com/games/dota-2  
League of Legends：https://streamscharts.com/games/league-of-legends  
Honor of Kings：https://streamscharts.com/games/honor-of-kings

### 解读限制

- 这些是 Twitch 的实际并发观看数据，不是全球所有直播平台合计。
- 2026 年 8 月 Dota 2 的 Twitch 峰值超过 100 万，与大型赛事期重合，因此不能把它当作普通月份日常观看水平。
- Honor of Kings 的 Twitch 数字极小，**不能据此判断王者整体观看规模很小**；其核心用户和直播生态高度集中在中国及其他非 Twitch 平台。

---

# 6. 电竞赛事观看：用统一第三方监测口径观察趋势

电竞赛事观众数据使用 Esports Charts。它能够统一统计大量国际直播平台，但对中国平台由于“热度/人气”等口径无法可靠还原 Concurrent Viewers，部分赛事会明确排除中国平台。

因此本文把这些数字表述为：

> **Esports Charts 可追踪平台上的峰值并发观众（Peak Viewers）**

而不是“全球全部观众”。

## 6.1 League of Legends World Championship

| 年份 | Worlds Peak Viewers |
|---|---:|
| 2021 | 4,018,728 |
| 2022 | 5,147,701 |
| 2023 | 6,402,760 |
| 2024 | **6,941,610** |
| 2025 | **6,752,585** |

来源：Esports Charts  
https://escharts.com/games/lol

Esports Charts 对 Worlds 数据明确说明，其统计中不包含无法可靠聚合的中国直播平台数据。

从这个统一监测口径看，LoL Worlds 在 2021–2024 的可追踪峰值观众持续增长，2025 年略低于 2024 历史高位，但仍显著高于 2021–2023。

## 6.2 Dota 2 — The International 2026

Esports Charts 当前赛事表记录：

- TI 2026 Peak Viewers：约 **1.796 百万**；
- Hours Watched：约 **64.5 百万小时**。

来源：Esports Charts — The International 2026  
https://escharts.com/news/team-spirit-makes-history-international-2026

该来源同时指出：

- TI10（2021）峰值约 **2.74 百万**；
- TI 2025 约 **1.785 百万**；
- TI 2026 约 **1.79 百万**，略高于 2025，并成为 TI 历史较高的一届之一。

因此可以确认的趋势是：Dota 2 顶级赛事观看在 2022–2023 低于 TI10 高峰，最近两届出现恢复，但当前仍未回到 TI10 的历史峰值。

## 6.3 Honor of Kings World Cup

| 年份 | Peak Viewers（Esports Charts 可追踪平台） |
|---|---:|
| 2025 | **653,309** |
| 2026 | **约 778,000** |

来源：Esports Charts  
https://escharts.com/news/honor-kings-world-cup-2026-viewership

该来源**明确说明中国直播平台不计入**，因为中国平台公开的“viewership/热度”与标准 Concurrent Viewers 不可直接兼容。

所以这两个数字适合观察王者国际电竞传播趋势，**不能代表王者在中国的完整赛事观众规模。**

---

# 7. 把“玩家规模”和“内容观看规模”分开看

基于目前能验证的数据，能够成立的判断如下。

## Dota 2

**已确认：**

- 全球 Steam 2026-08 平均同时在线约 **61.7 万**；月峰值约 **98.1 万**。
- 2024、2025、2026 YTD 的平均 CCU 总体高于 2021–2023 低位，近期存在明确回升。
- 斗鱼 Dota 2 当前仍在该平台游戏分类热度 Top 10 范围内。
- Twitch 日常观看规模低于 LoL，但大型赛事期能够出现百万级峰值并发观看。
- TI 2026 在 Esports Charts 可追踪平台约 **179 万级峰值观众**。

**未确认：**

- 中国 Dota 2 当前 DAU / MAU / 平均 CCU。
- 斗鱼、虎牙 Dota 2 的真实同时观看人数。
- Dota 2 当前全球 DAU / MAU。

## League of Legends

**已确认：**

- Riot 历史官方数据曾达到 2019 年约 **800 万平均每日峰值 PCU**（全球 Riot + 腾讯 + Garena 区域）。
- 腾讯披露 2026 Q2 LoL DAU 同比增长，但没有给出绝对数。
- Worlds 2025 在 Esports Charts 可追踪平台峰值约 **675 万**，电竞观看规模显著高于 Dota 2 / Honor of Kings 的国际可追踪赛事数据。
- 2026 年 7–8 月 Twitch 平均同时观看约 10 万级。

**未确认：**

- 2026 全球准确 DAU、MAU、平均 CCU。
- 中国 LoL 当前准确 DAU/MAU。
- 斗鱼、虎牙真实同时观看人数。

## 王者荣耀

**已确认：**

- 2025 国服 DAU **>1.39 亿**。
- 2025 Honor of Kings 系列全球 MAU **>2.6 亿**。
- 2020 官方曾公布 DAU 达到 **1 亿**。
- 虎牙当前王者荣耀分类热度处于头部。
- HOK World Cup 2026 的国际可追踪峰值观众约 **77.8 万**，但明确不包含中国直播平台。

**未确认：**

- 当前实时 CCU。
- 斗鱼/虎牙真实同时观看人数。
- 中国全部平台赛事的可比标准 CCV 总数。

---

# 8. 对 Dota 2 AI 陪玩项目最重要的事实

在不使用任何猜测数据的情况下，当前可以作为市场判断底座的事实是：

1. **Dota 2 不是“只剩几十万总玩家”。** 61.7 万是 2026-08 的全球平均“同时在线”，不是 DAU，也不是 MAU。
2. **Dota 2 近期 CCU 确实处于回升区间。** 这一点可以直接从 Steam 历史月度实测数据验证。
3. **Dota 2 中国玩家绝对数量目前缺少可靠公开数据。** 不能用全球 Steam CCU 假装成国服人数。
4. **Dota 2 在中国直播端仍有可见内容生态，尤其斗鱼。** 但中国直播平台公开的是热度指标，无法据此给出“真实有多少人同时看”。
5. **LoL 的绝对玩家盘无法用当前官方数字精确比较。** Riot/Tencent 近期只披露了 DAU 回升方向，没有公开绝对 DAU/MAU。
6. **王者荣耀的中国用户基本盘远大于能从 Dota/LoL 公布口径直接观察到的数字，且有官方 DAU/MAU支撑。** 但仍不能拿王者 DAU 与 Dota Avg. CCU 做倍数比较。
7. 对 AI 陪玩产品而言，下一步需要补的不是继续猜“Dota 中国有多少万人”，而是寻找可验证的 **中国 Dota 玩家画像、游戏时长、语音/组队习惯、付费与内容消费行为**。如果没有可靠来源，应继续标记为“未知”，而不是模型估算。

---

# 9. 数据源清单

### Dota 2 玩家数据
- SteamCharts — Dota 2: https://steamcharts.com/app/570
- SteamDB — Dota 2: https://steamdb.info/app/570/charts/
- SteamDB — Dota 2 Perfect World (CN): https://steamdb.info/sub/140254/

### League of Legends
- Riot Games 10-year article: https://www.leagueoflegends.com/en-us/news/riot-games/join-us-oct-15th-to-celebrate-10-years-of-league/
- Riot LoL Fact Sheet: https://www.riotgames.com/darkroom/original/8e2a0ca2dbd5c484ff503513ed591f32%3Adb0dce6771e17a4e90bf6ba43a32c7ab/leagueoflegends-fact-sheet.pdf
- Tencent Financial Reports: https://www.tencent.com/zh-cn/investors/financial-reports/

### 王者荣耀 / Honor of Kings
- 王者荣耀官方微博披露（新浪镜像）: https://www.sina.cn/news/detail/5226069252900334.html
- Level Infinite: https://www.levelinfinite.com/news/hok-plus-2-0-update/
- 腾讯云开发者社区（2020 官方 DAU 里程碑记录）: https://developer.cloud.tencent.com/article/1744759

### 中国直播
- DoHuya methodology: https://dohuya.com/zh/about
- DouYu game rankings: https://dohuya.com/zh/games?platform=douyu
- Huya game rankings: https://dohuya.com/zh/games?platform=huya
- DouYu Investor Relations: https://ir.douyu.com/Press-Releases/6a86c32bef90a235e9d8f471
- HUYA Investor Relations: https://ir.huya.com/2026-03-17-HUYA-Inc-Reports-Fourth-Quarter-and-Fiscal-Year-2025-Unaudited-Financial-Results-and-Announces-Cash-Dividend

### Twitch / 国际直播
- Streams Charts — Dota 2: https://streamscharts.com/games/dota-2
- Streams Charts — League of Legends: https://streamscharts.com/games/league-of-legends
- Streams Charts — Honor of Kings: https://streamscharts.com/games/honor-of-kings

### 电竞赛事
- Esports Charts — LoL: https://escharts.com/games/lol
- Esports Charts — The International 2026: https://escharts.com/news/team-spirit-makes-history-international-2026
- Esports Charts — Honor of Kings World Cup 2026: https://escharts.com/news/honor-kings-world-cup-2026-viewership

---

## 最后说明

这份文档故意保留了很多“未公开”。这是数据质量要求，而不是调研缺失。

对于当前 Dota 2 市场，最可靠、连续、可复核的玩家趋势数据是 Steam CCU；对于 LoL，Riot/Tencent 当前没有公开足够的绝对用户数；对于王者荣耀，官方 DAU/MAU里程碑相对明确；对于斗鱼/虎牙，目前公开数据只能可靠用于**热度和相对生态位置**，不能用于计算真实观众人数。
