---
name: fix-npm-ld-preload-issue
overview: 在执行 `openclaw onboard` 前临时 unset LD_PRELOAD，解决 npm 安装 feishu 时找不到 /bin/npm 的问题
todos:
  - id: modify-onboard-section
    content: 修改 openclaw onboard 执行逻辑，添加 LD_PRELOAD 临时取消和恢复代码
    status: completed
---

## 需求概述

在 Termux 环境中执行 `openclaw onboard` 时，由于 `npm install @openclaw/feishu` 会报错找不到 `/bin/npm`，需要在执行前临时 `unset LD_PRELOAD`，执行完毕后恢复原值。

## 核心功能

- 在执行 `openclaw onboard` 前，保存 `LD_PRELOAD` 环境变量的当前值
- 临时取消 `LD_PRELOAD` 设置
- 执行 `openclaw onboard` 命令
- 执行完毕后，恢复 `LD_PRELOAD` 的原始值（如果之前有值）

## 技术方案

修改 `/mnt/d/data/install-openclaw-on-termux.sh/install-openclaw-termux.sh` 文件中第 891-898 行的 if 代码块。

### 实现方式

在 `openclaw onboard` 命令前后添加环境变量处理逻辑：

1. **保存原值**：`OLD_LD_PRELOAD="${LD_PRELOAD:-}"`
2. **临时取消**：`unset LD_PRELOAD`
3. **执行命令**：`openclaw onboard`
4. **恢复原值**：如果原值非空则 `export LD_PRELOAD="$OLD_LD_PRELOAD"`，否则保持 unset 状态

### 实现位置

文件第 897 行，将单行命令替换为多行处理逻辑。

### 注意事项

- 仅在执行 `openclaw onboard` 前后临时修改环境变量，不影响脚本其他部分
- 使用 subshell 执行也是一种方案，但为了保持后续状态一致性，选择直接在同一 shell 中处理