# Dota 2 Game State Integration（GSI）调研

> 核验日期：2026-09-11  
> 目标：只记录 Dota 2 GSI 本身的已核验事实，不讨论任何具体产品方案。  
> 原则：区分 **Valve 官方确认**、**当前客户端运行证据**、**第三方实现/schema 核验**。第三方库的字段定义不等于 Valve 对未来版本的正式 API 合同。

---

## 1. 结论摘要

| 问题 | 结论 |
|---|---|
| Dota 2 是否真实存在 GSI | **是** |
| 是否是 Dota 2 客户端内置机制 | **是** |
| 是否由 Valve 明确承认 | **是**，Valve 官方更新直接使用 Game State Integration 名称 |
| 2026 年客户端是否仍加载 GSI | **是**，2026-08 的当前客户端日志仍出现 `Loading Game State Integration` |
| 是否属于 Steam Web API | **不是** |
| 是否是 WebSocket | **不是** |
| 数据如何送出 | Dota 2 客户端主动向配置的 URI 发 **HTTP POST** |
| 典型数据格式 | JSON |
| 外部程序是否需要轮询 Dota | **不需要**，由 Dota 主动推送 |
| 是否需要 Valve API Key | **未发现 GSI 需要 Valve 发放的 API Key** |
| 是否有 OAuth 授权流程 | **GSI 本身没有 Steam OAuth 授权流程** |
| 是否按调用量收费 | **未发现 Valve 对 GSI 设置调用计费或额度套餐** |
| 是否需要单独安装 GSI | **不需要**，能力在 Dota 2 客户端内 |
| 是否需要配置 | **需要** GSI `.cfg` 配置 |
| 是否需要启动参数 | **需要 `-gamestateintegration`**（Valve 2022 官方变更） |
| 接收数据的程序是否必须运行 | **必须**；否则没有 HTTP Listener 接收 POST |
| Playing 模式能否拿十名玩家完整数据 | **不能按这种方式理解**；当前实现明确以本地玩家数据为核心 |
| Spectating 模式数据是否更多 | **是**；当前 schema 可按玩家/队伍提供更多状态 |
| Valve 是否公开完整、版本化的 Dota 2 GSI Reference | **截至本次核验未找到** |

---

## 2. GSI 的官方身份

Valve 在 **2022-03-11 Dota 2 Update** 中明确写道：

> Game State Integration now requires the command-line option `-gamestateintegration` to function.

Valve 给出的理由是启用 GSI 可能产生逐帧性能影响。

官方来源：

