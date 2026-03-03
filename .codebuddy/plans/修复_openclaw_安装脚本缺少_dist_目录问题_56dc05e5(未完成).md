---
name: 修复 openclaw 安装脚本缺少 dist 目录问题
overview: 保持 --ignore-scripts 跳过原生模块编译，在安装后手动运行 TypeScript 构建生成 dist 目录
todos:
  - id: add-build-step
    content: 在 configure_npm 函数的安装命令后添加 TypeScript 编译步骤
    status: pending
  - id: add-verify-step
    content: 添加 dist 目录验证逻辑，确保构建成功
    status: pending
    dependencies:
      - add-build-step
  - id: test-installation
    content: 测试修改后的脚本是否能正确生成 dist 目录
    status: pending
    dependencies:
      - add-verify-step
---

## 用户需求

Termux 环境下安装 openclaw 后报错：`Error: openclaw: missing dist/entry.(m)js (build output).`

## 问题分析

- `--ignore-scripts` 跳过了 postinstall → dist 目录不生成
- 但不能移除 `--ignore-scripts`，因为 koffi、clipboard 原生模块在 Termux 上编译失败
- 脚本已有 koffi stub 处理，但缺少 dist 目录构建逻辑

## 核心需求

保持 `--ignore-scripts` 安装，添加手动 TypeScript 编译步骤生成 dist 目录

## 技术方案

### 解决方案：安装后手动构建

保持 `--ignore-scripts`，在安装完成后手动运行 TypeScript 编译。

### 修改位置

**文件**: `/mnt/d/data/install-openclaw-on-termux.sh/install-openclaw-termux.sh`

在 `configure_npm` 函数的安装命令后（约第 414 行）添加构建逻辑：

### 新增代码逻辑

```
# 手动构建 dist 目录（因为 --ignore-scripts 跳过了 postinstall）
cd "$BASE_DIR"
if [ -f "tsconfig.json" ]; then
    npx tsc --skipLibCheck 2>/dev/null || npm run build 2>/dev/null
fi
# 验证 dist 目录
if [ ! -d "dist" ]; then
    echo "错误：dist 目录构建失败"
    exit 1
fi
```

### 目录结构

```
/mnt/d/data/install-openclaw-on-termux.sh/
├── install-openclaw-termux.sh  # [MODIFY] 在 configure_npm 函数中添加构建步骤
```