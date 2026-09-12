# AI 游戏陪玩：开源技术参考与可复用模块调研

> 核验日期：2026-09-12  
> 目标：为 Dota 2 / 后续其他游戏的 AI 陪玩产品寻找可直接复用、可做 PoC、或值得借鉴架构的开源项目。  
> 搜索范围已扩展到游戏 AI、实时语音、主动 Agent、HCI、turn-taking、backchannel、interruptibility、AI Clone、情感计算、长期记忆和桌面 Companion。  
> 注意：代码许可证、模型许可证、主播声音/肖像/人格授权是三件不同的事，必须分别处理。

---

# 1. 当前结论

没有发现一个成熟开源项目可以直接 Fork 后变成完整的 Dota AI 陪玩，但核心基础设施已经高度齐全。

当前最重要的判断已经从早期的：

```text
Game Event Model + Speech Policy + Companion Behavior
```

进一步升级为：

```text
1. Game Event Model
2. Attention Engine
3. Interruptibility / Turn-Taking Engine
4. Reaction Arbiter
5. Reflex / Backchannel Engine
6. Persona State
7. Relationship State
8. Shared Memory
9. Cognitive Response Engine
```

真正最容易形成产品差异、不能简单外包给通用 LLM 的部分是：

> **Attention + Interruptibility + Reaction Arbitration + Behavior Clone。**

也就是：

- 什么值得反应；
- 什么时候保持沉默；
- 什么时候只“嗯 / 啊 / 卧槽 / 笑一下”；
- 什么时候说完整一句；
- 什么事件先记住，等安全窗口再说；
- 某个主播在类似时刻通常如何反应；
- 与用户熟悉以后，反应方式如何变化。

因此，GSI、STT、TTS、LLM、Overlay、Vector DB 都越来越像 commodity；真正的产品壁垒可能是一个 **Companion Interaction Engine**。

---

# 2. 推荐的总体架构

```text
Dota GSI / Screen / Mic
        ↓
Game State Model
        ↓
Game Event Interpreter
        ↓
Reaction Candidates
        ↓
┌──────────────────────────────┐
│ Attention / Relevance Gate   │
│ 这件事值得反应吗？             │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│ Interruptibility / Turn-Taking│
│ 现在适合开口吗？               │
└──────────────┬───────────────┘
               ↓
        Reaction Arbiter
      ┌────────┼─────────┐
      ↓        ↓         ↓
   Silence  Backchannel  Full Turn
               ↓         ↓
        Reflex Voice   Persona Brain
                       + Memory
                       + Relationship
                       + Streamer RAG
                       + Game Context
                         ↓
                       LLM
                         ↓
                    Streaming TTS
      └───────────┬─────────────┘
                  ↓
                Audio
```

关键原则：

1. **事件发生 ≠ 现在就说话。**
2. **“是否值得说”与“现在是否适合插话”必须分离。**
3. **Backchannel 与完整发言应分成两条延迟不同的通路。**
4. **实时反射层与复杂推理层不应共用一个延迟预算。**
5. **人格不应只存在于 System Prompt，关系和情绪应有显式状态。**

---

# 3. 当前参考优先级

| 能力层 | 当前首要参考 | 当前判断 |
|---|---|---|
| Dota 数据 | Dota GSI + `dota-ai-coach` / Dota2GSI | 已有成熟参考，不需要从零造 |
| 游戏事件语义 | `chasm` 思想 + 自研 Event Model | 需要形成自己的陪玩事件模型 |
| 主动开口 / Silence | **Miru AttentionEngine** | 当前最完整、最贴近目标的工程参考 |
| 主动 Agent 研究 | THUNLP ProactiveAgent / ProAgentBench | 用于 timing 和用户反馈学习 |
| Backchannel | **MaAI / VAP** | 当前最干净、低成本的 PoC 参考 |
| 中文 Turn Completion | **TEN Turn Detection** | 新增重点候选 |
| 实时语音 Orchestration | Pipecat / LiveKit / TEN Framework | 按场景组合，不是互斥关系 |
| 本地全双工模型 | MiniCPM-o 4.5 / PersonaPlex / Moshi | 后期上限测试 |
| 最终交互架构上限 | **Gander / Omni-Interaction-Agent** | 2026-09 新增首要参考 |
| 完整 AI Character / 游戏插件平台 | **AIRI** | 完整产品架构优先参考 |
| 情绪连续性 | Affect Kernel | 显式 deterministic affect state |
| 关系连续性 | eros-engine / social-persona-engine | 显式关系状态和行为决策 |
| 长期记忆 | Mem0 / Letta + LongMemEval / LoCoMo | 实现与评测需分开 |
| Windows 游戏客户端 | AiGameCompanion / Tauri 外部 Overlay | 外部非注入路线优先 |
| Voice Clone | GPT-SoVITS / OpenVoice | 中文第一批测试候选 |

