# AI 游戏陪玩：第二轮跨领域开源参考更新

> 核验日期：2026-09-12  
> 本文是 `docs/ai-companion-open-source-references.md` 的增量更新。  
> 本轮目标不是继续堆同类项目，而是专门寻找能否替代上一轮核心候选的更优实现，搜索范围扩展到：实时全双工语音、turn-taking、backchannel、interruptibility、主动 Agent、桌面陪伴、社交机器人、AI Clone、情感/关系状态与持续多模态感知。

---

## 1. 第二轮结论

上一轮大方向成立，但有两处需要升级：

1. **未来最终交互形态参考：由 PersonaPlex 单独领先，升级为以 Gander / Omni-Interaction-Agent 为首要上限参考，PersonaPlex / Moshi 作为其全双工语音基础参考。**
2. **中文实时语音 turn detection：新增 TEN Turn Detection，值得与 Pipecat Smart Turn / LiveKit 直接实测比较。**

其余关键结论暂不推翻：

- **主动陪伴 / “该不该开口”机制：Miru AttentionEngine 仍是目前最完整、最贴近目标的工程参考。**
- **backchannel / 轻量实时“嗯、哦、啊、笑”等倾听反应：MaAI 仍是目前最干净、最适合单独做 PoC 的参考。**
- **完整 AI Character / 游戏桥接平台：AIRI 仍是成熟度、规模、插件化与游戏接入综合最强的参考之一。**
- **实时语音工程框架：Pipecat 与 LiveKit 仍然成熟；TEN 主要新增中文 turn detection 候选，不直接替代整个语音框架。**
- **显式情绪/关系状态：Affect Kernel、eros-engine（原 rp-engine 方向）仍值得参考；不建议只把“人格”写在 system prompt 里。**

因此当前最合理的判断不是“找到一个仓库直接复制”，而是继续采用分层架构：

```text
Dota GSI / Screen / Mic
        ↓
Game Event Model
        ↓
Attention / Relevance Gate
        ↓
Interruptibility / Turn-Taking
        ↓
Reaction Arbiter
   ┌────┼─────┐
Silence Backchannel Full Turn
        ↓          ↓
  Reflex Voice   Persona + Memory + LLM
        └────┬─────┘
             ↓
            TTS
```

---

## 2. 新增首要参考：Gander / Omni-Interaction-Agent

仓库：<https://github.com/Omni-Interaction-Gander/Omni-Interaction-Agent>  
论文：<https://arxiv.org/abs/2609.08977>

### 为什么它重要

Gander 于 2026-09-09 正式发布。它不是传统 `STT → LLM → TTS` 拼接，而是把以下能力放入同一个持续交互体系：

- full-duplex voice；
- turn-taking；
- backchannel；
- overlap handling；
- interruption；
- proactive responses；
- continuous audio-visual perception；
- 用户说话、屏幕/视频与 Agent 执行共享时间线；
- 长任务执行期间仍可继续与用户说话、被打断、被重定向；
- Cerebellum（低延迟实时交互）+ Brain（复杂推理/Agent）的双层架构。

其最值得借鉴的并不是某个模型，而是：

> **实时陪伴和复杂推理不应该共用同一个延迟预算。**

这与本项目正在形成的 `Reflex Brain + Cognitive Brain` 高度一致。

### 对本项目的架构启发

```text
Cerebellum / Reflex Layer
- 低延迟监听
- 是否开口
- backchannel
- 打断/重叠
- 短反应

Brain / Cognitive Layer
- 游戏局势解释
- 主播人格
- 长期记忆
- 梗库 / RAG
- 复杂建议
```

### 是否直接采用

当前**不建议作为第一个本地 Demo**：

- 发布时间极新；
- 工程和模型体量显著高于 MaAI / Miru 单模块验证；
- 依赖 MiniCPM-o 4.5 等较重模型体系；
- 第一阶段目标只是验证产品交互价值，不需要先承担整套全双工大模型部署成本。

