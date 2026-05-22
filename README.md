# VeighNa框架的CTP底层接口

<p align="center">
  <img src ="https://vnpy.oss-cn-shanghai.aliyuncs.com/vnpy-logo.png"/>
</p>

<p align="center">
    <img src ="https://img.shields.io/badge/version-6.7.11.4-blueviolet.svg"/>
    <img src ="https://img.shields.io/badge/platform-windows|linux|macos-yellow.svg"/>
    <img src ="https://img.shields.io/badge/python-3.10|3.11|3.12|3.13-blue.svg" />
    <img src ="https://img.shields.io/github/license/vnpy/vnpy.svg?color=orange"/>
</p>

## 项目概述

`vnpy_ctp` 是 [VeighNa](https://www.vnpy.com)（vnpy）量化交易框架的 **CTP 期货接口网关包**，基于上期技术 CTP 期货版 **6.7.11** 官方 API 封装开发。接口中自带的是【穿透式实盘、评测环境合并】的动态库文件（`thostmduserapi_se` / `thosttraderapi_se`）。

项目通过 **pybind11** 将 CTP 官方 C++ API 封装为 Python 扩展模块（`MdApi` 行情接口、`TdApi` 交易接口），并实现了符合 VeighNa 框架规范的 `CtpGateway` 网关类，可直接接入 VeighNa 的事件引擎、策略模块和图形界面。

## 架构设计

```
VeighNa MainEngine（事件驱动引擎）
       │
       ▼
CtpGateway（ctp_gateway.py - VeighNa 网关实现）
       │
       ├── CtpMdApi（行情：连接、登录、订阅、Tick 推送）
       │       │
       │       ▼
       │   vnctpmd（pybind11 C++ 扩展 → CThostFtdcMdSpi）
       │       │
       │       ▼
       │   thostmduserapi_se（CTP 官方行情动态库）
       │
       └── CtpTdApi（交易：认证、登录、下单、撤单、查询）
               │
               ▼
           vnctptd（pybind11 C++ 扩展 → CThostFtdcTraderSpi）
               │
               ▼
           thosttraderapi_se（CTP 官方交易动态库）
```

**线程模型**：CTP 回调在 C++ 端写入 `TaskQueue`，由独立工作线程取出后通过 `PYBIND11_OVERLOAD` 调用 Python 重写方法，避免在回调线程直接操作 Python GIL。

## 技术栈

| 类别 | 技术 |
|------|------|
| 量化框架 | VeighNa (`vnpy>=3.0.0`) |
| 语言 | Python 3.10+、C++17（Mac 部分 C++11） |
| C++/Python 绑定 | pybind11 ≥2.13.6 |
| 构建系统 | Meson ≥1.7.0 + meson-python（PEP 517） |
| 代码质量 | Ruff（lint）、Mypy（类型检查） |
| 测试 | pytest（SimNow 集成测试） |
| CI | GitHub Actions（ruff + mypy + uv build） |
| 期货 API | 上期技术 CTP 6.7.11（穿透式 `_se` 版） |

## 项目结构

```
vnpy_ctp/
├── meson.build                    # Meson 构建配置（编译 C++ 扩展、安装动态库）
├── pyproject.toml                 # Python 包元数据、构建后端、工具链配置
├── README.md
├── CHANGELOG.md                   # 版本变更记录
├── LICENSE                        # MIT 许可证
│
├── vnpy_ctp/                      # 主 Python 包
│   ├── __init__.py                # 导出 CtpGateway
│   ├── api/                       # CTP 底层 API 封装层
│   │   ├── __init__.py            # 导出 MdApi, TdApi, ctp_constant
│   │   ├── ctp_constant.py        # CTP 常量定义（由生成器自动生成）
│   │   ├── vnctp/                 # pybind11 C++ 源代码
│   │   │   ├── vnctp.h            # 公共逻辑：TaskQueue、字典转换、GBK/UTF-8
│   │   │   ├── vnctpmd/           # 行情扩展模块（vnctpmd.cpp/.h）
│   │   │   └── vnctptd/          # 交易扩展模块（vnctptd.cpp/.h）
│   │   ├── generator/             # CTP 头文件 → C++/Python 绑定代码生成器
│   │   ├── include/               # CTP 官方 C++ 头文件
│   │   │   ├── ctp/               # Windows/Linux 头文件
│   │   │   └── mac/ctp/           # macOS 专用头文件
│   │   ├── libs/                  # Windows 静态链接库（.lib）
│   │   ├── *.dll / *.so           # 预编译 CTP 动态库（Windows/Linux）
│   │   └── *.framework/           # macOS Framework 形式 CTP 库
│   └── gateway/
│       └── ctp_gateway.py         # CtpGateway 网关实现（约 900 行）
│
├── script/
│   └── run.py                     # VeighNa Trader 启动脚本
├── test/
│   ├── test_md.py                 # 行情 API 测试（需 SimNow 账号）
│   └── test_td.py                 # 交易 API 测试（需 SimNow 账号）
├── docs/
│   ├── usage-guide.md             # 完整使用指南（SimNow 连接、策略开发等）
│   └── mac-install.md             # Mac 编译安装指南
└── .github/
    └── workflows/pythonapp.yml    # CI 工作流
```

## 安装

### 环境要求

- Python ≥ 3.10（支持 3.10、3.11、3.12、3.13）
- 推荐基于 4.0.0 版本以上的 [VeighNa Studio](https://www.vnpy.com)

### PyPI 安装（Windows/Linux）

```bash
pip install vnpy_ctp
```

### 源码安装

源码编译需要 C++ 编译器：**Visual Studio**（Windows）、**GCC**（Linux）、**Xcode**（Mac）。

```bash
git clone https://github.com/vnpy/vnpy_ctp.git
cd vnpy_ctp
pip install .
```

### 开发模式安装

```bash
pip install -e . --no-build-isolation --config-settings=build-dir=./vnpy_ctp/api
```

### macOS 注意事项

macOS 的 CTP 库采用 framework 目录结构，**无法从 PyPI 安装预编译 wheel**，必须本地源码编译。安装后需执行以下命令去除 quarantine 标记：

```bash
xattr -rd com.apple.quarantine /path/to/venv/lib/python3.x/site-packages/vnpy_ctp/
```

macOS 底层 CTP API 版本为 **6.7.7**，相对 6.7.11 缺少部分边缘接口（不影响核心交易功能）。详细说明见 [Mac 安装指南](docs/mac-install.md)。

## 使用

### 快速验证安装

```bash
python -c "from vnpy_ctp import CtpGateway; print('OK:', CtpGateway.default_name)"
# 期望输出: OK: CTP
```

### 脚本方式启动

```python
from vnpy.event import EventEngine
from vnpy.trader.engine import MainEngine
from vnpy.trader.ui import MainWindow, create_qapp

from vnpy_ctp import CtpGateway


def main():
    """主入口函数"""
    qapp = create_qapp()

    event_engine = EventEngine()
    main_engine = MainEngine(event_engine)
    main_engine.add_gateway(CtpGateway)

    main_window = MainWindow(main_engine, event_engine)
    main_window.showMaximized()

    qapp.exec()


if __name__ == "__main__":
    main()
```

### SimNow 仿真连接

连接参数配置文件位于 `~/.vntrader/connect_ctp.json`，SimNow 测试环境配置示例：

| 参数 | 值 |
|------|-----|
| 经纪商代码 | `9999` |
| 产品名称 | `simnow_client_test` |
| 授权编码 | `0000000000000000` |
| 交易服务器 | `180.168.146.187:10130` |
| 行情服务器 | `180.168.146.187:10131` |

更多使用说明参见 [使用指南](docs/usage-guide.md)。

## 测试

项目使用 pytest 进行集成测试，测试需要真实的 SimNow 账号和网络连接：

```bash
pip install pytest
pytest test/ -v
```

> 注意：测试文件中的 `UserID` / `Password` 需替换为自己的 SimNow 账号信息。

## CTP API 版本

| 平台 | API 版本 | 说明 |
|------|---------|------|
| Windows / Linux | 6.7.11 | 穿透式实盘 + 评测环境合并 |
| macOS | 6.7.7 | Framework 形式，部分边缘接口缺失 |

### 代码生成器

升级 CTP 版本时使用 `vnpy_ctp/api/generator/` 下的生成脚本：

1. 替换 `include/` 目录中的官方头文件和预编译库
2. 运行 `generate_data_type.py` → 生成 `ctp_constant.py`
3. 运行 `generate_struct.py` → 生成结构体定义
4. 运行 `generate_api_functions.py` → 生成 C++ 绑定代码
5. 重新编译安装

## 版本历史

完整变更记录见 [CHANGELOG.md](CHANGELOG.md)，主要里程碑：

- **6.7.11.x** — API 升级至 6.7.11，优化撤单状态映射、登录兼容性
- **6.7.7.x** — 适配 vnpy 4.0 框架，API 升级至 6.7.7
- **6.7.2.x** — API 升级至 6.7.2，类型声明使用内置类型
- **6.6.9.x** — API 升级至 6.6.9，支持大商所毫秒级时间戳
- **6.6.7.x** — 首次增加 macOS 支持，使用 zoneinfo 替换 pytz

## 许可证

MIT License — Copyright (c) 2015-present, Xiaoyou Chen

详见 [LICENSE](LICENSE) 文件。