---

# 4. Dota 数据与游戏事件层

## 4.1 BrightGir/dota-ai-coach

仓库：<https://github.com/BrightGir/dota-ai-coach>

当前最直接的 Dota AI 应用参考之一，已经实现：

- Dota 2 GSI；
- 本地 HTTP Listener；
- JSON → GameState；
- Thread-safe State Store；
- Hero / Items / Abilities；
- Gemini / OpenRouter；
- RAG；
- Dota 英雄、装备、技能知识库；
- 自动 Advice；
- Windows Overlay；
- Hotkey。

核心链路：

```text
Dota 2
  ↓ GSI HTTP POST
Parser / GameState Store
  ↓
Prompt + RAG
  ↓
LLM
  ↓
Overlay
```

价值：直接参考 GSI 接入、状态存储、RAG 和 Windows 应用组织方式。

限制：当前更像 Coach，自动建议主要按周期调用 LLM，不适合直接当自然陪玩行为。

License：MIT。

---

## 4.2 antonpup/Dota2GSI

仓库：<https://github.com/antonpup/Dota2GSI>

最值得参考的是：

```text
Previous GameState
        +
Current GameState
        ↓
High-level Event
```

例如 InventoryItemAdded、TowerDestroyed 等。

这与 AI 陪玩真正需要的 Event Normalizer 很接近。

许可证注意：仓库当前许可证覆盖范围存在歧义，直接复制代码前需要再次确认；更适合作为事件 Diff 和 Schema 设计参考。

---

## 4.3 MrBean355/dota2-gsi

仓库：<https://github.com/MrBean355/dota2-gsi>

Kotlin/JVM，Apache-2.0，当前最大的价值是作为较新的 typed GSI schema 参考，用于核对：

- PlayingGameState；
- SpectatingGameState；
- Player；
- Hero；
- Items；
- Abilities；
- Map；
- Events。

生成文档：<https://mrbean355.github.io/dota2-gsi/>

---

# 5. Game Event Model：必须形成自己的语义层

原始 GSI 不应直接进入 LLM。

推荐：

```text
Raw GSI
  ↓
Typed State
  ↓
Previous vs Current State
  ↓
Semantic Event Normalizer
```

最终输出类似：

```text
PLAYER_DIED
PLAYER_GOT_KILL
MULTI_KILL_WINDOW
ITEM_PURCHASED
ITEM_COMPLETED
LEVEL_UP
ULTIMATE_READY
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

这层建议自研，因为“什么事件对陪玩有意义”本身就是产品定义。

Coach 看到死亡会分析原因；Companion 看到同一个死亡，可能吐槽、安慰、回调之前的梗，或者完全沉默。

---

# 6. 主动开口：Miru AttentionEngine

仓库：<https://github.com/kiyotakali/Miru>

这是目前找到最接近“持续观察 → 理解 → 决定要不要主动靠近”的完整工程实现。

最重要的不是主动消息本身，而是把以下内容独立建模：

- `AttentionEngine` 独立负责 `say or stay quiet`；
- `speak_intent` 与最终消息措辞分离；
- `inner`：AI 最近没有说出口的想法；
- `user_affect`：AI 对用户当前状态的持续判断；
- `self_emotion`：AI 自己的连续情绪；
- current focus；
- proactive cadence；
- 未回应主动消息感知；
- intent queue；
- dedup；
- salience；
- cooldown / silence；
- 普通共同生活场景，而不是只有告警和提醒。

其思想更接近：

```text
持续接收信号
  ↓
合并 / 降噪
  ↓
更新内部连续状态
  ↓
Attention LLM 判断
  ↓
null 或 speak_intent
  ↓