因此定位为：

> **未来最终交互架构 / 技术上限参考第一优先级。**

---

## 3. 新增中文 Turn Detection 参考：TEN Turn Detection

仓库：<https://github.com/TEN-framework/ten-turn-detection>  
框架：<https://github.com/TEN-framework/ten-framework>

TEN 将用户语句分成三个状态：

- `finished`：表达完成，可以回应；
- `unfinished`：只是停顿，还没说完；
- `wait`：明确要求 AI 暂停/别说。

它明确支持中文和英文，并提供公开测试集。仓库公布的自测中，中文：

- finished accuracy：98.90%；
- unfinished accuracy：92.74%；
- wait accuracy：92%。

注意：这是项目方在其自建测试集上的结果，不能等同于独立第三方 benchmark；但对中文产品而言，值得真实 A/B 测试。

### 与 MaAI / Pipecat / LiveKit 的关系

它们解决的问题不同：

```text
VAD
= 用户有没有在发声

TEN / Smart Turn
= 用户这句话说完没有 / 是否明确让我等

MaAI / VAP
= 下一刻谁更应该讲话；是否适合 backchannel

LiveKit / Pipecat
= 如何把上述信号接入真实实时语音 pipeline、处理 barge-in 与播放控制
```

所以不是四选一，而更可能组合使用。

---

## 4. MiniCPM-o 4.5：比上一轮更值得关注的本地全双工底座

主仓库：<https://github.com/OpenBMB/MiniCPM-V>  
官方 Demo：<https://github.com/OpenBMB/MiniCPM-o-Demo>

MiniCPM-o 4.5 的 Duplex 模式每秒持续接收音频（可附带视频帧），模型自己在每一步输出 `listen` 或 `speak`，而不是完全依赖外部 VAD 决定轮次。

官方当前提供：

- Audio Full-Duplex；
- Omnimodal Full-Duplex；
- autonomous listen/speak；
- interruption；
- pause/resume；
- reference audio / voice 条件；
- 本地 Web Demo。

官方给出的低资源门槛约为：

- Mac M4 Max 24GB 级别，或
- NVIDIA GPU 约 12GB VRAM 的低资源路径；

官方完整 PyTorch Demo 的默认 Token2Wav worker 初始化后约 21.5GB VRAM，另有 GGUF / AWQ 低显存版本。

### 对本项目的意义

如果后面要验证“AI一边听玩家说话，一边看屏幕/游戏状态，并自己决定什么时候开口”的端到端体验，MiniCPM-o 4.5 是比单纯 STT+LLM+TTS 更值得实测的本地候选。

但仍不建议把它作为第一项 PoC，因为第一项 PoC 应该只验证一个产品假设。

---

## 5. Miru 仍然是主动开口机制首选参考

仓库：<https://github.com/kiyotakali/Miru>

本轮找到若干更简单的主动桌面陪伴实现，例如：

- <https://github.com/LuizEduPP/Companion>
- <https://github.com/inni918/warashi>
- <https://github.com/yuri-os/YuriOS>
- <https://github.com/yier-deer/social-persona-engine>

但它们目前都没有在“持续观察 → 内部状态 → speak_intent → 去重/节奏 → 独立生成最终措辞”这一整条链路上明显超过 Miru。

Miru 当前仍然最值得借鉴的部分包括：

- `AttentionEngine` 独立负责 `say or stay quiet`；
- `speak_intent` 与最终消息生成分离；
- `inner` / `user_affect` / `self_emotion` 连续状态；
- current focus；
- proactive cadence；
- unanswered proactive message awareness；
- intent queue / dedup；
- salience；
- silence / cooldown；
- 对普通共同生活场景而非只有告警/DDL进行主动靠近。

结论：

> **如果当前只验证“AI主动开口会不会更像陪伴者”，第一项 Demo 仍应优先从 Miru/AttentionEngine 思想出发，而不是先部署大模型全双工体系。**

---

