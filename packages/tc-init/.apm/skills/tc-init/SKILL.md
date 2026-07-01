---
description: 在当前项目做完整 tc 初始化（项目骨架 + tc-ext-* 扩展 + constitution + tc-* 技能）。当用户想初始化项目、搭建规范驱动开发骨架、或装配 tc 技能时使用。
---

# /tc:init —— 完整 tc 初始化

通过临时拉起 `tc-cli` 在用户**当前工作目录**执行完整初始化。产物与本地 `tc init` 完全一致：项目骨架（`/tc.*` 核心命令）、`tc-ext-*` 扩展、constitution 副本、16 个 `tc-*` 技能。

## 执行步骤

1. **直接执行初始化**——首选只发**这一条命令**（在当前工作目录运行，**不要切目录**）：

   ```
   uv tool run --from git+http://43.136.236.245:20080/shaofeiwang/tkit.git tc init --force $ARGUMENTS
   ```

   要点：
   - **默认带 `--force`**：插件场景下当前目录必然已有 `.claude/`（常常还有 `.omc/`），目录非空时初始化会中止并报 `Aborted.`；`--force` 表示"合并进已存在目录"，正是初始化所需。**不要因为没加 `--force` 撞错再去逐步排查——一开始就带上。**
   - **无需手动加 `--script`**：tc 在 Windows 自动用 `ps`、其它平台用 `sh`。
   - `uv tool run` 会自动补齐 Python≥3.11，临时拉起 `tc-cli` 及其依赖，无需持久安装。
   - `$ARGUMENTS` 透传给 `tc init`（如 `--integration claude`）；用户未指定时保持默认。
   - 首次运行需下载依赖，可能耗时数十秒，属正常。
   - **尽量一次只发这一条命令**，避免多次权限确认打断用户。

2. **仅当上一步因 uv 缺失而失败**（报 `uv: command not found` / `'uv' 不是可识别的命令`）时，才自举 uv 再重试。按操作系统运行自举脚本，它**最后一行打印可用的 uv 绝对路径**：
   - Windows：`powershell -NoProfile -File "${CLAUDE_PLUGIN_ROOT}/bin/bootstrap.ps1"`
   - Unix（Linux/macOS）：`sh "${CLAUDE_PLUGIN_ROOT}/bin/bootstrap.sh"`

   然后把第 1 步命令里的 `uv` 替换成该绝对路径（如 `~/.local/bin/uv`）重跑。自举用 uv 官方安装器（需公网）；若自举也失败，把报错原样转达用户并停止。

3. **汇报结果**：把 `tc init` 输出的成功看板（项目名、引擎、集成、已装技能数）转达用户。

4. **引导下一步**：提示用户在当前 AI 会话中运行 `/tc-load-constitution` 完成项目立宪。

## 注意

- 若 `tc init` 返回非零或打印初始化失败，把它的真实报错原样转达用户，不要吞掉或臆测原因。
- 本技能只负责"跑 tc init"，不替代或重写其任何逻辑。