- [Valve / Steam：Dota 2 Update - March 11th, 2022](https://store.steampowered.com/news/posts/?appids=570&enddate=1648162911&feed=steam_community_announcements)

因此可以确认：

1. Game State Integration 是 Valve/Dota 2 客户端正式存在的能力；
2. 它不是通过读取内存偶然发现的非官方漏洞；
3. 2022 年以后，Valve 明确要求使用 `-gamestateintegration` 启动参数。

### 2.1 2026 年仍存在的运行证据

Valve 官方 Dota 2 Gameplay Bug Tracker 中，2026-08-15 提交的当前客户端日志仍包含：

```text
Loading Game State Integration: gamestate_integration_logitech.cfg
```

同一日志显示当时 Dota 2 当前构建正在实际加载 GSI 配置。

来源：

- [ValveSoftware/Dota2-Gameplay #34380](https://github.com/ValveSoftware/Dota2-Gameplay/issues/34380)

这可以证明 **2026 年当前客户端仍保留并加载 GSI**。

需要注意：这是当前客户端事实，不是 Valve 对未来永久兼容性的承诺。

---

## 3. GSI 不是什么

GSI **不是 Steam Web API**。

传统云 API 更像：

```text
Application
    ↓ request
Valve Server
    ↓ response
Application
```

GSI 的方向相反：

```text
Dota 2 Client
    ↓ HTTP POST / JSON
Configured URI
    ↓
Receiving Application
```

因此更准确的定义是：

> **Dota 2 客户端本地的 Game State Integration 推送接口。**

它的行为类似 webhook：Dota 在本地运行时主动把游戏状态推给配置的接收端。

它不是：

- WebSocket；
- 外部程序不断轮询 Valve 的 REST API；
- 只知道 Match ID 就能从云端实时查询整场比赛的公共 API。

参考实现：

- [antonpup/Dota2GSI](https://github.com/antonpup/Dota2GSI)
- [MrBean355/dota2-gsi](https://github.com/MrBean355/dota2-gsi)

---

## 4. 基本工作机制

完整链路可以抽象为：

```text
Dota 2
  │
  │ 读取 gamestate_integration_*.cfg
  │
  ├── 决定发送 URI
  ├── 决定 buffer / throttle / heartbeat
  └── 决定订阅的数据类别
          │
          ▼
    HTTP POST (JSON)
          │
          ▼
   HTTP Listener / Server
          │
          ▼
      Game State
```

### 4.1 GSI 不需要另外安装

GSI 能力本身在 Dota 2 客户端中。

第三方程序真正需要做的是：

1. 准备 GSI 配置；
2. 启动一个能够接收 HTTP POST 的 Listener；
3. 解析 Dota 发来的 JSON；
4. 根据连续状态自行处理业务逻辑。

不存在一个必须另外安装的 Valve GSI Runtime / Driver / Service。

---

## 5. 启用方式

### 5.1 启动参数

Valve 官方确认，自 2022-03-11 起 GSI 需要：

```text
-gamestateintegration
```

官方来源：

- [Dota 2 Update - March 11th, 2022](https://store.steampowered.com/news/posts/?appids=570&enddate=1648162911&feed=steam_community_announcements)

### 5.2 配置目录

当前公开实现及 Valve Bug Tracker 中的实际配置均使用：

```text
dota 2 beta/game/dota/cfg/gamestate_integration/
```

配置文件名通常采用：

```text
gamestate_integration_*.cfg
```

例如：

```text
gamestate_integration_example.cfg
```

### 5.3 典型配置结构

下面是当前公开实现和实际问题报告中反复出现的配置结构。它是 **当前可核验的实际用法**，不是 Valve 发布的稳定版本化 Dota 2 API Schema：

```cfg
"Dota 2 Integration Configuration"
{
    "uri"           "http://127.0.0.1:6000/"
    "timeout"       "5.0"
    "buffer"        "0.1"
    "throttle"      "0.1"
    "heartbeat"     "30.0"

    "data"
    {
        "provider"      "1"
        "map"           "1"
        "player"        "1"
        "hero"          "1"
        "abilities"     "1"
        "items"         "1"
        "buildings"     "1"
        "draft"         "1"
        "wearables"     "1"
        "events"        "1"
    }

    "auth"
    {
        "token" "example-token"
    }
}
```

公开证据：

- [ValveSoftware/Dota2-Gameplay #27260：2025 年实际 GSI 配置示例](https://github.com/ValveSoftware/Dota2-Gameplay/issues/27260)
- [BrightGir/dota-ai-coach：实际 GSI listener/config 示例](https://github.com/BrightGir/dota-ai-coach)
- [MrBean355/dota2-gsi](https://github.com/MrBean355/dota2-gsi)

---

## 6. `buffer`、`throttle`、`heartbeat` 与刷新率

GSI 配置中存在：

- `buffer`
- `throttle`
- `heartbeat`
- `timeout`

因此不应把 GSI 简化成“Valve 官方固定 10Hz API”。

更准确的理解是：

- `buffer`：把一段时间内的状态变化聚合；
- `throttle`：约束两次状态发送之间的节奏；
- `heartbeat`：即使没有明显变化，也按配置发送心跳；
- `timeout`：与请求/连接等待相关的配置。

网上经常出现 `throttle = 0.1`，这只能证明配置允许使用该值，**不能解释成 Valve 对所有环境承诺固定每秒 10 次更新**。

---

## 7. 通信与认证

### 7.1 HTTP POST

当前 Dota GSI 实现普遍采用一个 HTTP Server / Listener 接收 Dota 发出的 POST。

典型过程：

```text
Dota
  ↓ POST JSON
localhost:port
  ↓
Parser
```

`BrightGir/dota-ai-coach` 的实际架构就是：

```text
Dota 2
→ HTTP POST (JSON)
→ GSI Handler
→ Parser
→ GameState Store
```

来源：

- [BrightGir/dota-ai-coach](https://github.com/BrightGir/dota-ai-coach)

### 7.2 `auth.token` 不是 Valve API Key

GSI 配置可以带 `auth` 数据，例如：

```cfg
"auth"
{
    "token" "example-token"
}
```

该 token 是配置方用于接收端校验的自定义数据，不是 Valve Developer Portal 发放的 OAuth Token / API Key。

因此：

- 不需要为 GSI 向 Valve 申请 API Key；
- GSI 本身不是 Steam OAuth 授权流程；
- 接收端可以自行设计认证 token。

---

## 8. 是否免费、是否有调用额度

截至 2026-09-11 本次核验：

- 未发现 Valve 为 Dota 2 GSI 设置按请求计费；
- 未发现 GSI API 套餐；
- 未发现调用额度申请流程；
- 未发现必须申请 Valve API Key 才能使用的要求。

这与它的技术形态一致：GSI 是 **本机 Dota 2 客户端向配置 URI 推送状态**，而不是 Valve 提供的按调用量计费云 API。

严谨表述应为：

> **当前 GSI 不是一个 Valve 按 API 调用次数计费的服务。**

这不是对未来政策的保证。

---

## 9. 接收程序是否需要一直运行

**需要。**

配置文件只告诉 Dota：

> 数据应该送到哪里。

真正接收状态的 HTTP Listener 必须在需要 GSI 数据时处于运行状态。

```text
GSI cfg
   ↓ defines URI
Dota 2 ──────────→ http://127.0.0.1:PORT
                        ↓
                  Receiving App
```

如果接收程序关闭，则没有服务处理这些 POST。

这不代表 GSI 本身需要单独常驻；GSI 是 Dota 2 的客户端能力，常驻的是**消费数据的第三方应用**。

---

## 10. Playing 与 Spectating 的关键边界

这是 Dota 2 GSI 最容易被误解的部分。

长期维护的 `antonpup/Dota2GSI` 明确说明：

- **正常 Playing**：只暴露本地玩家级别的信息；
- **Spectating**：可以暴露所有玩家的更多信息。

来源：

- [Dota2GSI README — About Game State Integration](https://github.com/antonpup/Dota2GSI/blob/master/README.md)

`MrBean355/dota2-gsi` 当前类型模型也明确把 Playing 与 Spectating 分成不同 GameState。

来源：

- [MrBean355/dota2-gsi](https://github.com/MrBean355/dota2-gsi)
- [生成文档](https://mrbean355.github.io/dota2-gsi/)

### 10.1 Playing 模式

当前实现中，核心对象是单个本地玩家相关状态，例如：

```text
player
hero
items
abilities
map
buildings
events
provider
wearables
```

关键点不是字段数量，而是：

> `player` / `hero` / `items` / `abilities` 的核心语义是 **local player**，不是十名玩家的完整数组。

因此不能把 Playing GSI 描述成：

```text
players[10]
heroes[10]
items[10]
abilities[10]
```

### 10.2 Spectating 模式

当前 Spectating schema 则存在按 Player ID / Team 组织的多人结构，例如：

```text
players
heroes
items
abilities
buildings
map
draft
events
```

所以：

> **Playing 和 Spectating 是两个明显不同的数据可见范围。**

---

## 11. “玩家肉眼可见”不等于“GSI 一定提供”

这是必须明确的原则。

例如正常比赛中，玩家可能通过 Dota UI 看到部分队友或当前可见敌方的信息，但这不能推出：

> GSI 会把屏幕上所有可见信息全部结构化输出。

当前 Playing schema 没有一个可据此宣称：

```text
enemy[0].items
enemy[1].items
...
```

的完整敌方装备数组。

因此凡是需要判断“某个屏幕可见信息 GSI 有没有”的问题，都应以 **真实 payload / 当前 schema** 为准，而不能按 UI 可见性推断。

---

## 12. Playing 模式当前可核验的典型数据

以下不是 Valve 的版本化正式 API 合同，而是当前主流 Dota GSI schema 中可直接核验的数据。

### 12.1 本地 Player

典型字段包括：

- 玩家标识 / Steam 相关标识；
- Team / Slot；
- Kills / Deaths / Assists；
- Last Hits / Denies；
- Kill Streak；
- Gold；
- Reliable / Unreliable Gold；
- GPM / XPM；
- Activity 等。

### 12.2 本地 Hero

典型字段包括：

- Hero ID / Hero Name；
- Level / XP；
- Position；
- Health / Max Health；
- Mana / Max Mana；
- Alive / Respawn；
- Buyback Cost / Cooldown；
- Aghanim's Scepter / Shard；
- Stun / Silence / Hex / Disarm / Break / Mute / Magic Immune / Smoke 等状态。

### 12.3 Abilities

典型字段包括：

- 技能标识；
- 技能等级；
- Cooldown；
- 当前是否可施放；
- Passive / Ultimate；
- Charges / Charge Cooldown 等。

### 12.4 Items

当前实现可区分本地玩家的多类物品位置，例如：

- Inventory / Backpack；
- Stash；
- Teleport；
- Neutral Item。

单个 item 可包含名称、cooldown、charges、当前是否可用等状态。

### 12.5 Map / Match State

典型字段包括：

- Match ID；
- Clock / Game Time；
- Radiant / Dire Score；
- Game/Match State；
- Pause；
- Day / Night；
- Winning Team 等。

字段参考：

- [MrBean355/dota2-gsi generated API docs](https://mrbean355.github.io/dota2-gsi/)

---

## 13. Events：原始事件与“派生事件”需要分开

GSI 配置可以订阅 `events`。

但第三方库经常进一步比较连续两份 GameState，自行派生更适合业务使用的事件，例如：

```text
InventoryItemAdded
TowerDestroyed
TimeOfDayChanged
...
```

这些高层事件不能全部写成：

> “Valve 原始 GSI JSON 直接发了这个事件。”

`antonpup/Dota2GSI` 明确说明，其一部分 Game Events 是通过监听连续 GameState、比较前后状态生成的。

来源：

- [antonpup/Dota2GSI README](https://github.com/antonpup/Dota2GSI/blob/master/README.md)

因此在实现时应区分：

```text
Valve / Dota 原始 payload
        ↓
State diff / Event normalizer
        ↓
业务高层事件
```

---

## 14. GSI 与读取游戏内存不是同一技术路线

GSI 的方向是：

```text
Dota 2
→ Valve 选择暴露的状态
→ HTTP POST
→ 外部程序
```

而内存读取 / DLL 注入 / 客户端内部 introspection 是：

```text
第三方程序
→ 主动读取或修改 Dota 客户端内部状态
```

两者不能混为一谈。

`antonpup/Dota2GSI` 对 GSI 的定义也强调：GSI 用来暴露 Valve 决定公开的当前状态，不需要读取游戏内存。

来源：

- [antonpup/Dota2GSI](https://github.com/antonpup/Dota2GSI)

这也是为什么：

> **GSI 可获得的信息边界由 Valve 决定。**

不能因为某条信息技术上存在于 Dota 客户端内存里，就假设 GSI 会提供。

---

## 15. Valve 官方文档现状

截至 2026-09-11，本次检索没有发现 Valve 提供一套类似成熟商业 API 的：

```text
Dota 2 Game State Integration vX.X
Official API Reference
Versioned JSON Schema
Compatibility / SLA
```

2025 年 Valve 官方 Gameplay Bug Tracker 中，开发者自己也明确提到找不到官方完整文档，只能参考第三方项目。

来源：

- [ValveSoftware/Dota2-Gameplay #27260](https://github.com/ValveSoftware/Dota2-Gameplay/issues/27260)

因此 GSI 应被理解为：

> **Valve 正式存在并持续保留在 Dota 2 客户端中的 Game State Integration 机制，但不是一个具有完整官方 Dota Reference、长期固定 Schema 和兼容性 SLA 的公共 Web API 产品。**

实际开发时应采用：

1. 宽容 JSON 解析；
2. 未知字段向前兼容；
3. 保存原始 payload；
4. 每次 Dota 大版本后做真实对局回归；
5. 不把第三方 schema 当作 Valve 永久合同。

---

## 16. 已确认与未确认边界

### 已确认

- GSI 是 Dota 2 内置机制；
- Valve 官方承认 GSI；
- 2026 年客户端仍加载 GSI；
- 2022 年后要求 `-gamestateintegration`；
- 使用 `.cfg` 配置 URI 与数据类别；
- Dota 主动 HTTP POST JSON；
- 不属于 Steam Web API；
- 不属于 WebSocket；
- 没有 GSI 专用 Valve API Key / OAuth 流程；
- 未发现调用计费；
- 接收端程序必须运行；
- Playing 和 Spectating 数据范围不同；
- Playing 核心是本地玩家信息，而不是十名玩家的完整状态；
- 第三方库可以在原始 GameState 上派生高层事件。

### 不应在没有真实 payload 验证时宣称

- “所有玩家肉眼看到的信息 GSI 都能拿到”；
- “正常 Playing 可以直接得到敌方五人的完整装备/等级/技能/CD”；
- “GSI 官方固定 10Hz”；
- “Valve 保证某字段永久存在”；
- “第三方库暴露的所有 Event 都是 Valve 原始 Event”；
- “只凭 Match ID 可以远程从 GSI 查询任意实时比赛”。

---

## 17. 主要参考资料

### Valve / 官方运行证据

1. [Dota 2 Update - March 11th, 2022](https://store.steampowered.com/news/posts/?appids=570&enddate=1648162911&feed=steam_community_announcements)  
   Valve 明确要求 `-gamestateintegration`。

2. [ValveSoftware/Dota2-Gameplay #34380](https://github.com/ValveSoftware/Dota2-Gameplay/issues/34380)  
   2026-08 当前客户端日志仍显示 `Loading Game State Integration`。

3. [ValveSoftware/Dota2-Gameplay #27260](https://github.com/ValveSoftware/Dota2-Gameplay/issues/27260)  
   2025 年当前 GSI 使用实例；同时反映 Dota 2 缺少完整官方字段 Reference。

### 当前第三方实现 / Schema

4. [antonpup/Dota2GSI](https://github.com/antonpup/Dota2GSI)  
   长期维护的 C# Dota 2 GSI 实现；清楚区分 Playing / Spectating 数据边界，并实现状态差异事件。

5. [MrBean355/dota2-gsi](https://github.com/MrBean355/dota2-gsi)  
   Kotlin/JVM GSI 库；当前类型化 Playing / Spectating schema。

6. [MrBean355/dota2-gsi generated docs](https://mrbean355.github.io/dota2-gsi/)  
   用于字段级核验。

7. [BrightGir/dota-ai-coach](https://github.com/BrightGir/dota-ai-coach)  
   2026 年实际以 Go 接收 Dota GSI、解析 GameState 的应用实现。

---

## 18. 最终结论

Dota 2 GSI 可以准确描述为：

> **Valve/Dota 2 客户端内置的本地 Game State Integration 机制。启用后，Dota 依据配置文件主动通过 HTTP POST 向指定 URI 推送 JSON 游戏状态。它无需 Steam Web API Key，也不是 WebSocket 或云端轮询 API。Playing 与 Spectating 的数据可见范围明显不同；正常 Playing 以本地玩家状态为核心，不能把它理解成十名玩家完整实时数据库。由于 Valve 没有提供完整、版本化的 Dota 2 GSI Reference，真实客户端 payload 与当前 schema 应作为字段实现的最终校验依据。**
