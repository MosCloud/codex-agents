# MosCloud Codex Agents

## 项目用途

本仓库保存 MosCloud 使用的全局 Codex 代理指导文本。根目录的 `AGENTS.md` 与 `~/.codex/AGENTS.md` 保持字节级一致，`archive/` 保存历史版本，便于版本管理、审阅、备份和在新环境中恢复同一套工作约束。

这份仓库内容适合安装到 Codex 的全局配置目录，也可以作为项目级 `AGENTS.md` 的参考来源；仓库不包含凭据、模型权重或机器专属数据。

## `AGENTS.md` 的核心效果

- 作用域与规则路由：全局文件只保存稳定、跨项目的原则；仓库、模型、平台、硬件、环境、版本、基准和路线图等细节由最近的项目级规则、操作手册或报告负责。
- 主代理所有权与独立判断：主代理端到端负责目标、设计、规划、风险、验收和交付，并以证据、假设和取舍为依据明确表达分歧与不确定性。
- 变更纪律：先明确设计、范围、验收标准和所有权边界，再以最小、持久且单一的实现完成已证明的需求，避免无依据的兼容路径和防御性复杂度。
- 委派与模型路由：仅将可独立界定的复杂执行工作交给子代理；工程执行使用 `gpt-5.6-luna` 与 `reasoning_effort: max`，主代理持续跟踪并在 Luna 不可用或停滞时接管。
- 子代理执行质量：提供最小相关上下文和明确的任务契约；将工作拆为原子、可验证的单元并尽早报告进展；监控超时不等于任务失败，仅在条件实质改善后重试；若进展未恢复，及时接管。
- 比例化验证与证据：根据风险和实际受影响边界选择验证范围，区分测量、推导、外推、推断和计划，并记录执行环境及验证取舍。
- 协调、环境与安全：共享文件、工作树、服务、硬件和外部操作均设置唯一所有者；诊断时区分产品缺陷与权限、沙箱或运行环境影响，并在授权范围内执行外部或破坏性操作。
- PR 与文档：围绕一个明确结果组织变更，保持标题、实现、测试和文档一致；按读者分层呈现操作说明、评审摘要和详细实验，并从关联评论中解决底层设计问题。

## 历史归档

根目录的 `AGENTS.md` 始终代表当前可安装版本。每次替换前，将旧版原样保存为 `archive/AGENTS-YYYYMMDD-<source-commit>.md`，其中日期是归档日期，短提交号标识旧版来源。归档文件用于追溯，不作为安装入口，也不随新版本改写。

## Codex 官方发现与优先级语义

Codex 会从用户全局配置和项目目录发现适用的 `AGENTS.md` 指导文本，并按目录层级合并规则。

- 全局位置：默认全局文件是 `~/.codex/AGENTS.md`；设置 `CODEX_HOME` 后，使用 `$CODEX_HOME/AGENTS.md` 作为全局位置。
- 覆盖优先级：同一目录存在 `AGENTS.override.md` 时，它优先于同目录的 `AGENTS.md`；覆盖文件不存在时才使用普通文件。
- 项目层级：规则从项目根目录向当前工作目录逐层叠加，越接近当前工作目录的文件越后生效，因此更近的规则可以细化或覆盖更远的规则。
- 加载时机：适用指导通常在每次启动新 Codex 会话时加载；更新全局或项目文件后应重新启动会话，以确保新内容被发现。

