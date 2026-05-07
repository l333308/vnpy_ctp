# Mac 系统安装 vnpy_ctp 指南

## 前提条件

- macOS 系统（Intel 或 Apple Silicon 均支持）
- Python 3.10 ~ 3.13
- XCode Command Line Tools（提供 C++ 编译器）

安装 XCode 命令行工具：

```bash
xcode-select --install
```

验证 Python 版本：

```bash
python3 --version
```

## 安装步骤

### 1. 克隆项目

```bash
git clone https://github.com/vnpy/vnpy_ctp.git
cd vnpy_ctp
```

> **注意**：不要将源码直接放在用户主目录下，放在一个子文件夹中（如 `~/projects/vnpy_ctp`）。

### 2. 创建虚拟环境

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
```

### 3. 安装构建依赖

```bash
pip install "meson-python>=0.17.1" "meson>=1.7.0" "pybind11>=2.13.6" ninja
```

### 4. 安装 vnpy 核心框架

```bash
pip install vnpy
```

### 5. 编译安装 vnpy_ctp（开发模式）

```bash
pip install -e . --no-build-isolation
```

如果遇到编译报错 "Tried to form an absolute path to a dir in the source tree"，说明 `meson.build` 中 pybind11 include 路径需要调整（详见下方"已知问题"）。

### 6. 信任 macOS 动态库

macOS 安全机制会阻止加载未信任的动态库。执行以下命令移除 quarantine 属性：

```bash
xattr -rd com.apple.quarantine vnpy_ctp/api/thostmduserapi_se.framework
xattr -rd com.apple.quarantine vnpy_ctp/api/thosttraderapi_se.framework
```

或者在 Finder 中找到以下两个文件，双击打开一次以添加到系统信任名单：

- `vnpy_ctp/api/thostmduserapi_se.framework/Versions/A/thostmduserapi_se`
- `vnpy_ctp/api/thosttraderapi_se.framework/Versions/A/thosttraderapi_se`

### 7. 验证安装

```bash
python -c "from vnpy_ctp import CtpGateway; print('OK:', CtpGateway.default_name)"
```

输出 `OK: CTP` 表示安装成功。

### 8. 运行

```bash
python script/run.py
```

## 已知问题与注意事项

### 源码目录不能移动

编译时 framework 的绝对路径被写入 rpath，移动源码目录后会导致动态库加载失败。如需迁移，请重新执行 `pip install -e . --no-build-isolation`。

### Mac 版 CTP API 功能差异

Mac framework 基于 CTP 6.7.7 版本，相比标准版（6.7.11）缺少以下边缘功能，**不影响核心交易**：

| 缺失接口 | 功能 | 影响 |
|---------|------|------|
| ReqQryUserSession | 查询用户会话 | 无，vnpy 未使用 |
| RegisterWechatUserSystemInfo | 微信终端上报 | 无，仅微信接入场景 |
| ReqQryCombLeg | 查询组合腿 | 辅助查询，非核心 |
| ReqOffsetSetting | 对冲设置 | 券商管理功能 |

行情订阅、报单、撤单、持仓/账户查询等核心功能完全正常。

### meson.build 修改说明

本项目 `meson.build` 已针对 Mac 做了以下适配：

1. pybind11 include 路径通过 `-I` 编译参数传递（避免 venv 在源码目录内导致的 meson 路径冲突）
2. 使用 `-F` + `-framework` 链接 CTP 动态库（替代 Linux 的 `-l` 方式）
3. rpath 设置为 framework 所在的源码目录绝对路径

### C++ 源码条件编译

`vnctpmd.cpp`、`vnctptd.cpp`、`vnctptd.h` 中使用 `#ifndef __APPLE__` 屏蔽了 Mac framework 不支持的 API 调用和类型引用。

## 快速一键安装（汇总）

```bash
git clone https://github.com/vnpy/vnpy_ctp.git
cd vnpy_ctp
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install "meson-python>=0.17.1" "meson>=1.7.0" "pybind11>=2.13.6" ninja vnpy
pip install -e . --no-build-isolation
xattr -rd com.apple.quarantine vnpy_ctp/api/thostmduserapi_se.framework
xattr -rd com.apple.quarantine vnpy_ctp/api/thosttraderapi_se.framework
python -c "from vnpy_ctp import CtpGateway; print('OK:', CtpGateway.default_name)"
```