另一个 Agent 负责最终措辞
```

这比“每隔 X 分钟问 LLM 要不要说一句”成熟得多。

当前结论：

> **如果只验证“AI主动开口是否能形成陪伴感”，Miru 是第一项本地 Demo 的优先参考。**

---

# 7. 主动 Agent 研究：ProactiveAgent / ProAgentBench

## 7.1 THUNLP ProactiveAgent

仓库：<https://github.com/thunlp/ProactiveAgent>

值得借鉴的点：用户对主动介入可以 Accept / Reject / Ignore，并将反馈用于学习用户对不同介入方式的接受程度。

对游戏陪玩可以映射为：

```text
某类事件主动说话
→ 用户回应 / 笑 / 不回应 / 打断 / 说“闭嘴”
→ 更新个人化 intervention preference
```

这比固定“话多 / 正常 / 话少”三个档位更有潜力。

## 7.2 ProAgentBench

2026 年主动 Agent 评测工作将问题拆为：

- Timing Prediction：现在该不该介入；
- Assist Content Generation：介入后说什么。

其结论之一是长期用户历史会提高 timing 判断质量。这进一步支持：GSI 当前事件不足以单独决定是否开口。

---

# 8. Backchannel：MaAI / VAP

仓库：<https://github.com/MaAI-Kyoto/MaAI>

京都大学的 MaAI 专门处理：

- turn-taking；
- backchannel timing；
- backchannel type；
- nodding；
- 英 / 中 / 日；
- CPU 实时运行。

这里最关键的是一个此前容易遗漏的产品能力：

> **AI陪玩不应该只有“完整说一句话”这一种输出。**

真人在旁边大量存在：

```text
“嗯”
“啊？”
“卧槽”
“可以”
（笑）
“啧”
“哎哟”
```

这类输出不是完整 Conversation Turn，而是“我与你共同经历这个时刻”的轻反应。

2026 年 AI Clone 相关实验表明，加入实时 backchannel / nodding 后，对 attentiveness、真人感和 co-presence 都有明显提升。

因此 MaAI 是第二项本地 Demo 的优先候选。

---

# 9. 中文 Turn Detection：TEN Turn Detection

仓库：<https://github.com/TEN-framework/ten-turn-detection>  
框架：<https://github.com/TEN-framework/ten-framework>

TEN 将用户当前语句分成：

- `finished`：表达完成，可以回应；
- `unfinished`：只是停顿，还没有说完；
- `wait`：明确要求 AI 暂停 / 先别说。

项目明确支持中文和英文。

项目方在自建中文测试集报告：

- finished accuracy：98.90%；
- unfinished accuracy：92.74%；
- wait accuracy：92%。

这些是项目方自测，不应视为独立第三方 benchmark，但足以成为中文产品实测候选。

需要区分：

```text
VAD
= 用户有没有发声

TEN / Smart Turn
= 用户这句话说完没有 / 是否让我等

MaAI / VAP
= 下一刻谁应该讲话 / 是否适合 backchannel

LiveKit / Pipecat
= 如何把这些信号接入真正的实时音频管线
```

不是四选一，而可能组合使用。

---

# 10. 实时语音：Pipecat / LiveKit

## 10.1 Pipecat

仓库：<https://github.com/pipecat-ai/pipecat>

BSD-2-Clause，大型活跃实时 Voice Agent 框架。

适合承担：

```text
Mic
 ↓
VAD / Turn Detection
 ↓
STT
 ↓
Agent / LLM
 ↓
TTS
 ↓
Audio
```

以及 streaming、provider abstraction、barge-in 等。

Pipecat 的 Smart Turn 系列比“检测到静音就当用户说完”更先进：它尝试判断用户是真的结束表达，还是仅仅句中停顿。

## 10.2 LiveKit Agents

LiveKit 更值得借鉴的是把实时语音拆成不同问题：

```text
VAD
用户有没有在说话？

STT
用户说了什么？

Turn Detection
用户这一轮真的说完了吗？

