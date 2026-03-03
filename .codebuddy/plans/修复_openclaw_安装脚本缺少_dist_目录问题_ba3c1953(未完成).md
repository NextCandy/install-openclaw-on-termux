---
name: 修复 openclaw 安装脚本缺少 dist 目录问题
overview: 修复 `--ignore-scripts` 导致 openclaw 缺少 dist/ 构建输出的问题，通过分离安装和构建步骤来解决
todos:
  - id: analyze-build-process
    content: 分析 openclaw 的构建流程，确认 postinstall 是否包含 TypeScript 编译
    status: pending
  - id: modify-install-commands
    content: 移除三处 --ignore-scripts 参数，保留 NODE_LLAMA_CPP_SKIP_DOWNLOAD 环境变量
    status: pending
  - id: add-fallback-build
    content: 在 configure_npm 函数中添加构建验证和手动构建回退逻辑
    status: pending
  - id: test-installation
    content: 验证修改后的安装流程是否能正确生成 dist 目录
    status: pending
---

## 用户需求

用户在 Termux 环境下安装 openclaw 后，运行命令报错：`Error: openclaw: missing dist/entry.(m)js (build output).`

## 问题分析

脚本第 362、378、402 行使用 `--ignore-scripts` 跳过了所有 npm 生命周期脚本，包括 postinstall。openclaw 需要运行 postinstall 脚本来构建 `dist/` 目录，导致构建输出文件缺失。

## 核心需求

修改安装脚本，在保持 Termux 兼容性的同时，确保 openclaw 的 dist 目录能够正确生成。

## 技术方案

### 问题根因

1. `--ignore-scripts` 跳过了所有 npm 脚本，包括 openclaw 的 postinstall
2. openclaw 的 postinstall 负责 TypeScript 编译，生成 `dist/entry.js` 或 `dist/entry.mjs`
3. 脚本后续的 `apply_patches` 函数依赖 dist 目录存在

### 解决方案：两阶段安装

**不使用 `--ignore-scripts`**，改用更精细的控制：

1. **正常安装**：移除 `--ignore-scripts`，让 postinstall 正常运行
2. **环境变量控制**：保持 `NODE_LLAMA_CPP_SKIP_DOWNLOAD=true` 跳过 node-llama-cpp
3. **容错处理**：如果 postinstall 失败，手动运行构建命令

### 修改点

| 位置 | 修改内容 |
| --- | --- |
| 第 362 行 | 移除 `--ignore-scripts` |
| 第 378 行 | 移除 `--ignore-scripts` |
| 第 402 行 | 移除 `--ignore-scripts`，添加错误处理 |


### 备选方案（如果上述方案失败）

在安装后添加手动构建步骤：

```
cd "$BASE_DIR" && npm run build 2>/dev/null || \
  npm rebuild openclaw 2>/dev/null || \
  node scripts/build.js 2>/dev/null
```