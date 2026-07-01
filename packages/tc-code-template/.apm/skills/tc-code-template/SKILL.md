---
name: tc-code-template
description: 获取云端管理平台代码模板。从远程 COS 存储下载 AutoPai 代码模板压缩包，解压到当前工作目录，并删除临时 zip 文件。用于快速初始化项目代码骨架。
---

# 下载并初始化 AutoPai 代码模板

## Overview

本技能从腾讯云 COS 远程存储下载 `autopai_code_template.zip` 代码模板压缩包，将其解压到当前工作目录，然后删除 zip 文件以保持目录整洁。

## When to Use

- 初始化一个新的 AutoPai 项目，需要标准代码骨架时。
- 需要获取最新的代码模板覆盖当前目录时。

Do **not** use this skill if you only need to查看模板内容而不需要实际下载解压。

## Workflow

### 1. 确认当前工作目录

确认用户期望将模板解压到的目标目录。默认使用用户的当前工作目录（workspace root）。如果用户指定了其他目录，使用用户指定的路径。

### 2. 下载压缩包

从远程 COS 存储下载代码模板 zip 文件到目标目录：

**PowerShell (Windows):**
```powershell
curl -L -o autopai_code_template.zip "https://autopai-test-1258763774.cos.ap-chengdu.myqcloud.com/sdd_poc_store/code/autopai_code_template.zip"
```

**Bash (Linux/macOS):**
```bash
curl -L -o autopai_code_template.zip "https://autopai-test-1258763774.cos.ap-chengdu.myqcloud.com/sdd_poc_store/code/autopai_code_template.zip"
```

> [!IMPORTANT]
> 下载 URL: `https://autopai-test-1258763774.cos.ap-chengdu.myqcloud.com/sdd_poc_store/code/autopai_code_template.zip`
> 该地址不可更改，是代码模板的唯一来源。

下载完成后，验证文件存在且大小大于 0：

**PowerShell:**
```powershell
Get-Item autopai_code_template.zip | Select-Object Name, Length
```

**Bash:**
```bash
ls -lh autopai_code_template.zip
```

如果文件不存在或大小为 0，**停止并报告下载失败**——不要继续解压。

### 3. 解压到当前目录

将 zip 文件内容解压到当前工作目录：

**PowerShell (Windows):**
```powershell
Expand-Archive -Path autopai_code_template.zip -DestinationPath . -Force
```

**Bash (Linux/macOS):**
```bash
unzip -o autopai_code_template.zip -d .
```

> [!WARNING]
> 解压操作会覆盖当前目录中的同名文件。执行前请确认当前目录中没有需要保留的同名文件。

### 4. 删除 zip 文件

清理临时压缩包：

**PowerShell (Windows):**
```powershell
Remove-Item autopai_code_template.zip
```

**Bash (Linux/macOS):**
```bash
rm autopai_code_template.zip
```

### 5. 报告结果

向用户报告：
- 下载是否成功
- 解压产生了哪些文件/目录（列出顶层内容）
- zip 文件是否已清理

## Red Flags

**Never:**
- 修改下载 URL 或使用其他来源替代。
- 在下载失败时继续执行解压步骤。
- 跳过删除 zip 文件的清理步骤。
- 在未确认目标目录的情况下解压到意外位置。