Interruption Detection
用户现在发声，是要我闭嘴，还是只是“嗯 / 对 / okay”？
```

这对自然陪玩尤其重要，因为用户的短 backchannel 不应该导致 AI 每次立刻停止讲话。

---

# 11. 未来上限架构：Gander / Omni-Interaction-Agent

仓库：<https://github.com/Omni-Interaction-Gander/Omni-Interaction-Agent>  
论文：<https://arxiv.org/abs/2609.08977>

Gander 于 2026-09-09 发布，是本轮最重要的新发现之一。

它不再是传统：

```text
STT → LLM → TTS
```

而是把以下能力放进持续交互体系：

- full-duplex voice；
- turn-taking；
- backchannel；
- overlap handling；
- interruption；
- proactive responses；
- continuous audio-visual perception；
- 用户语音、屏幕/视频与 Agent 执行共享时间线；
- 长任务期间仍可继续对话、被打断和重定向；
- Cerebellum（低延迟实时交互）+ Brain（复杂推理）的双层架构。

最重要的启示：

> **实时陪伴和复杂推理不能共用同一个延迟预算。**

映射到本项目：

```text
Reflex / Cerebellum
- 实时监听
- backchannel
- 打断
- overlap
- 极短游戏反应

Cognitive Brain
- 游戏局势解释
- 长期记忆
- 主播人格
- 梗库 / RAG
- 技术建议
- 长句生成
```

当前不建议把 Gander 当第一项本地 Demo：它太新、太重，而且会把多个产品假设混在一起。

定位：**未来最终交互架构上限参考第一优先级。**

---

# 12. 本地全双工模型：MiniCPM-o 4.5 / PersonaPlex / Moshi

## 12.1 MiniCPM-o 4.5

主仓库：<https://github.com/OpenBMB/MiniCPM-V>  
Demo：<https://github.com/OpenBMB/MiniCPM-o-Demo>

Duplex 模式可以持续接收音频（可加视频帧），模型自己在每一步决定 `listen` 或 `speak`，而不是完全依赖外部 VAD 决定轮次。

当前官方能力包括：

- Audio Full-Duplex；
- Omnimodal Full-Duplex；
- autonomous listen/speak；
- interruption；
- pause/resume；
- reference audio；
- 本地 Web Demo。

硬件明显比 MaAI / Miru 单模块实验更重，因此建议后测。

## 12.2 PersonaPlex / Moshi

PersonaPlex 与 Moshi 仍是 full-duplex spoken dialogue 的重要参考，代表未来从模块化 STT→LLM→TTS 走向持续双向音频交互的方向。

但本项目第一阶段仍推荐模块化，因为游戏事件、关系状态、主播行为需要精确控制与可测试性。

---

# 13. 完整 AI Character / 游戏桥接平台：AIRI

仓库：<https://github.com/moeru-ai/airi>

AIRI 是本轮扩大搜索范围后最值得完整阅读的项目之一。

其方向不是简单聊天机器人，而是一个可以长期存在于多个数字环境中的 AI Character，覆盖：

- 实时语音；
- Memory；
- Live2D / VRM；
- Desktop / Web；
- Discord / Telegram；
- Minecraft；
- Factorio；
- 插件化环境接入。

AIRI 的 Plugin Platform 方向尤其重要：底层不同游戏可以使用各自 Bridge，但上层统一接 Companion Core。

这与本项目的正确方向一致：

```text
Dota Bridge → GSI
LOL Bridge → 另一套数据接口
Minecraft Bridge → Mod/API
...

        ↓
统一 Game Event Contract
        ↓
Companion Core
```

也就是说，不需要强行让不同游戏共享底层数据实现；应该统一的是上层事件契约。

License：MIT。

---

# 14. 游戏事件驱动 Agent：chasmlol/chasm

仓库：<https://github.com/chasmlol/chasm>

这是目前游戏行为架构上仍然非常重要的参考。

其核心链路：

```text
GAME
 ↓
Bridge
 ↓
Game State / Events
 ↓
Persona
 ↓
Memory / Retrieval
 ↓
LLM
 ↓
Streaming TTS / Voice
```

最值得借鉴的是：

```text
Game Event
 ↓
Mechanical Trigger
 ↓
Witness / Context Gate
 ↓
Cooldown
 ↓
Agent Turn
 ↓
