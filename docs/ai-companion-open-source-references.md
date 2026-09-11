# AI 游戏陪玩：开源技术参考与可复用模块调研

> 核验日期：2026-09-11  
> 范围：GitHub 开源技术参考。目标不是寻找一个“完全相同”的成品，而是拆解 AI 游戏陪玩所需模块，寻找可直接复用、可做 PoC、或值得借鉴架构的项目。  
> 注意：本文中的许可证状态以 2026-09-11 当前仓库为准。**代码许可证、模型许可证、主播声音/肖像/人格授权是三件不同的事。**

---

## 1. 结论摘要

目前没有发现一个成熟开源项目能够完整覆盖：

```text
Dota 2 GSI
+ 游戏事件理解
+ 主动发言决策
+ 长期人格/关系记忆
+ 实时语音
+ 主播声音克隆
+ Windows 游戏客户端
```

但已经找到多组高度重叠的项目，意味着第一版技术上不需要从零开始。

最重要的五个参考项目是：

| 项目 | 最有价值的部分 | 与目标重叠度 | 当前许可状态 |
|---|---|---:|---|
| [`BrightGir/dota-ai-coach`](https://github.com/BrightGir/dota-ai-coach) | Dota GSI → GameState → RAG → LLM → Overlay | **极高：Dota 半边** | MIT |
| [`chasmlol/chasm`](https://github.com/chasmlol/chasm) | 游戏事件 → 人格 → 记忆 → 主动反应 → Voice → Agent | **极高：陪伴行为半边** | **当前未声明许可证** |
| [`Open-LLM-VTuber/Open-LLM-VTuber`](https://github.com/Open-LLM-VTuber/Open-LLM-VTuber) | 实时语音、打断、Persona、TTS/STT、Live2D | 高 | MIT；Live2D 示例资产另有许可 |
| [`Wintersta7e/AiGameCompanion`](https://github.com/Wintersta7e/AiGameCompanion) | Windows Overlay、Steam 发现、一键启动、Hotkey | 高：桌面壳 | MIT |
| [`CodeNeuron58/Yumii`](https://github.com/CodeNeuron58/Yumii) | Windows AI Companion、实时语音、记忆、安装封装 | 高：Companion 壳 | MIT |

整体技术判断：

> **基础设施大部分已经有可复用方案。真正需要自主设计、也最可能形成产品差异的核心，是 Game Event Model、Speech Policy 和 Companion Behavior。**

即：

1. **Dota Event Model**：原始 GSI 到“这一刻真正发生了什么”；
2. **Speech Policy**：AI 此刻该不该主动说话；
3. **Companion Behavior**：如果要说，结合人格、关系、历史、当前事件后应该怎么说。

---

# 2. 产品技术拆解

AI 游戏陪玩可以拆为七个主要模块。

```text
① 游戏数据接入
        ↓
② Game State / Event Model
        ↓
③ 主动发言决策 Speech Policy
        ↓
④ Persona + 游戏/主播知识
        ↓
⑤ 用户长期记忆 / 关系记忆
        ↓
⑥ 实时语音输入输出
        ↓
⑦ Windows 客户端 / Overlay / Launcher
```

进一步可以加入：

```text
⑧ Voice Cloning / TTS
⑨ Telemetry / Debug / Raw Event Replay
⑩ 安装、更新、模型下载与本地服务生命周期
```

### 2.1 各模块的开源成熟度

| 模块 | 开源成熟度 | 是否建议自研 |
|---|---:|---|
| Dota GSI 接入 | 高 | 不需要从零造 |
| GSI JSON → Typed State | 高 | 可复用/参考 |
| 原始 State → 陪玩语义事件 | 中 | **建议自主实现** |
| 主动发言时机 | 中低 | **核心自研** |
| LLM Provider 接入 | 很高 | 不自研 |
| RAG | 很高 | 不自研底层 |
| Persona Prompt/Character Card | 高 | 数据和策略自研 |
| 长期记忆 | 很高 | 可采用 Mem0 / 简化自建 |
| VAD/STT/TTS Pipeline | 很高 | 不自研底层 |
| Voice Clone | 高 | 直接评估成熟 TTS |
| Windows Overlay | 中高 | 可直接参考成熟实现 |
| Steam 游戏发现 / Launcher | 中高 | 可直接参考 |
| 安装/更新/后台服务 | 中高 | 可参考 Yumii/Tauri 模式 |

---

# 3. 最接近完整目标的项目

## 3.1 BrightGir/dota-ai-coach

仓库：<https://github.com/BrightGir/dota-ai-coach>

### 当前状态

截至 2026-09-11：

- 语言：Go；
- Windows；
- License：MIT；
- 2026 年创建；
- 项目规模较小，属于真实可运行项目而不是成熟生产平台。

### 已实现能力

README 中明确实现：

- Dota 2 GSI；
- 本地 HTTP GSI Listener；
- JSON → `GameState`；
- Thread-safe State Store；
- Hero / Items / Abilities 等当前状态；
- Gemini / OpenRouter；
- RAG；
- 英雄、技能、物品知识库；
- BERT Embedding + Chroma；
- 自动 Advice；
- 游戏内 Overlay；
- Hotkey；
- 用户主动提问。

其核心链路已经是：

```text
Dota 2
   ↓ HTTP POST / JSON
GSI Handler
   ↓
Parser
   ↓
GameState Store
   ↓
Prompt Builder
   ↓
RAG
   ↓
LLM
   ↓
Overlay
```

### 与 AI 陪玩的重叠

**非常高。**

它已经解决了“如何让 AI 获得当前 Dota 局势”这一整侧工程问题。

缺失的是：

- 长期 Persona；
- 用户关系记忆；
- 实时语音；
- Voice Clone；
- 真正的 Event-driven Speech Policy；
- 陪伴式主动反应。

### 特别需要注意

它的自动建议目前主要是：

```text
每 N 秒
→ 读取当前状态
→ 调 LLM
→ 给建议
```

这适合作为 Coach，但不适合直接作为自然陪玩行为。

陪玩更应该：

```text
发生重要事件
→ 决定是否值得说
→ 再调用 LLM
```

### 推荐用途

**第一优先级参考。**

适合：

- 直接阅读 GSI Handler；
- Parser；
- GameState Store；
- Prompt Builder；
- RAG；
- Windows Overlay 组织方式。

MIT 允许较宽松的商业复用，但正式产品仍需保留相应版权/许可声明。

---

## 3.2 chasmlol/chasm

仓库：<https://github.com/chasmlol/chasm>

### 为什么它非常重要

它不是 Dota 项目，而是一个：

> **game-agnostic agentic NPC backend**

但其架构与 AI 游戏陪玩高度重合：

```text
GAME
 ↓
Bridge
 ↓
Game State / Events
 ↓
Character Persona
 ↓
Memory / Retrieval
 ↓
LLM
 ↓
Streaming TTS / Voice Clone
 ↓
主动反应 / Actions
```

### 已实现的关键能力

README 当前明确描述：

- Game Bridge；
- Headless HTTP API；
- live gamestate；
- Character Card / Lore / Persona；
- 本地或云 LLM；
- Push-to-talk；
- STT；
- Streaming TTS；
- 每角色 Voice Clone；
- Relationships Ledger；
- Game Event Log；
- Witness Memory；
- Event-triggered Reactions；
- Rate Limiting；
- Persistent Memory；
- Retrieval；
- 游戏运行时自动启动本地 AI 服务，游戏退出后自动关闭。

### 最值得借鉴：事件驱动行为

`chasm` 的思想不是：

```text
每一帧问 LLM：
“你现在想不想讲话？”
```

而是：

```text
Game Event
   ↓
确定性 Trigger
   ↓
Witness / Context Gate
   ↓
Cooldown
   ↓
满足条件
   ↓
触发 Agent Turn
   ↓
LLM 决定具体怎么表达
```

它的 self-improving “skills” 甚至把：

```text
事件触发是否命中
```

设计成**纯机械检查**，只有命中以后才让 Agent 进行自由生成。

这一模式非常值得用于实时游戏陪玩，因为：

- 成本可控；
- 延迟低；
- 不会每帧调用 LLM；
- 发言频率可以精确限制；
- 事件判断与人格表达分离；
- 便于测试。

### License 风险

**重要：截至 2026-09-11，GitHub 仓库元数据没有声明 License，根目录也未找到 LICENSE 文件。**

因此不能把它当成 MIT/Apache/AGPL 等已有明确授权的代码库。

建议：

- 可以研究其公开架构、交互思想和工程模式；
- **未经作者明确许可，不应直接复制其代码进入闭源商业产品。**

### 推荐用途

**最高优先级架构参考，但不作为当前可直接复制的商业代码依赖。**

---

## 3.3 Open-LLM-VTuber

仓库：<https://github.com/Open-LLM-VTuber/Open-LLM-VTuber>

### 定位

一个成熟度明显高于普通个人项目的本地 AI Companion / AI VTuber 框架。

核心定位：

> hands-free voice interaction + voice interruption + Live2D + LLM

### 可参考能力

项目覆盖：

- 实时语音；
- Voice Interruption / Barge-in；
- STT Provider Adapter；
- TTS Provider Adapter；
- LLM Provider；
- Persona；
- 主动/陪伴交互模式；
- Live2D；
- 本地运行；
- 多平台。

### 与目标的关系

可以把它理解成：

```text
Open-LLM-VTuber
= 陪伴/语音/角色这一半
```

而：

```text
Dota GSI Adapter
= 游戏实时状态这一半
```

二者在概念上非常容易组合。

### License

仓库实际 `LICENSE` 文件是 **MIT License**。

但其 LICENSE 同时明确：

> Live2D sample models 由独立 `LICENSE-Live2D.md` 管理。

所以：

- 主项目代码：MIT；
- Live2D 示例模型/素材：**不能简单按 MIT 一并处理**。

### 推荐用途

重点研究：

- 实时语音 Pipeline；
- interruption；
- TTS/STT provider abstraction；
- Persona；
- Companion loop。

---

## 3.4 Wintersta7e/AiGameCompanion

仓库：<https://github.com/Wintersta7e/AiGameCompanion>

### 当前状态

- Rust；
- Tauri 2；
- Svelte 5；
- Windows；
- License：MIT；
- 2026 年仍在活跃开发。

### 最有价值的能力

README 明确实现：

- 外部透明 Overlay；
- Always-on-top；
- Idle 时 click-through；
- 全局 Hotkey；
- Screenshot Vision；
- Windows.Graphics.Capture；
- Steam Library 自动发现；
- Steam CDN Cover；
- One-click Launch；
- Tray；
- Launch-on-startup；
- 外部进程监测；
- API Key 使用 Windows Credential Manager；
- Streaming AI response。

### 一个非常有价值的工程决策

作者早期版本曾采用 DLL Injection / Renderer Hook，后续**主动删除这一路径**，原因是：

- 脆弱；
- 容易像 anti-cheat 会标记的行为；
- 一个 Companion 不值得冒这种风险。

当前改为：

```text
独立透明 Windows Window
→ 由 Windows compositor 叠在游戏上
→ 不注入游戏
→ 不 Hook Graphics API
```

这一路径更适合 Companion 产品。

### 局限

该项目 README 明确把 competitive / kernel anti-cheat titles 列为自身 non-goal，因此不能把“它现成支持 Dota 竞技环境”当成事实。

值得借的是：

> **Windows 客户端、Steam Launcher、Overlay、Hotkey、进程生命周期的工程设计。**

### 推荐用途

Windows 产品壳第一参考。

---

## 3.5 CodeNeuron58/Yumii

仓库：<https://github.com/CodeNeuron58/Yumii>

### 定位

Voice-first Desktop AI Companion。

### 已实现

README 当前明确写明：

- Windows-first；
- Tauri Desktop；
- Silero VAD；
- Whisper；
- Barge-in；
- Kokoro / ElevenLabs / CAMB.ai TTS；
- Groq / Ollama / OpenAI / Anthropic；
- persistent personality；
- SQLite persistent memory；
- sessions / facts / transcripts / summaries / checkpoints；
- 本地优先；
- 一条 PowerShell 命令完成安装；
- 自动准备 Python 环境、Backend、Desktop App、Start Menu。

### 特别值得借鉴的地方

它给出了一个现实的答案：

> **如何把 Python AI Runtime + 本地语音模型 + Tauri 桌面 UI 包装成普通 Windows 用户可以安装的产品。**

这比单独的 STT/TTS 算法更有参考价值。

### 当前缺口

Yumii 当前 Roadmap 把：

- screen seeing；
- proactiveness

列在后续计划中。

因此它不能直接解决“基于游戏事件主动讲话”。

### License

MIT。

### 推荐用途

- Windows Installer；
- Local Backend 生命周期；
- Voice Pipeline；
- Persistent Memory；
- Desktop Companion packaging。

---

# 4. Dota GSI 专项开源参考

## 4.1 BrightGir/dota-ai-coach

上文已述，是目前找到的**完整 Dota + GSI + LLM 应用参考**。

优先级最高。

---

## 4.2 antonpup/Dota2GSI

仓库：<https://github.com/antonpup/Dota2GSI>

### 定位

长期存在的 C# Dota 2 GSI 库。

### 有价值的能力

- GSI HTTP Listener；
- JSON → typed state；
- Playing / Spectating 边界；
- 对连续 GameState 做 Diff；
- 生成更好用的高层事件。

高层事件设计尤其值得参考：

```text
Raw Game State
    ↓
Previous vs Current
    ↓
InventoryItemAdded / TowerDestroyed / ...
```

这与 AI 陪玩 Event Model 十分接近。

### 维护情况

项目历史较长，最后代码 push 比 2026 新项目早，因此更适合：

> schema / event-diff 设计参考，而不是默认认定为“最新唯一正确实现”。

### License 注意

GitHub 当前无法把仓库识别为标准 SPDX License。

仓库 `LICENSE.md` 中存在 MIT 风格许可文本，但文件开头写的是：

```text
JSON.Net is Copyright (c) 2007 James Newton-King
```

因此其许可文本到底覆盖哪些仓库源码，不应仅凭 GitHub 页面武断判断。

**建议：正式复制代码之前单独核实许可证归属。**

---

## 4.3 MrBean355/dota2-gsi

仓库：<https://github.com/MrBean355/dota2-gsi>

### 当前状态

- Kotlin/JVM；
- 2026 年仍有代码更新；
- License：Apache-2.0；
- 有生成的 API Docs。

### 最大价值

不是一定要把 JVM 带进最终产品，而是：

> **它是目前非常好用的 Dota GSI 当前 typed schema 参考。**

尤其适合核对：

- PlayingGameState；
- SpectatingGameState；
- Player；
- Hero；
- Items；
- Abilities；
- Map；
- Events。

生成文档：

<https://mrbean355.github.io/dota2-gsi/>

### 推荐用途

Schema reference / regression reference。

---

# 5. Game Event Model：真正需要自主实现的第一层

原始 GSI 不应该直接进入 LLM。

推荐中间增加一个语义 Event Layer：

```text
Raw GSI
   ↓
Typed State
   ↓
Previous State vs Current State
   ↓
Semantic Event Normalizer
```

例如最终得到：

```text
PLAYER_DIED
PLAYER_GOT_KILL
MULTI_KILL_WINDOW
ITEM_PURCHASED
ITEM_COMPLETED
LEVEL_UP
ULTIMATE_READY
BUYBACK_AVAILABLE
LOW_HP_ESCAPE
LAST_HIT_MILESTONE
EARLY_GAME_LH_BEHIND
EARLY_GAME_LH_AHEAD
KILL_STREAK_STARTED
KILL_STREAK_ENDED
TEAM_SCORE_SWING
TOWER_DESTROYED
MATCH_STARTED
MATCH_ENDED
```

### 可参考项目

- `antonpup/Dota2GSI`：state diff → high-level event；
- `chasm`：game event log / event trigger；
- `BrightGir/dota-ai-coach`：GSI parser / GameState store。

### 为什么建议自研

因为：

> **“什么游戏事件对陪玩有意义”本身就是产品定义，而不是基础设施。**

Coach 关心的事件和 Companion 关心的事件不同。

例如：

```text
死亡
```

对 Coach 是：

> 分析死亡原因。

对 Companion 可能是：

> 吐槽、安慰、接上一分钟前刚说过的话，或者完全保持沉默。

---

# 6. Speech Policy：真正需要自主实现的第二层

这是目前没有找到可直接拿来即用的 Dota 专用开源模块。

推荐结构：

```text
Semantic Event
      ↓
Event Importance
      ↓
Speech Policy
      │
      ├─ 是否值得说？
      ├─ 距离上次说话多久？
      ├─ 是否同类事件刚说过？
      ├─ 用户是否正在讲话？
      ├─ AI 是否正在讲话？
      ├─ 当前是否高强度操作？
      ├─ 是否存在更高优先级事件？
      ├─ 本局总发言密度是否过高？
      ├─ 用户偏好话多还是话少？
      └─ 当前关系/情绪状态？
      ↓
SPEAK / SILENCE
      ↓
LLM 生成具体内容
```

核心原则：

> **LLM 负责“说什么”，确定性 Policy 尽量负责“要不要说”。**

不建议：

```text
每 100ms / 每次 GSI 更新
→ 把全部状态发给 LLM
→ 问“现在要不要说话？”
```

原因：

- 成本高；
- 延迟高；
- 输出不稳定；
- 难以控制说话频率；
- 难以自动化测试；
- 容易让 AI 变成一直说话的干扰源。

---

## 6.1 chasm 的事件触发机制

`chasm` 是目前最值得借鉴的设计：

```text
事件发生
→ mechanical trigger matching
→ witness/context check
→ per-skill cooldown
→ agent turn
```

只有触发 Agent Turn 后 LLM 才参与。

这是一种非常合理的 realtime companion 架构。

---

## 6.2 Shikigami-Protocol

仓库：<https://github.com/Shikigami-Lab/Shikigami-Protocol>

当前项目描述明确强调：

- emotion × energy × affinity state machine；
- memory pipeline；
- background reflection；
- proactive speech after silence；
- “persona bones”，而不是只靠 prompt。

它的价值在于：

> 主动发言不仅可以看外部事件，也可以考虑 AI 自己的内部关系/情绪状态。

### License

AGPL-3.0。

因此：

- 很适合读架构；
- 闭源商业产品若直接采用/修改其代码，需要认真评估 AGPL 义务。

---

# 7. Persona / Companion Behavior

Persona 不应只实现为：

```text
system_prompt = "你现在是某主播"
```

更完整的输入结构应考虑：

```text
Character Core
+ Speaking Style
+ Known Catchphrases
+ Allowed / Forbidden Topics
+ Game Opinions
+ Current Relationship State
+ User Profile
+ Recent Shared Events
+ Relevant Long-term Memories
+ Current Dota Event
+ Current Game Context
+ Retrieved Streamer/Game Knowledge
```

最终：

```text
Speech Policy = 要不要说
Companion Behavior = 以什么状态说
LLM = 最终自然语言实现
```

---

## 7.1 Open-LLM-VTuber

适合研究：

- Persona；
- Character Prompt；
- Voice-first interaction；
- TTS/STT adapters。

MIT（Live2D 示例素材另算）。

---

## 7.2 Noema

仓库：<https://github.com/HappyFox001/Noema>

当前定位：

> voice-first desktop AI companion with memory, emotion, tools, and plugins

可参考：

- Emotional Layer；
- Memory；
- Voice；
- Tool/Plugin；
- Companion architecture。

### License

AGPL-3.0。

推荐：

> **架构参考，不作为闭源产品随意复制的依赖。**

---

## 7.3 Personality Machine

仓库：<https://github.com/emotion-machine-org/personality-machine>

当前定位：

> Open-source platform for AI companions with persistent relationships

### 优点

它把重点直接放在：

> Companion 与 User 之间的持久关系。

这与一次性 chatbot session 的数据模型不同。

### 当前成熟度

项目于 2026-08 才创建，当前规模很小。

所以：

- 数据模型/概念值得看；
- 不建议现在将其作为关键生产依赖。

### License

MIT。

---

# 8. 长期记忆

## 8.1 Mem0

仓库：<https://github.com/mem0ai/mem0>

### 当前定位

> Memory Layer for AI Agents

特点：

- persistent context；
- agent memory；
- production-oriented；
- 活跃社区；
- Python。

### License

Apache-2.0。

### 推荐用途

如果希望快速做 PoC，而不是先自己研究一整套 Memory Architecture：

> **Mem0 是很强的默认候选。**

可以保存：

- 用户游戏偏好；
- 常用英雄；
- 对话偏好；
- 已发生的重要共同事件；
- 用户长期梗；
- 关系事实；
- 可检索 episodic memories。

但正式产品仍建议把**游戏事件数据库**和**LLM 自由记忆**分开，不要把所有状态都塞进向量记忆。

---

## 8.2 Letta

仓库：<https://github.com/letta-ai/letta>

定位：

> Platform for stateful agents with advanced memory

### License

Apache-2.0。

### 判断

Letta 比纯 Memory SDK 更接近完整 stateful-agent platform。

优点：

- stateful agent；
- advanced memory；
- persistent agent architecture。

缺点：

- 对只做 MVP 可能偏重。

适合第二阶段评估，而不是默认第一选择。

---

## 8.3 Yumii 自有 SQLite Memory

如果第一版目标是可控、可解释，Yumii 的思路同样值得参考：

```text
SQLite
├─ sessions
├─ facts
├─ transcript + FTS
├─ summaries
└─ checkpoints
```

对于早期 MVP，这种简单、可 Debug 的结构有时比直接接复杂 Agent Memory Framework 更合适。

---

# 9. 实时语音 Pipeline

## 9.1 Pipecat

仓库：<https://github.com/pipecat-ai/pipecat>

### 当前状态

- 大型活跃项目；
- 2026-09 仍持续提交；
- Python；
- License：BSD-2-Clause。

### 定位

> Open Source framework for voice agents, multimodal apps, and realtime AI

它适合承担：

```text
Audio Input
  ↓
VAD / Turn Detection
  ↓
STT
  ↓
LLM
  ↓
TTS
  ↓
Audio Output
```

以及：

- Streaming；
- interruption / barge-in；
- provider abstraction；
- realtime pipelines。

### 判断

如果希望减少自己维护实时语音工程的成本，Pipecat 是当前最值得认真评估的基础框架之一。

---

## 9.2 Silero VAD

仓库：<https://github.com/snakers4/silero-vad>

### 当前状态

- 成熟 VAD；
- 2026 年持续更新；
- License：MIT。

### 用途

用于检测：

> 用户什么时候开始/结束说话。

如果交互主要使用 Push-to-talk，VAD 不是唯一方案；如果未来支持 hands-free voice / interruption，它就非常重要。

---

## 9.3 STT

从现有项目可以看到成熟组合包括：

- Whisper 系列；
- Groq Whisper；
- Vosk；
- 其他 cloud STT。

`Open-LLM-VTuber`、`Yumii`、`Pipecat` 都已经提供现成的 STT 集成模式。

因此 STT 本身不是需要从零研发的核心技术。

---

# 10. TTS / Voice Cloning

## 10.1 GPT-SoVITS

仓库：<https://github.com/RVC-Boss/GPT-SoVITS>

### 当前状态

- 大型活跃 Voice Clone/TTS 项目；
- 2026 年仍持续更新；
- Python；
- License：MIT。

项目定位直接强调：

> 少量语音数据即可进行 few-shot voice cloning。

### 为什么优先

对于中文主播语音场景，它值得作为第一批基准测试对象：

- 中文生态成熟；
- Voice Clone；
- 本地部署；
- 大量社区实践；
- MIT 代码许可。

### 重要法律边界

**MIT 只授权 GPT-SoVITS 代码，不授权任何真人声线。**

如果使用具体主播声音，仍然必须解决：

- 本人明确授权；
- 声音模型训练权；
- 商业使用权；
- 生成内容范围；
- 是否允许二次模型训练/导出；
- 终止合作后的模型处理。

---

## 10.2 OpenVoice

仓库：<https://github.com/myshell-ai/OpenVoice>

### 当前状态

- Instant voice cloning；
- Zero-shot TTS；
- Python；
- License：MIT。

### 定位

可以作为 GPT-SoVITS 的重要替代基准：

- 接入简单；
- permissive license；
- zero-shot voice cloning。

项目活跃度不如当前 GPT-SoVITS，但仍是值得保留的备选。

---

## 10.3 Fish Speech

仓库：<https://github.com/fishaudio/fish-speech>

### 当前 License

截至 2026-09-11，官方仓库根目录 `LICENSE` 是：

> **FISH AUDIO RESEARCH LICENSE AGREEMENT**  
> Last Updated: March 7, 2026

许可明确写明：

> **Any Commercial use ... requires a separate written license agreement from Fish Audio.**

因此不能沿用旧文章或旧 fork 中可能出现的 Apache / CC BY-NC-SA 信息来判断当前商业权限。

### 判断

技术上值得 benchmark；

商业产品若要直接使用当前 Fish Audio Materials：

> **必须单独取得商业许可。**

所以不作为当前“开箱即用的 permissive commercial OSS”首选。

---

## 10.4 Kokoro

仓库：<https://github.com/hexgrad/kokoro>

### 当前状态

- lightweight TTS；
- License：Apache-2.0。

### 用途

非常适合：

- 开发期默认语音；
- 不需要真人 Voice Clone 的角色；
- 本地低成本 TTS。

但如果目标是：

> 高还原主播本人声音

它不是第一 Voice Clone 方案。

---

# 11. Windows 客户端 / Launcher / Overlay

推荐直接研究两个项目：

## 11.1 AiGameCompanion

提供：

```text
Tauri 2
+ Svelte 5
+ Rust
+ Steam Discovery
+ One-click Launch
+ Tray
+ Launch-on-startup
+ Process Watcher
+ Global Hotkey
+ Transparent Overlay
+ Windows.Graphics.Capture
```

而且明确采用：

> **external overlay, no injection**

这是最值得复用的 Windows 游戏 Companion 壳设计。

---

## 11.2 Yumii

重点不是 Overlay，而是安装和运行时：

```text
用户安装
   ↓
自动准备 private Python
   ↓
Backend
   ↓
Desktop App
   ↓
Start Menu
   ↓
第一次启动下载模型/语音组件
```

它证明：

> Python AI backend 并不等于普通用户必须面对 Python、pip、终端和本地端口。

所有复杂度都可以封装进 Installer / Launcher。

---

# 12. 建议的技术拼装图

如果仅根据当前开源生态拼一个 PoC，合理结构是：

```text
┌────────────────────────────────────┐
│ Windows Companion Client           │
│                                    │
│ AiGameCompanion / Yumii patterns   │
│ Tauri + Tray + Hotkey + Launcher   │
└────────────────┬───────────────────┘
                 │
                 ▼
┌────────────────────────────────────┐
│ Dota Adapter                       │
│                                    │
│ BrightGir/dota-ai-coach patterns   │
│ + current Dota GSI schemas         │
└────────────────┬───────────────────┘
                 │
                 ▼
        Typed GameState
                 │
                 ▼
┌────────────────────────────────────┐
│ Custom Event Normalizer            │
│                                    │
│ Previous State + Current State     │
│ → semantic Dota events             │
└────────────────┬───────────────────┘
                 │
                 ▼
┌────────────────────────────────────┐
│ Custom Speech Policy               │
│                                    │
│ Importance / Cooldown / Priority   │
│ User Speaking / AI Speaking        │
│ Repetition / Silence Budget        │
│ Personal talk-density preference   │
└────────────────┬───────────────────┘
                 │ SHOULD_SPEAK
                 ▼
┌────────────────────────────────────┐
│ Companion Brain                    │
│                                    │
│ Persona                            │
│ Current Game Event                 │
│ Recent Conversation                │
│ Relationship State                 │
│ Memory Retrieval                   │
│ Game / Streamer RAG                │
└───────────────┬────────────────────┘
                │
        ┌───────┴────────┐
        ▼                ▼
     Memory             RAG
 Mem0 / SQLite    Game + Persona KB
        │                │
        └───────┬────────┘
                ▼
               LLM
                │
                ▼
     GPT-SoVITS / OpenVoice
                │
                ▼
             Audio
```

实时用户语音侧：

```text
Push-to-talk / Mic
        ↓
Silero VAD（可选）
        ↓
STT
        ↓
Conversation Turn
        ↓
Companion Brain
        ↓
TTS
```

---

# 13. 开源许可证矩阵

> 本表只记录当前仓库代码/材料许可证状态，不构成法律意见。

| 项目 | 当前核验许可证 | 闭源商业直接复用风险 | 建议 |
|---|---|---:|---|
| BrightGir/dota-ai-coach | MIT | 低 | 可研究并按 MIT 条款复用 |
| chasmlol/chasm | **未声明 License** | **高** | 只借架构；复制代码前先获授权 |
| Open-LLM-VTuber | MIT；Live2D 示例资产独立许可 | 中 | 代码可用；资产单独核验 |
| AiGameCompanion | MIT | 低 | Windows 壳优先参考 |
| Yumii | MIT | 低 | 安装/Voice/Memory 可参考 |
| MrBean355/dota2-gsi | Apache-2.0 | 低 | Schema / JVM 实现可用 |
| antonpup/Dota2GSI | 仓库许可归属存在歧义 | 中高 | 先核实覆盖范围再复制代码 |
| Pipecat | BSD-2-Clause | 低 | Realtime Voice 强候选 |
| Mem0 | Apache-2.0 | 低 | Memory 强候选 |
| Letta | Apache-2.0 | 低 | 第二阶段 Stateful Agent 候选 |
| Noema | AGPL-3.0 | 高 | 架构参考为主 |
| Shikigami-Protocol | AGPL-3.0 | 高 | 主动人格架构参考 |
| Personality Machine | MIT | 低，但项目很早期 | 数据模型参考 |
| GPT-SoVITS | MIT | 代码低；真人声线权利另算 | 中文 Voice Clone 第一批测试 |
| OpenVoice | MIT | 代码低；真人声线权利另算 | Voice Clone 备选 |
| Fish Speech | Fish Audio Research License | **高** | 商业使用需单独书面许可 |
| Kokoro | Apache-2.0 | 低 | 默认/开发期 TTS |
| Silero VAD | MIT | 低 | VAD 推荐候选 |

---

# 14. 代码许可证之外必须单独处理的权利

## 14.1 真人声音

即使使用 MIT 的：

- GPT-SoVITS；
- OpenVoice；

也不意味着可以未经许可克隆任意主播声音。

代码权利和主播授权完全不同。

至少应单独约定：

- 声音样本是否允许用于训练；
- 训练后的模型归属；
- 是否允许商业生成；
- 使用地域；
- 使用期限；
- 生成内容类别；
- 是否允许导出模型；
- 是否允许再训练；
- 是否允许第三方部署；
- 合作终止后模型如何处置。

## 14.2 人格/名称/肖像

同理：

> 开源 Persona Framework ≠ 自动获得真实主播的人格、姓名、肖像、品牌商业授权。

这些必须由产品层单独解决。

---

# 15. 当前不建议的技术路线

## 15.1 DLL Injection / 读取游戏内存

没有必要。

Dota 已有 GSI。

同时 `AiGameCompanion` 的实际演化也说明：游戏 Companion 采用独立窗口通常比注入 renderer 更稳健。

推荐边界：

```text
Valve GSI
+ 外部 Companion App
+ External Overlay
```

不应为了多拿一点信息就进入：

```text
DLL Injection
Memory Scanning
Hidden Client State
```

## 15.2 每次 GSI Update 都调用 LLM

不推荐。

应先：

```text
Raw State
→ Event
→ Speech Policy
→ 只有需要说话才调 LLM
```

## 15.3 把所有历史都塞入 Prompt

不推荐。

建议区分：

```text
Structured User Profile
Structured Game History
Recent Conversation
Long-term Episodic Memory
Retrieved Relevant Memories
Streamer/Game Knowledge RAG
```

分别管理。

---

# 16. 推荐优先阅读顺序

如果只安排工程团队读代码，建议顺序：

### 第一组：先理解完整闭环

1. [`BrightGir/dota-ai-coach`](https://github.com/BrightGir/dota-ai-coach)  
   看 Dota GSI 如何实际接进 AI 应用。

2. [`chasmlol/chasm`](https://github.com/chasmlol/chasm)  
   看 Game Event、Memory、Proactive Reaction、Voice、Agent 如何组合。注意无许可证，只读架构。

3. [`Open-LLM-VTuber/Open-LLM-VTuber`](https://github.com/Open-LLM-VTuber/Open-LLM-VTuber)  
   看实时 AI Companion 的 Voice/Persona/Interruption。

### 第二组：Windows 产品壳

4. [`Wintersta7e/AiGameCompanion`](https://github.com/Wintersta7e/AiGameCompanion)

5. [`CodeNeuron58/Yumii`](https://github.com/CodeNeuron58/Yumii)

### 第三组：核心基础设施

6. [`MrBean355/dota2-gsi`](https://github.com/MrBean355/dota2-gsi)

7. [`pipecat-ai/pipecat`](https://github.com/pipecat-ai/pipecat)

8. [`mem0ai/mem0`](https://github.com/mem0ai/mem0)

9. [`RVC-Boss/GPT-SoVITS`](https://github.com/RVC-Boss/GPT-SoVITS)

### 第四组：行为与人格架构

10. [`HappyFox001/Noema`](https://github.com/HappyFox001/Noema)

11. [`Shikigami-Lab/Shikigami-Protocol`](https://github.com/Shikigami-Lab/Shikigami-Protocol)

12. [`emotion-machine-org/personality-machine`](https://github.com/emotion-machine-org/personality-machine)

---

# 17. 从开源生态能得到的最终技术判断

现有开源生态已经覆盖：

- Dota GSI 接入；
- GameState Parser；
- State Diff；
- Windows Overlay；
- Steam 游戏发现；
- Launcher；
- Global Hotkey；
- Screenshot Capture；
- 实时 Voice Pipeline；
- VAD；
- STT；
- Streaming TTS；
- Voice Clone；
- Persona；
- RAG；
- Vector Memory；
- Persistent Agent；
- Windows Installer；
- Local AI Runtime Lifecycle。

因此技术上真正没有“现成答案”、并且需要形成自己的系统设计的是：

```text
1. Dota Semantic Event Model
2. Speech Policy
3. Companion Behavior / Relationship Model
```

尤其是第二项：

> **什么时候主动说话，比“LLM 能不能生成一句话”更关键，也更难。**

当前最合理的工程原则是：

```text
GSI / Game State
       ↓
Deterministic Event Engine
       ↓
Deterministic Speech Policy
       ↓
Only if SPEAK
       ↓
LLM + Persona + Memory + RAG
       ↓
Voice Clone TTS
```

这是目前开源调研后最值得保留的总体架构。

---

# 18. 核验过但不纳入推荐清单的项目

早期探索中曾记录一个名为 `Atrium`、带 `TimingJudge / SilenceBudget` 概念的仓库引用。

本次正式落盘前重新核验时：

- 原先记录的 GitHub 路径返回 404；
- 当前 GitHub 搜索也无法可靠找到对应仓库。

因此本报告**不把它作为有效技术参考来源**，避免把已失效或错误引用带入正式项目文档。

---

# 19. 主要仓库清单

### Dota / Game Integration

- <https://github.com/BrightGir/dota-ai-coach>
- <https://github.com/antonpup/Dota2GSI>
- <https://github.com/MrBean355/dota2-gsi>

### Game-aware Agent / Companion

- <https://github.com/chasmlol/chasm>
- <https://github.com/Open-LLM-VTuber/Open-LLM-VTuber>
- <https://github.com/Wintersta7e/AiGameCompanion>
- <https://github.com/CodeNeuron58/Yumii>
- <https://github.com/HappyFox001/Noema>
- <https://github.com/Shikigami-Lab/Shikigami-Protocol>
- <https://github.com/emotion-machine-org/personality-machine>

### Realtime Voice / Memory

- <https://github.com/pipecat-ai/pipecat>
- <https://github.com/snakers4/silero-vad>
- <https://github.com/mem0ai/mem0>
- <https://github.com/letta-ai/letta>

### TTS / Voice Clone

- <https://github.com/RVC-Boss/GPT-SoVITS>
- <https://github.com/myshell-ai/OpenVoice>
- <https://github.com/fishaudio/fish-speech>
- <https://github.com/hexgrad/kokoro>

---

# 20. 一句话结论

> **目前没有一个可以直接 fork 就变成完整 Dota AI 陪玩的开源项目，但核心基础设施已经高度齐全：`dota-ai-coach` 提供 Dota 实时数据/AI骨架，`chasm` 提供游戏事件驱动人格 Agent 的关键架构，Open-LLM-VTuber/Yumii 提供实时 Companion，AiGameCompanion 提供 Windows 游戏壳，Pipecat/Mem0/GPT-SoVITS 等提供成熟基础组件。真正应该自主开发并形成壁垒的，不是 STT/TTS/LLM 接口，而是 Dota Event Model、Speech Policy 和长期 Companion Behavior。**
