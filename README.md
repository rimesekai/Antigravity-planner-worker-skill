# Antigravity Planner-Worker Skill

<p align="center">
  <a href="#english">English</a> | <a href="#简体中文">简体中文</a>
</p>

---

<a id="english"></a>

## Planner-Worker Workflow Skill (English)

> [!NOTE]
> **Project Status & Comparison with Native `Orchestrator`**: Google Antigravity's native **Orchestrator** mode is rolling out rapidly (including support for Pro-tier users) and covers most of this skill's core quota-saving capabilities. The **only remaining differences** between this skill and native `Orchestrator` are:
> 1. **Claude as Lead Model**: Allows using **Claude Opus/Sonnet** (or other non-Gemini high-reasoning models) as the Lead Architect to orchestrate Flash workers.
> 2. **Quota vs. Quality Trade-off**: Because this skill strictly enforces offloading execution to subagents, it **may save even more high-tier quota** than native `Orchestrator`; however, **its overall output quality and fluidity may fall short of native `Orchestrator`**.
>
> Active maintenance will wind down as native orchestration continues to mature.

> **Core Motivation**: In Google Antigravity, **high-tier flagship model quotas** (such as Claude Opus/Sonnet or Gemini Pro) are precious and limited, while **Gemini Flash quotas are abundant and blazingly fast**. This skill is designed to **drastically save your high-tier quotas and fully leverage abundant Flash quotas—while simultaneously improving overall task quality** through structured architectural planning and independent final auditing.

An open-source **Skill** designed for **Google Antigravity (AGY)** that implements a complete software engineering collaboration workflow: **High-Reasoning Architect Planning + Ultra-Fast Worker Execution + Independent Read-Only Final Audit + Self-Evolving Knowledge Base**.

Instead of burning expensive high-tier tokens on repetitive coding, boilerplate writing, and trial-and-error test runs—or letting a lightweight model tackle complex architecture alone and drift—the lead model focuses strictly on architectural decomposition, concurrency conflict prevention, acceptance review, and fallback takeover. Heavy implementation is offloaded to parallel Gemini Flash subagents, and before major deliveries, an independent, clean-context Pro subagent performs a strict read-only blind audit to guarantee top-tier code quality.

### Why This Skill Instead of Native `/swarm`?

| Dimension | Antigravity Native `/swarm` | This `planner-worker` Skill |
|---|---|---|
| **Model & Cost Strategy** | Homogeneous by default (spawns multiple instances of your expensive main model); marked as **Quota Heavy**. | Heterogeneous by design: strictly locks execution workers to cheap, abundant **`flash`**, reserving high-tier tokens strictly for planning and major audits. |
| **Communication Overhead** | Shared message board with broadcast wakeups ($O(N^2)$ token growth) and heavy coordination turns (`narrator`, `monitor`). | Lightweight star-topology delegation via `invoke_subagent`; workers return a compact 5-item summary directly to the Lead Architect. |
| **Best Use Case** | Large-scale autonomous multi-agent exploration across isolated git worktrees. | High-frequency daily engineering, refactoring, feature implementation, and test generation where quota efficiency and determinism matter. |

### Key Features

1. **Three-Pillar Collaborative Architecture**
   - **Lead Architect (Main Model, e.g., Claude Opus/Sonnet or Gemini Pro)**: Handles requirement analysis, atomic task decomposition, interface contract definition, stage verification, and fallback takeover.
   - **Execution Worker (Gemini Flash, `Model: "flash"`)**: Executes single-responsibility coding, file edits, build commands, and unit tests at high speed and minimal cost.
   - **Independent Auditor (Pro Code Auditor, `Model: "pro"`)**: Triggered when changes span 4+ files or introduce core/security-sensitive modules, using a fresh cold context defaulting to `TypeName="research"` (a physical read-only sandbox). Strictly prohibited from modifying files, outputting only a P0/P1 inspection checklist.

2. **Industrial-Grade Guardrails & Circuit Breakers**
   - **Concurrent Write Isolation**: Parallel workers in the same batch are strictly forbidden from editing overlapping files, preventing race-condition overwrites.
   - **Two-Tier Circuit Breaker**: A single worker stops after 2 failed local fix attempts and reports a `[BLOCKED]` summary; if the same atomic task hits 2 blocked rounds in total, the Lead Architect immediately takes over directly instead of looping.
   - **Compact Structured Reporting**: Workers report back using a strict 5-item schema, minimizing context bloat for the Lead Architect.

3. **Self-Evolving Knowledge Base (Phase 5)**
   - Automatically captures reusable pitfall-avoidance rules after complex tasks (and remains silent when no new generalizable rule emerges).
   - **Capacity Cap & Deduplication**: Strictly capped at **10 rules total (max 4 per sub-section)**. When exceeding the cap, existing rules must be consolidated, deduplicated, and abstracted into higher-level contracts before appending new ones—preventing prompt bloat and attention dilution.