LLM 决定具体表达
```

而不是每帧问 LLM“要不要说话”。

这在实时游戏环境中非常合理：低成本、低延迟、可测试、发言密度可控。

许可证风险：截至当前仓库未明确声明标准许可证，因此建议只借鉴公开架构和思路，未经授权不要直接复制代码进入闭源商业产品。

---

# 15. 情绪状态：Affect Kernel

仓库：<https://github.com/kevindechang/affect-kernel>

Affect Kernel 的价值是把情绪做成显式、可测试、可回放状态，而不是每次让 LLM 临场“演情绪”。

可参考状态包括：

- PAD mood；
- valence；
- arousal；
- dominance；
- OCC-inspired emotion；
- OCEAN personality；
- relationship stage；
- trust；
- attachment；
- expectation；
- carried thought。

输入 appraised event，输出更新后的 affect state 和 response controls。

License：Apache-2.0。

适合参考“主播情绪连续性”与“连续三连败后状态应该变化”这类问题。

---

# 16. 关系状态：eros-engine / social-persona-engine

关系不能只存在于长期记忆文本里。

可参考把关系显式拆成多个维度，例如：

```text
warmth
trust
intimacy
intrigue
patience
tension
```

同时区分：

```text
User Profile Memory
= 用户稳定事实

Relationship / Shared Memory
= 我们共同经历过什么、内部梗、没说完的话题
```

对游戏陪玩来说：

```text
User Profile
- 主玩影魔
- 偏好中路
- 常在晚上游戏

Shared Memory
- 昨天三连败后说“再选影魔我是狗”
- 第一次五杀
- 上次AI吐槽他没按BKB
```

这两类数据必须分开。

AGPL 类项目只作为架构参考，正式闭源商业产品需重新审查许可。

---

# 17. Persona / Companion Behavior

Persona 不应只是：

```text
system_prompt = "你现在是某主播"
```

更完整的输入应包含：

```text
Character Core
Speaking Style
Catchphrases
Allowed / Forbidden Topics
Game Opinions
Current Emotion State
Current Relationship State
User Profile
Recent Shared Events
Relevant Long-term Memories
Current Game Event
Current Game Context
Streamer / Game RAG
```

其中：

```text
Attention Engine
= 要不要开口

Reaction Arbiter
= 沉默 / 短反应 / 完整发言

Companion Behavior
= 以怎样的关系、情绪和角色状态表达