## 6. MaAI 仍然是 Backchannel 首选 PoC

仓库：<https://github.com/MaAI-Kyoto/MaAI>

MaAI 0.2.0（2026-04-17）继续提供：

- turn-taking；
- backchannel timing；
- backchannel type；
- nodding；
- 英/中/日；
- CPU 可运行；
- pip 安装；
- 实时连续预测。

结合 2026 ICMI AI Clone 研究，加入实时 backchannel / nodding 后，参与者对 AI Clone 的 attentiveness、真人感与 co-presence 均显著提升。

因此它依旧是：

> **验证“主播不只会说完整句子，还会像真人一样听和轻微反应”的最低成本高价值 Demo。**

---

## 7. 当前更新后的参考优先级

| 能力层 | 当前首要参考 | 结论 |
|---|---|---|
| Dota 数据 | Dota GSI + dota-ai-coach / Dota2GSI | 不变 |
| 游戏事件语义 | chasm 思想 + 自研 Event Model | 不变 |
| 主动开口 | **Miru AttentionEngine** | 本轮未找到更优工程替代 |
| 低成本主动 Agent 库 | ProactiveAgent / THUNLP ProactiveAgent | 可辅助，但不替代 Miru |
| Backchannel | **MaAI / VAP** | 本轮未找到更轻更完整替代 |
| 中文 turn completion | **TEN Turn Detection** | **新增** |
| 实时语音 orchestration | Pipecat / LiveKit / TEN Framework | 并列按需求选择 |
| 本地全双工模型 | MiniCPM-o 4.5 / PersonaPlex / Moshi | MiniCPM-o 4.5 实用性上升 |
| 未来最终交互架构 | **Gander** | **新增并升级为首要上限参考** |
| 完整 AI Character / 游戏插件平台 | AIRI | 不变 |
| 情绪连续性 | Affect Kernel | 不变 |
| 关系连续性 | eros-engine / social-persona-engine | 新增候选，但原则不变 |
| Windows 游戏客户端 | AiGameCompanion / Tauri 外部 Overlay | 不变 |

---

## 8. 本轮后的产品技术判断

之前把核心写成：

```text
Game Event Model + Speech Policy + Companion Behavior
```

现在应进一步拆成：

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

其中最容易形成产品差异、又不能简单外包给通用 LLM 的，是：

> **Attention + Interruptibility + Reaction Arbitration + Behavior Clone。**

即：

- 什么值得反应；
- 什么时候该保持沉默；
- 什么时候只“嗯/啊/卧槽/笑一下”；
- 什么时候说完整一句；
- 什么事件先记住，等安全窗口再说；
- 这个主播在类似时刻通常如何反应；
- 与这个用户熟悉以后，反应方式如何变化。

这比单独提高 LLM 智力更接近“主播真的坐在旁边”的核心体验。

---

## 9. 建议的本地验证顺序

不要一次部署所有模块。每次只验证一个产品假设。

建议顺序：

1. **主动开口 / Silence Demo**：验证 AI 是否能在没有用户提问时，基于事件与上下文选择“说或不说”。
2. **Backchannel Demo（MaAI）**：验证“嗯、哦、笑、卧槽”等极短反馈是否显著增强共同在场感。
3. **Turn / Interruption Demo（TEN + Pipecat/LiveKit）**：验证中文说话没结束时 AI 不抢话、明确打断时能闭嘴。
4. **关系/记忆 Demo**：验证同一事件在不同历史关系下产生不同反应。
5. **Dota GSI 接入 Demo**：把前四项接到真实 Dota 事件上。
6. **全双工上限 Demo（MiniCPM-o 4.5 / Gander 路线）**：最后验证是否值得用端到端全双工模型替换模块化 pipeline。

第一项 Demo 不需要 Dota、不需要主播 Voice Clone，也不需要 Overlay；先把最核心问题单独跑通：

> **AI 能不能像一个有判断力的旁观者一样，有时主动开口，有时故意不说。**