### Prerequisites

- **Platform**: Google Antigravity (AGY)
- **Tool Dependencies**: Tailored for Antigravity's native `invoke_subagent` (`Model` tiers `"flash"` / `"pro"`, `TypeName` supporting `"self"` and read-only `"research"`) and `manage_subagents`. Adaptations may be required for other agent frameworks.

### Installation & Usage

#### Option 1: Git Clone (Recommended)

```bash
# macOS / Linux
git clone https://github.com/rimesekai/Antigravity-planner-worker-skill.git ~/.gemini/config/skills/planner-worker

# Windows (PowerShell)
git clone https://github.com/rimesekai/Antigravity-planner-worker-skill.git "$env:USERPROFILE\.gemini\config\skills\planner-worker"
```

#### Option 2: Manual Setup

1. Download `SKILL.md` from this repository.
2. Place it into `~/.gemini/config/skills/planner-worker/SKILL.md` (on Windows: `C:\Users\<YourUsername>\.gemini\config\skills\planner-worker\SKILL.md`).

#### Activation

Once installed, simply prompt Antigravity with phrases like:
- "Use planner-worker workflow to implement..."
- "Delegate coding tasks to Flash and build..."

### Workflow Overview

1. **Phase 1: Analysis & Task Breakdown** - Explore context and enforce single-file ownership across parallel tasks.
2. **Phase 2: Dispatch to Gemini Flash** - Inject only 1–2 relevant rules; enforce 2-attempt local retry limit.
3. **Phase 3: Review & Verification** - Accept work or trigger Lead Architect takeover on the 2nd global block.
4. **Phase 4: Final Audit & Delivery** - For 4+ file changes or core modules, spin up `Model: "pro"` + `research` sandbox for read-only audit, then summarize delivery.
5. **Phase 5: Skill Self-Evolution** - Triggered only when new generalizable lessons arise; consolidate & deduplicate if over 10 rules.

---

<a id="简体中文"></a>

## Planner-Worker Skill (简体中文)

> [!NOTE]
> **项目状态与官方原生 `Orchestrator` 的仅存区别**：Google Antigravity 官方的原生 **Orchestrator** 功能已逐步向 Pro 用户开放，基本覆盖了本 Skill 的核心省配额调度能力。目前本 Skill 与官方 `Orchestrator` **仅剩以下两点区别**：
> 1. **支持使用 Claude 作为主模型**：本 Skill 允许你继续使用 **Claude Opus / Sonnet** 等非 Gemini 模型担任总架构师来调度 Flash 干活。
> 2. **额度与质量的客观权衡**：由于本 Skill 采用强制派发机制，**也许能比官方 `Orchestrator` 省下更多高阶额度**；但受限于应用层提示词转发，**其整体任务质量可能会不如原生底层的 `Orchestrator`**。
>
> 随着官方原生能力不断完善，本项目也将逐步停止维护。

> **设计初衷与核心目的**：在 Google Antigravity 中，**高阶旗舰模型（如 Claude Opus/Sonnet 或 Gemini Pro 等）的额度十分宝贵且有限，而 Gemini Flash 的额度则非常充沛且响应极快**。本 Skill 旨在**大幅节省昂贵的高阶模型额度、充分利用充沛的 Flash 额度**，同时通过严谨的架构拆解与独立终审机制，**反而进一步提高复杂任务的最终交付质量**。

这是一个专为 **Google Antigravity (AGY)** 设计的开源 Skill（技能），实现了**“高推理架构师规划 + 极速小模型执行 + 独立大模型只读终审 + 经验库自迭代”**的完整软件工程协同工作流。

如果让昂贵的高阶主模型亲自编写所有样板代码、跑测试、修琐碎报错，配额很快就会见底；而如果全程只用轻量模型单打独斗，复杂工程又极易跑偏。本工作流让高阶主模型“把好钢用在刀刃上”，专注于全局拆解、接口契约、并发避碰与兜底把关；把繁重编码与跑测下放给可并行的 Gemini Flash 子智能体；最后在重大变更交付前再由独立干净上下文的 Pro 智能体执行只读盲审——真正实现**省配额与高质量的完美双赢**。

### 为什么不直接用官方内置 `/swarm`？