LLM
= 最终自然语言实现
```

---

# 18. 长期记忆：Mem0 / Letta / SQLite

## 18.1 Mem0

仓库：<https://github.com/mem0ai/mem0>

Apache-2.0，适合快速 PoC 的 Persistent Memory Layer。

可以保存：

- 游戏偏好；
- 常用英雄；
- 对话偏好；
- 长期梗；
- 重要共同事件；
- 关系事实。

正式产品仍建议把结构化游戏历史和 LLM 自由记忆分开。

## 18.2 Letta

仓库：<https://github.com/letta-ai/letta>

Apache-2.0，更接近完整 Stateful Agent Platform，能力强但对 MVP 可能偏重。

## 18.3 简化 SQLite

早期 PoC 完全可以先采用简单可调试结构：

```text
sessions
facts
transcripts
summaries
checkpoints
shared_events
```

复杂 Memory Framework 不应成为 MVP 的前置条件。

## 18.4 Memory Benchmark

后续不应只凭 Star 选择记忆方案，应该用 LongMemEval / LoCoMo 一类基准思想建立自己的 `Game Companion Memory Eval`。

重点测试：

- 事实是否记得；
- 多局之间能否推理；
- 新事实能否覆盖旧事实；
- 时间关系；
- 不知道时能否 abstain；
- 长期隐含偏好能否影响当前行为。

---

# 19. 语音识别与 VAD

Silero VAD：<https://github.com/snakers4/silero-vad>

MIT，成熟轻量。

用途：检测用户是否开始/停止发声。

STT 本身不应作为核心自研方向；Whisper、Groq Whisper、FunASR 等均已有成熟方案，Open-LLM-VTuber、Yumii、Pipecat 也提供成熟集成模式。

如果早期采用 Push-to-talk，甚至可以先降低 VAD/turn detection 的复杂度，后续再升级 hands-free/full-duplex。

---

# 20. TTS / Voice Clone

## 20.1 GPT-SoVITS

仓库：<https://github.com/RVC-Boss/GPT-SoVITS>

MIT，中文 Voice Clone 第一批 benchmark 候选。

优势：

- 中文社区成熟；
- 少样本 Voice Clone；
- 本地部署；
- 可作为主播授权声线的主要技术候选。

## 20.2 OpenVoice

仓库：<https://github.com/myshell-ai/OpenVoice>

MIT，Instant Voice Cloning / Zero-shot TTS，适合作为 GPT-SoVITS 的备选基准。

## 20.3 Fish Speech

仓库：<https://github.com/fishaudio/fish-speech>

当前官方材料使用 Fish Audio Research License，商业使用需要单独书面许可，因此不作为默认 permissive 商业 OSS 首选。

## 20.4 Kokoro

仓库：<https://github.com/hexgrad/kokoro>

Apache-2.0，轻量，适合开发期默认语音，但不是高还原真人 Voice Clone 第一选择。

### 真人声音法律边界

即使 TTS 代码是 MIT，也不等于可以未经授权克隆主播声音。

需要单独约定：

- 训练样本授权；
- 商业生成权；
- 模型归属；
- 使用期限；
- 内容范围；
- 是否允许导出 / 再训练；
- 合作终止后的模型处理。

---

# 21. Windows 客户端 / Overlay / Launcher

## 21.1 Wintersta7e/AiGameCompanion

仓库：<https://github.com/Wintersta7e/AiGameCompanion>

Tauri 2 + Svelte + Rust，MIT。

最值得借鉴：

- 外部透明 Overlay；
- Always-on-top；
- Click-through；
- Global Hotkey；
- Windows.Graphics.Capture；
- Steam Library 自动发现；
- One-click Launch；
- Tray；
- Launch-on-startup；
- 进程监测；
- Windows Credential Manager。

特别重要：项目从 DLL Injection / Renderer Hook 转向外部透明窗口，原因就是稳定性和 anti-cheat 风险。

推荐边界：

```text
Valve GSI
+ 外部 Companion App
+ External Overlay
```

不建议：

```text
DLL Injection
Memory Scanning
Hidden Client State
```

## 21.2 Yumii

仓库：<https://github.com/CodeNeuron58/Yumii>

MIT。

价值在于：如何把 Python AI Runtime + 本地语音 + Tauri UI 包装成普通 Windows 用户可安装的软件。

可参考：

- Windows-first Desktop；
- Silero VAD；
- Whisper；
- barge-in；
- Kokoro / ElevenLabs；
- Ollama / OpenAI / Anthropic；
- persistent personality；
- SQLite memory；
- Installer / Start Menu / Backend lifecycle。

---

# 22. Open-LLM-VTuber

仓库：<https://github.com/Open-LLM-VTuber/Open-LLM-VTuber>

主代码 MIT，Live2D 示例资产有独立许可证。

值得借鉴：

- hands-free voice；
- interruption / barge-in；
- STT / TTS provider abstraction；
- Persona；
- Live2D；
- 本地 Companion loop。

它可以理解为“陪伴 / Voice / 角色”这一半，而 Dota GSI Adapter 是“游戏状态”这一半。

---

# 23. 当前不建议的技术路线

## 23.1 每次 GSI Update 都调 LLM

不推荐。

正确链路更接近：

```text
Raw State
→ Event
→ Attention / Policy
→ Interruptibility
→ Reaction Arbiter
→ 只有需要完整发言时才调 LLM
```

## 23.2 所有反应都走 LLM + TTS

不推荐。

需要两条通路：

```text
Reflex Brain
→ “卧槽 / 嗯 / 啊？ / 笑”
→ 极低延迟

