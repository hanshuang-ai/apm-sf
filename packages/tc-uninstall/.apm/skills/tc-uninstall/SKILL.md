---
description: 回收当前项目里 tc init 装入的产物（tc-* 技能、tkit 扩展、constitution 副本）。当用户想卸载 tc、清理 tc init 产物或移除规范驱动骨架时使用。
---

# /tc:uninstall —— 回收 tc init 产物

通过临时拉起 `tc-cli` 在用户**当前工作目录**执行卸载，回收 `tc init` 装入的产物。默认保守：只删能明确归属 tkit 的内容（tc-* 技能、tkit 装的扩展、constitution 副本），不动 spec-kit 骨架与核心命令技能。

## 执行步骤

1. **先预演**——卸载有破坏性，首选直接发**这一条** dry-run 命令让用户确认将删内容（不要先单独跑 `uv --version` 之类的预检）：

   ```
   uv tool run --from git+http://43.136.236.245:20080/shaofeiwang/tkit.git tc uninstall --dry-run $ARGUMENTS
   ```

2. **执行卸载**（用户确认后；非交互场景加 `-y`）：

   ```
   uv tool run --from git+http://43.136.236.245:20080/shaofeiwang/tkit.git tc uninstall $ARGUMENTS
   ```

   说明：
   - `$ARGUMENTS` 透传给 `tc uninstall`（如 `--all` 彻底卸载、`-y` 跳过确认、`--dry-run` 预演）。
   - `--all` 会追加移除 git 扩展、核心命令技能与 `.specify` 骨架，接近回到 init 前，破坏性更强，执行前务必向用户确认。

3. **汇报结果**：把卸载摘要（删除的技能/扩展/constitution 数量、失败项）转达用户。

## 注意

- 卸载有破坏性，尤其 `--all`。默认先 dry-run 并取得用户确认后再实删。
- 把 `tc uninstall` 的真实输出与失败项原样转达，不要吞掉。
- **仅当命令因 uv 缺失而失败**（报 `uv: command not found` / `'uv' 不是可识别的命令`）时，才先自举 uv 再重试：Windows 跑 `powershell -NoProfile -File "${CLAUDE_PLUGIN_ROOT}/bin/bootstrap.ps1"`、Unix 跑 `sh "${CLAUDE_PLUGIN_ROOT}/bin/bootstrap.sh"`，用其打印的 uv 绝对路径替换命令里的 `uv`。
