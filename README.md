# MosCloud Codex Agents

## 项目用途

本仓库保存 MosCloud 使用的全局 Codex 代理指导文本，根目录的 `AGENTS.md` 与 `~/.codex/AGENTS.md` 保持字节级一致，便于版本管理、审阅、备份和在新环境中恢复同一套工作约束。

这份仓库内容适合安装到 Codex 的全局配置目录，也可以作为项目级 `AGENTS.md` 的参考来源；仓库不包含凭据、模型权重或机器专属数据。

## `AGENTS.md` 的核心效果

- 主代理职责：主代理负责理解目标、分析证据、设计方案、规划范围、评估风险与取舍、验收结果和最终报告，并可直接完成轻量检查和小范围编辑。
- 独立判断：建议必须基于证据、假设、风险和取舍，而不是迎合偏好；证据不足、存在分歧或不确定性时要明确说明。
- 复杂任务委派与模型路由：能够拆分的复杂工作交给子代理并限定边界；工程执行任务（包括实现、profiling、服务操作、测试、基准测试和机械性改动）必须使用 `gpt-5.6-luna` 与 `reasoning_effort: max`；Luna 不可用时由主代理直接执行，或明确报告限制，不静默替换模型；研究与分析任务的模型由主代理选择。
- 协调与沙箱诊断：为共享工作树、服务、硬件和外部操作分配唯一所有者；诊断异常时区分产品缺陷与 Codex 沙箱的文件系统、网络、系统调用、进程或设备限制，并按审批流程进行必要的非沙箱检查。

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

- `AGENTS.md` 是给代理使用的指导文本，不是安全边界，不能替代操作系统权限、容器隔离、网络策略、审批流程或密钥管理。
- 项目级和更接近当前工作目录的规则可以覆盖或补充全局规则，因此安装全局文件不会取消项目维护者的本地约束。
- 指导文本可以要求使用某个模型或路由，但实际模型是否可用取决于当前 Codex 环境、账户权限、配置和服务状态；若 `gpt-5.6-luna` 不可用，主代理应直接执行工程任务或明确报告限制，不能静默替换模型。
- 规则用于约束代理行为，不保证第三方工具、远程服务或外部系统始终按预期工作；重要变更仍需要独立验证。

---

## Purpose

This repository stores MosCloud's global Codex agent guidance. The root `AGENTS.md` is byte-for-byte identical to `~/.codex/AGENTS.md`, making the instructions easy to review, back up, restore, and reproduce in a new environment.

The repository content can be installed into Codex's global configuration directory or used as a source for project-level `AGENTS.md` files; it contains no credentials, model weights, or machine-specific data.

## Core effects of `AGENTS.md`

- Primary-agent responsibility: the primary agent owns goal understanding, evidence-based analysis, design, scope planning, risk and tradeoff decisions, acceptance, and the final report, while still handling lightweight checks and small targeted edits directly.
- Independent judgment: recommendations must be grounded in evidence, assumptions, risks, and tradeoffs rather than preference; weak evidence, disagreement, and uncertainty must be stated clearly.
- Complex-task delegation and model routing: divisible complex work is assigned to child agents with explicit boundaries; engineering execution tasks, including implementation, profiling, service operations, tests, benchmarks, and mechanical changes, must use `gpt-5.6-luna` with `reasoning_effort: max`; when Luna is unavailable, the primary agent executes the work directly or reports the limitation, without silently substituting another model; the primary agent chooses the model for research and analysis tasks.
- Coordination and sandbox diagnostics: shared worktrees, services, hardware, and external operations have a single owner; unexpected behavior is diagnosed by separating product defects from Codex sandbox restrictions on filesystems, networks, system calls, processes, or devices, using the approval flow for necessary unsandboxed checks.

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

- `AGENTS.md` is guidance for an agent, not a security boundary; it cannot replace operating-system permissions, container isolation, network policy, approval workflows, or secret management.
- Project-level rules and rules closer to the current working directory can override or supplement global rules, so installing the global file does not remove constraints maintained by a project owner.
- Guidance can request a particular model or route, but model availability depends on the current Codex environment, account permissions, configuration, and service state; when `gpt-5.6-luna` is unavailable, the primary agent should execute engineering work directly or report the limitation, without silently substituting another model.
- Rules constrain agent behavior but do not guarantee that third-party tools, remote services, or external systems will behave as expected; important changes still require independent verification.
