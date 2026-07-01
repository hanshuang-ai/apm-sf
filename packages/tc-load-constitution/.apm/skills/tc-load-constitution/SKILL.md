---
name: tc-load-constitution
description: 在SDD项目需要加载或应用宪章时使用。当输入参数为空时，确定性地将默认宪章拷入且注入技术规约指针；当输入参数不为空时，转为调用 tc.constitution 宪章技能来映射加载宪章。
---

# 加载宪章（tc-load-constitution）

## Overview

在SDD项目需要应用宪章时，智能体根据用户触发时的输入参数是否为空，智能选择不同的加载路径：
- **输入参数为空**：代表使用默认的内置宪章，由脚本/手工进行确定性拷入 `.specify/memory/constitution.md` 并注入技术规约 `ARCHITECTURE_CONTRACT.md` 的指针，快速且无 LLM 改写风险。
- **输入参数不为空**：代表需要自定义加载或映射特定的宪章。此时转为调用 `tc.constitution` 技能，交由 LLM 完成宪章映射。

## When to Use

- 初始化 spec-kit 项目需要应用宪章，或更新宪章配置时。

## Workflow

根据输入参数的值采取不同路径：

### 路径 A：输入参数为空（确定性加载默认宪章）

如果不提供任何输入参数，智能体应当使用快速确定性加载逻辑：

#### 1. 运行脚本

脚本随 `ext` 扩展安装在项目内，按平台二选一：

- Unix/macOS：`bash .specify/extensions/ext/scripts/bash/load-constitution.sh`
- Windows：`pwsh -File .specify/extensions/ext/scripts/powershell/load-constitution.ps1`

脚本行为：
1. 校验 `.specify/tc/constitution/constitution.md` 存在；缺失则报错并以非零码退出（**不要**伪造路径或编造宪章内容）。
2. 确保 `.specify/memory/` 存在，拷贝宪章覆盖写入 `.specify/memory/constitution.md`。
3. 若 `ARCHITECTURE_CONTRACT.md` 存在，把指针写入上下文文件的托管块（`<!-- tc:contracts:start -->`…`<!-- tc:contracts:end -->`）：已存在该块则整块替换，否则追加；缺失技术规约只告警不阻断。
4. 上下文目标：更新项目根下已存在的 `CLAUDE.md` / `AGENTS.md` / `GEMINI.md` / `CODEBUDDY.md`；若一个都没有，则新建 `CLAUDE.md`。

#### 2. 兜底（脚本不可用时）

若脚本文件不存在（例如 `ext` 扩展未安装），按其逻辑**手工**完成同样两步：拷贝默认宪章覆盖 `.specify/memory/constitution.md`；把上面格式的托管块幂等写入上下文文件。仍然：宪章源缺失就停下报告，绝不编造。

#### 3. Report

向用户汇报：默认宪章是否已写入 `.specify/memory/constitution.md`，技术规约指针写进了哪些上下文文件，以及任何告警（如技术规约缺失）。

### 路径 B：输入参数不为空（调用 tc.constitution 映射宪章）

如果传入了特定的输入参数（例如指定了特定的宪章路径、自定义要求、配置参数等）：

#### 1. 调用 tc.constitution 宪章技能
智能体**必须跳过**拷贝脚本，转而调用底层内置的 `tc.constitution` 技能（在会话中直接以该输入参数调用 `/tc.constitution <参数>` ），交由该技能背后的 LLM 引擎对宪章进行生成、渲染与落盘。

#### 2. Report
根据 `tc.constitution` 技能的最终落盘与反馈结果向用户汇报。

## Red Flags

**Never:**
- 当输入参数为空时，依然调用 `tc.constitution` —— 这会引入不必要的 LLM 改写开销和风险。
- 当输入参数不为空时，直接运行拷贝脚本 —— 这会忽略用户传入的个性化参数。
- 在宪章源缺失时编造路径或宪章内容——停下并报告。
- 手工兜底时改写、精简或"优化"宪章内容——必须 verbatim 拷贝。
- 把技术规约 `ARCHITECTURE_CONTRACT.md` 的全文内联进宪章或上下文——只写指针。
- 在托管块之外重复堆叠指针——同一块必须整块替换以保持幂等。