Cognitive Brain
→ Game + Memory + Persona + RAG
→ LLM
→ 完整句子
```

## 23.3 把所有历史塞入 Prompt

不推荐。

应区分：

```text
Structured User Profile
Structured Game History
Recent Conversation
Shared Relationship Memory
Long-term Episodic Memory
Retrieved Memories
Streamer / Game Knowledge RAG
```

## 23.4 一开始就上完整 Full-duplex 大模型

不推荐。

第一阶段应该逐个验证产品假设，否则体验不好时无法判断是主动时机、backchannel、语音延迟、人格还是模型的问题。

---

# 24. 建议的本地验证顺序

每次只验证一个产品假设。

1. **主动开口 / Silence Demo（Miru）**  
   验证：AI 能否在用户没提问时，基于事件和上下文主动说或主动保持沉默。

2. **Backchannel Demo（MaAI）**  
   验证：极短“嗯 / 哦 / 啊 / 笑 / 卧槽”等是否显著增强共同在场感。

3. **Turn / Interruption Demo（TEN + Pipecat / LiveKit）**  
   验证：中文说话没结束时 AI 不抢话；用户真正打断时 AI 能停。

4. **关系 / 记忆 Demo**  
   验证：同一事件在不同用户历史和关系状态下产生不同反应。

5. **Dota GSI 接入 Demo**  
   把前四项接到真实 Dota Game Event。

6. **全双工上限 Demo（MiniCPM-o 4.5 / Gander 路线）**  
   最后再判断是否值得用端到端 full-duplex 体系替换部分模块化 pipeline。

当前第一项实验仍应是：

> **Miru / AttentionEngine 风格的“主动开口 + Silence”能力。**

---

# 25. 开源许可证与直接复用风险

> 仅为工程筛选，不构成法律意见。

| 项目 | 当前许可证 / 状态 | 闭源商业直接复用判断 |
|---|---|---|
| BrightGir/dota-ai-coach | MIT | 低风险，按 MIT 条款 |
| chasmlol/chasm | 当前未明确 License | 高风险，只借架构 |
| AIRI | MIT | 低风险，需另查模型/资产 |
| Open-LLM-VTuber | MIT；Live2D 资产另算 | 代码低，资产单独核验 |
| AiGameCompanion | MIT | 低 |
| Yumii | MIT | 低 |
| MrBean355/dota2-gsi | Apache-2.0 | 低 |
| antonpup/Dota2GSI | 许可覆盖存在歧义 | 先核实再复制 |
| Pipecat | BSD-2-Clause | 低 |
| Silero VAD | MIT | 低 |
| Mem0 | Apache-2.0 | 低 |
| Letta | Apache-2.0 | 低 |
| Affect Kernel | Apache-2.0 | 低 |
| GPT-SoVITS | MIT | 代码低；真人声线另算 |
| OpenVoice | MIT | 代码低；真人声线另算 |
| Fish Speech | Research License | 商业需单独许可 |
| Kokoro | Apache-2.0 | 低 |
| 部分关系/Persona 项目 | AGPL-3.0 | 闭源产品需谨慎 |

---

# 26. 优先阅读顺序

如果工程团队开始阅读源码，建议按下面顺序：

### A. 核心产品机制

1. `kiyotakali/Miru` — Attention / speak_intent / silence
2. `MaAI-Kyoto/MaAI` — turn-taking / backchannel
3. `Omni-Interaction-Gander/Omni-Interaction-Agent` — 最终交互上限
4. `moeru-ai/airi` — 完整 Character / Plugin / Game Bridge
5. `chasmlol/chasm` — Game Event → Agent Reaction（只借架构）

### B. Dota

6. `BrightGir/dota-ai-coach`
7. `MrBean355/dota2-gsi`
8. `antonpup/Dota2GSI`

### C. 实时语音

9. `TEN-framework/ten-turn-detection`
10. `pipecat-ai/pipecat`
11. LiveKit Agents
12. `OpenBMB/MiniCPM-V` / MiniCPM-o Demo

### D. 关系 / 情绪 / 记忆

13. Affect Kernel
14. Mem0
15. Letta
16. eros-engine / social-persona-engine 类关系状态项目

### E. 产品壳与声音

17. `Wintersta7e/AiGameCompanion`
18. `CodeNeuron58/Yumii`
19. `Open-LLM-VTuber/Open-LLM-VTuber`
20. `RVC-Boss/GPT-SoVITS`
21. `myshell-ai/OpenVoice`

---

# 27. 最终技术判断

当前开源生态已经覆盖：

- Dota GSI；
- Typed GameState；
- State Diff；
- 游戏事件触发；
- Windows Overlay；
- Steam Launcher；
- Global Hotkey；
- Screen Capture；
- VAD / STT；
- Streaming TTS；
- Voice Clone；
- Realtime Voice Pipeline；
- Turn Detection；
- Backchannel；
- Persistent Memory；
- Persona；
- 显式 Affect / Relationship State；
- Desktop Companion；
- Full-duplex Speech；
- 多环境 Character / Plugin 架构。

因此真正需要形成自己的系统设计的不是“怎么把语音和 LLM 接起来”，而是：

```text
Game Event Model
+ Attention Engine
+ Interruptibility Model
+ Reaction Arbiter
+ Reflex / Backchannel Engine
+ Behavior Clone
+ Relationship / Shared Memory
```

一句话总结：

> **技术上最有价值的方向已经从“做一个能看懂 Dota 的语音 AI”升级为“做一个持续感知游戏与用户、知道什么时候沉默、什么时候只反应一下、什么时候真正开口，并随着共同经历形成熟悉感的 Companion Interaction Engine”。**