| 对比维度 | 官方原生 `/swarm` | 本 `planner-worker` 技能 |
|---|---|---|
| **模型与成本策略** | 默认同构模型并行（你选了什么高阶模型，每个 Worker 就用什么高阶模型），官方明确标注为 **Quota Heavy（高配额消耗）**。 | 异构分工架构：严格锁死执行端仅使用低成本且充沛的 **`flash`**，高阶额度只花在最关键的架构规划与重大终审上。 |
| **通信开销与延迟** | 基于公共看板的全员广播（每发一条消息所有成员都可能唤醒重读），还需配套 `narrator`/`monitor`，调度链条重。 | 轻量极速的星型单向委派，仅用 `invoke_subagent` 瞬发调用；Worker 仅回传 5 项结构化摘要，零广播浪费。 |
| **适用场景** | 超大型工程的多角色自治协作与多工作区探索。 | 日常高频业务开发、模块化重构、单元测试补全等**追求极致省额度和高确定性**的开发场景。 |

### 核心特性

1. **铁三角分级协同架构**
   - **总架构师（当前高推理主模型，如 Claude Opus/Sonnet 或 Gemini Pro）**：负责需求剖析、原子化任务拆解、接口契约声明、阶段验收与兜底接管。
   - **执行者（Gemini Flash Worker，`Model: "flash"`）**：负责高速执行单一编码、文件改写、终端构建与单元测试。
   - **独立质检员（Pro Code Auditor，`Model: "pro"`）**：当改动跨越 4 个及以上文件、涉及核心/安全敏感模块时唤起全新冷上下文的独立 Pro 实例，默认运行于 `TypeName="research"` 物理只读沙箱，严禁修改文件，只输出 P0/P1 体检清单（1~3 个文件的常规改动由主模型直接审核 `git diff`，省去额外开支）。

2. **工业级防御与熔断机制**
   - **并发写冲突隔离**：同一批次并行的 Worker 严禁修改重叠文件，消除竞态覆盖风险。
   - **双层熔断止损**：Worker 单次派发就地修复最多尝试 2 次即触发熔断；同一原子任务第 2 次收到阻断报告时，主模型直接接管修复，避免死循环空耗配额。
   - **结构化极简回传**：强制规范 Worker 的 5 项汇报格式，极大降低主模型验收时的上下文负担。

3. **经验库自进化与防膨胀机制（Phase 5）**
   - 仅在任务结束后沉淀出新的可复用避坑经验时，才自动更新 `SKILL.md` 的经验库。
   - 内置容量上限控制：**经验库总量严格控制在 10 条以内（单小节不超过 4 条）**，超限前强制触发“合并、去重与抽象提炼”，防止规则越写越长导致注意力稀释。

### 适用环境与前置要求

- **运行平台**：Google Antigravity (AGY)
- **依赖工具**：本 Skill 的执行规范深度适配 Antigravity 内置的 `invoke_subagent`（`Model` 枚举 `"flash"` / `"pro"`、`TypeName` 支持 `"self"` 与 `"research"` 只读沙箱）以及 `manage_subagents` 工具。其它不支持子智能体分级调度的工具可能需要酌情修改参数。

### 安装与使用

#### 方法一：Git 一键克隆安装（推荐）

```bash
# macOS / Linux
git clone https://github.com/rimesekai/Antigravity-planner-worker-skill.git ~/.gemini/config/skills/planner-worker

# Windows (PowerShell)
git clone https://github.com/rimesekai/Antigravity-planner-worker-skill.git "$env:USERPROFILE\.gemini\config\skills\planner-worker"
```

#### 方法二：手动下载放置

1. 下载本仓库中的 `SKILL.md` 文件（使用精简高效的英文提示词编写以节省 Token，且内置了“自动适配用户对话语言”指令，中文对话同样全中文回复与沉淀）。
2. 将其放入本地目录：`~/.gemini/config/skills/planner-worker/SKILL.md`（Windows 路径为 `C:\Users\<你的用户名>\.gemini\config\skills\planner-worker\SKILL.md`）。

#### 如何激活

安装完成后，在对话中直接输入以下任一指令即可自动触发本工作流：
- “用 planner worker 模式帮我开发……”
- “指挥 Flash 干活，帮我实现……”
- “按照分工执行工作流重构这个模块……”

### 工作流概览

1. **Phase 1: 需求剖析与原子化拆解**（确认单文件归属唯一性，隔离并行写冲突）
2. **Phase 2: 派发执行任务**（严格指定 `Model: "flash"`，按需注入 1~2 条经验，附带 2 次重试熔断）
3. **Phase 3: 审查与验收**（第 2 次阻断由主模型亲自接管兜底）
4. **Phase 4: 独立终审与交付总结**（4 个以上文件或核心模块触发 `Model: "pro"` + `research` 沙箱执行只读体检）
5. **Phase 5: 复盘与自迭代**（仅在有新通用坑点时触发，先合并去重再沉淀入库）

---

## License

This project is open-sourced under the [MIT License](./LICENSE).