详细的官方说明见 [Codex agent configuration 文档](https://learn.chatgpt.com/docs/agent-configuration/agents-md)。

## 安装、备份、更新和验证

首次使用私有仓库时先完成 GitHub CLI 登录，然后将仓库克隆到本地目录：

```bash
gh auth login
gh repo clone MosCloud/codex-agents /path/to/codex-agents
```

首次安装或覆盖全局文件前，先创建配置目录并为现有的 `~/.codex/AGENTS.md` 创建带时间戳的本地备份；如果文件不存在则只创建目录：

```bash
CODEX_DIR="${CODEX_HOME:-$HOME/.codex}"
mkdir -p "$CODEX_DIR"
if test -f "$CODEX_DIR/AGENTS.md"; then
  cp -- "$CODEX_DIR/AGENTS.md" "$CODEX_DIR/AGENTS.md.bak.$(date +%Y%m%d%H%M%S)"
fi
```

备份完成后，再将仓库根目录的指导文件安装到全局配置目录：

```bash
CODEX_DIR="${CODEX_HOME:-$HOME/.codex}"
install -m 0644 /path/to/codex-agents/AGENTS.md "$CODEX_DIR/AGENTS.md"
```

更新仓库并重新安装最新指导文件：

```bash
git -C /path/to/codex-agents pull --ff-only
CODEX_DIR="${CODEX_HOME:-$HOME/.codex}"
install -m 0644 /path/to/codex-agents/AGENTS.md "$CODEX_DIR/AGENTS.md"
```

验证仓库文件与已安装全局文件完全一致，并检查 Git 补丁格式：

```bash
CODEX_DIR="${CODEX_HOME:-$HOME/.codex}"
cmp --silent /path/to/codex-agents/AGENTS.md "$CODEX_DIR/AGENTS.md"
sha256sum /path/to/codex-agents/AGENTS.md "$CODEX_DIR/AGENTS.md"
git -C /path/to/codex-agents diff --check
```

## 适用边界

- 全局 `AGENTS.md` 只规定跨项目工作原则；项目、模型、平台和环境的具体要求应放在适用范围更近的项目级文件或专用文档中。
- `AGENTS.md` 是给代理使用的指导文本，不是安全边界，不能替代操作系统权限、容器隔离、网络策略、审批流程或密钥管理。
- 项目级和更接近当前工作目录的规则可以覆盖或补充全局规则，因此安装全局文件不会取消项目维护者的本地约束。
- 指导文本可以要求使用某个模型或路由，但实际模型是否可用取决于当前 Codex 环境、账户权限、配置和服务状态；若 `gpt-5.6-luna` 不可用，主代理应直接执行工程任务或明确报告限制，不能静默替换模型。
- 规则用于约束代理行为，不保证第三方工具、远程服务或外部系统始终按预期工作；重要变更仍需要独立验证。

---

## Purpose

This repository stores MosCloud's global Codex agent guidance. The root `AGENTS.md` is byte-for-byte identical to `~/.codex/AGENTS.md`, while `archive/` retains prior versions for review, backup, restoration, and reproducibility.

The repository content can be installed into Codex's global configuration directory or used as a source for project-level `AGENTS.md` files; it contains no credentials, model weights, or machine-specific data.

## Core effects of `AGENTS.md`

- Scope and routing: the global file contains stable cross-project principles; repository, model, platform, hardware, environment, version, benchmark, and roadmap details belong in the nearest project guidance, playbook, or report.
- Primary-agent ownership and independent judgment: the primary agent owns the objective, design, planning, risk, acceptance, and delivery end to end, grounding decisions in evidence, assumptions, and tradeoffs while stating disagreement and uncertainty clearly.
- Change discipline: define design, scope, acceptance criteria, and ownership before substantial work, then satisfy the demonstrated requirement through the smallest durable and canonical implementation without speculative compatibility paths or defensive complexity.
- Delegation and model routing: delegate only independently bounded complex execution work; engineering execution uses `gpt-5.6-luna` with `reasoning_effort: max`, while the primary agent monitors progress and takes over when Luna is unavailable or stalled.
- Subagent execution quality: provide the minimum relevant context and an explicit task contract; make work atomic and verifiable with early progress; treat monitoring timeouts as distinct from task failure and retry only after materially improved conditions; take over promptly if progress does not resume.
- Proportional validation and evidence: select validation according to risk and affected boundaries, distinguish measurement from derivation, extrapolation, inference, and plans, and record execution context and validation tradeoffs.
- Coordination, environment, and safety: assign a single owner to shared files, worktrees, services, hardware, and external operations; separate product defects from permission, sandbox, and runtime effects, and keep external or destructive actions within granted authority.
- Pull requests and documentation: organize each change around one outcome, keep title, implementation, tests, and documentation aligned, separate operator guidance from review evidence and detailed experiments, and resolve the design concern underlying related comments.

## Historical archive

The root `AGENTS.md` is always the current installable version. Before replacing it, preserve the prior file unchanged as `archive/AGENTS-YYYYMMDD-<source-commit>.md`, where the date is the archive date and the short commit identifies the prior version. Archive files are immutable history, not installation targets.

## Official Codex discovery and precedence

Codex discovers applicable `AGENTS.md` guidance from the user's global configuration and project directories, then combines it according to directory precedence.

- Global location: the default global file is `~/.codex/AGENTS.md`; when `CODEX_HOME` is set, `$CODEX_HOME/AGENTS.md` is the global location.
- Override precedence: `AGENTS.override.md` takes precedence over `AGENTS.md` in the same directory; the ordinary file is used when the override file is absent.
- Project layering: rules are accumulated from the project root toward the current working directory, with closer files taking effect later, so a closer rule can refine or override a more distant one.
- Loading timing: applicable guidance is usually loaded when each new Codex session starts; restart the session after changing global or project files so the new content is discovered.

See the [official Codex agent configuration documentation](https://learn.chatgpt.com/docs/agent-configuration/agents-md) for the detailed semantics.

## Install, back up, update, and verify

Authenticate the GitHub CLI before cloning this private repository for the first time, then clone it to a local directory:

```bash
gh auth login
gh repo clone MosCloud/codex-agents /path/to/codex-agents
```

Before the first installation or replacement, create the configuration directory and make a timestamped local backup of an existing `~/.codex/AGENTS.md`; when the file does not exist, this only creates the directory:

```bash
CODEX_DIR="${CODEX_HOME:-$HOME/.codex}"
mkdir -p "$CODEX_DIR"
if test -f "$CODEX_DIR/AGENTS.md"; then
  cp -- "$CODEX_DIR/AGENTS.md" "$CODEX_DIR/AGENTS.md.bak.$(date +%Y%m%d%H%M%S)"
fi
```

After the backup completes, install the guidance file from the repository root into the global configuration directory:

```bash
CODEX_DIR="${CODEX_HOME:-$HOME/.codex}"
install -m 0644 /path/to/codex-agents/AGENTS.md "$CODEX_DIR/AGENTS.md"
```

Update the repository and reinstall the latest guidance:

```bash
git -C /path/to/codex-agents pull --ff-only
CODEX_DIR="${CODEX_HOME:-$HOME/.codex}"
install -m 0644 /path/to/codex-agents/AGENTS.md "$CODEX_DIR/AGENTS.md"
```

Verify that the repository file and installed global file are identical, then check the Git patch format:

```bash
CODEX_DIR="${CODEX_HOME:-$HOME/.codex}"
cmp --silent /path/to/codex-agents/AGENTS.md "$CODEX_DIR/AGENTS.md"
sha256sum /path/to/codex-agents/AGENTS.md "$CODEX_DIR/AGENTS.md"
git -C /path/to/codex-agents diff --check
```

## Applicability boundaries

- The global `AGENTS.md` defines cross-project working principles only; project, model, platform, and environment specifics belong in more narrowly scoped project files or dedicated documentation.
- `AGENTS.md` is guidance for an agent, not a security boundary; it cannot replace operating-system permissions, container isolation, network policy, approval workflows, or secret management.
- Project-level rules and rules closer to the current working directory can override or supplement global rules, so installing the global file does not remove constraints maintained by a project owner.
- Guidance can request a particular model or route, but model availability depends on the current Codex environment, account permissions, configuration, and service state; when `gpt-5.6-luna` is unavailable, the primary agent should execute engineering work directly or report the limitation, without silently substituting another model.
- Rules constrain agent behavior but do not guarantee that third-party tools, remote services, or external systems will behave as expected; important changes still require independent verification.
