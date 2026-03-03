---
name: 修复 openclaw 安装脚本 - 参考官方 Android 方案
overview: 参考 OpenClaw-Android 项目，通过 GYP 配置和环境变量解决原生模块编译问题，移除 --ignore-scripts
todos:
  - id: add-gyp-config
    content: 在 configure_npm 函数中添加 GYP 配置和环境变量设置
    status: pending
  - id: add-compile-deps
    content: 在 check_deps 函数中添加 clang、ninja、pkg-config 依赖
    status: pending
  - id: remove-ignore-scripts
    content: 移除三处 npm install 命令中的 --ignore-scripts 参数
    status: pending
  - id: test-installation
    content: 验证修改后的脚本能否正确生成 dist 目录
    status: pending
---

## 用户需求

Termux 环境下安装 openclaw 后运行命令报错：`Error: openclaw: missing dist/entry.(m)js (build output).`

## 问题根因

脚本使用 `--ignore-scripts` 跳过了 npm 生命周期脚本，导致 openclaw 的 postinstall 未运行，dist 目录未生成。但直接移除 `--ignore-scripts` 会导致 koffi、clipboard 等原生模块编译失败。

## 核心需求

参考 OpenClaw-Android 项目，通过 GYP 配置和环境变量解决原生模块编译问题，让 postinstall 正常运行生成 dist 目录。

## 技术方案

### 解决方案：GYP 配置 + 移除 --ignore-scripts

参考 OpenClaw-Android 项目的成功做法：

1. 在安装前创建 `~/.gyp/include.gypi` 配置文件，指定编译器路径
2. 设置 `TMPDIR`、`TMP`、`TEMP` 环境变量
3. 确保安装了必要的编译工具（clang、ninja、pkg-config）
4. 移除 `--ignore-scripts`，让 postinstall 正常运行

### 修改位置

**文件**: `/mnt/d/data/install-openclaw-on-termux.sh/install-openclaw-termux.sh`

### 修改内容

#### 1. check_deps 函数（约第 193 行）

添加编译工具依赖：

```
DEPS=("nodejs-lts" "git" "openssh" "tmux" "termux-api" "termux-tools" "cmake" "python" "golang" "which" "clang" "ninja" "pkg-config")
```

#### 2. configure_npm 函数（约第 298 行）

在 npm install 命令前添加 GYP 配置：

```
# 创建 GYP 配置文件，解决原生模块编译问题
mkdir -p "$HOME/.gyp"
cat > "$HOME/.gyp/include.gypi" << 'EOF'
{
  'variables': {
    'clang': 1,
    'make_global_settings': [
      ['CC', 'clang'],
      ['CXX', 'clang++'],
      ['AR', 'llvm-ar'],
      ['NM', 'llvm-nm'],
      ['READLN', 'llvm-readelf']
    ]
  }
}
EOF

# 设置编译环境变量
export TMPDIR="$HOME/tmp"
export TMP="$HOME/tmp"
export TEMP="$HOME/tmp"
```

#### 3. 移除 --ignore-scripts（第 362、378、402 行）

修改前：

```
run_cmd env NODE_LLAMA_CPP_SKIP_DOWNLOAD=true npm i -g openclaw@$TARGET_VERSION --ignore-scripts
```

修改后：

```
run_cmd env NODE_LLAMA_CPP_SKIP_DOWNLOAD=true npm i -g openclaw@$TARGET_VERSION
```

### 目录结构

```
/mnt/d/data/install-openclaw-on-termux.sh/
└── install-openclaw-termux.sh  # [MODIFY] 添加 GYP 配置，移除 --ignore-scripts
```