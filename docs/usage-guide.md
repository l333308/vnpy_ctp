# VeighNa (vnpy) CTP 完整使用指南

## 一、界面为什么是空的？

`script/run.py` 只添加了 `CtpGateway`（交易接口），没有通过 `add_app()` 添加任何**应用模块**。

VeighNa Trader 左侧工具栏的每个图标都对应一个独立的应用模块（pip 包），需要：

1. `pip install` 安装对应的包
2. 在 `run.py` 中 import 并调用 `main_engine.add_app()`

## 二、可用的应用模块一览

| 模块名 | pip 包名 | 功能说明 |
|--------|---------|---------|
| CTA策略自动交易 | `vnpy_ctastrategy` | CTA策略的开发、回测、实盘自动交易 |
| CTA回测研究 | `vnpy_ctabacktester` | 图形化回测界面、参数优化 |
| 价差套利 | `vnpy_spreadtrading` | 多合约价差套利交易 |
| 期权交易 | `vnpy_optionmaster` | 期权波动率交易、Greeks风控 |
| 组合策略 | `vnpy_portfoliostrategy` | 多合约组合策略 |
| 算法交易 | `vnpy_algotrading` | TWAP/Iceberg/Sniper等算法委托 |
| 脚本策略 | `vnpy_scripttrader` | 命令行/Jupyter REPL式交易 |
| 仿真交易 | `vnpy_paperaccount` | 本地仿真撮合 |
| 行情记录 | `vnpy_datarecorder` | 实盘行情实时录制 |
| 数据管理 | `vnpy_datamanager` | 历史数据导入/导出/查看/删除 |
| 风险管理 | `vnpy_riskmanager` | 事前风控（流控、下单限制等） |
| K线图表 | `vnpy_chartwizard` | 实时K线图表 |
| 投资组合 | `vnpy_portfoliomanager` | 投资组合管理 |
| RPC服务 | `vnpy_rpcservice` | 分布式架构服务 |
| Excel RTD | `vnpy_excelrtd` | Excel实时数据接口 |
| Web交易 | `vnpy_webtrader` | Web服务器模块 |

## 三、添加应用模块（解决界面空白问题）

### Step 1: 安装需要的模块

```bash
# 激活项目的虚拟环境
source .venv/bin/activate

# 安装数据库驱动（必须，vnpy 4.x 不再内置）
pip install vnpy_sqlite

# 安装常用模块
pip install vnpy_ctastrategy vnpy_ctabacktester vnpy_datamanager \
    vnpy_chartwizard vnpy_riskmanager vnpy_algotrading \
    vnpy_spreadtrading vnpy_paperaccount vnpy_datarecorder
```

### Step 2: 修改 `script/run.py`

```python
from vnpy.event import EventEngine
from vnpy.trader.engine import MainEngine
from vnpy.trader.ui import MainWindow, create_qapp

from vnpy_ctp import CtpGateway

# 导入应用模块
from vnpy_ctastrategy import CtaStrategyApp
from vnpy_ctabacktester import CtaBacktesterApp
from vnpy_datamanager import DataManagerApp
from vnpy_chartwizard import ChartWizardApp
from vnpy_riskmanager import RiskManagerApp
from vnpy_algotrading import AlgoTradingApp
from vnpy_spreadtrading import SpreadTradingApp
from vnpy_paperaccount import PaperAccountApp
from vnpy_datarecorder import DataRecorderApp


def main():
    qapp = create_qapp()

    event_engine = EventEngine()
    main_engine = MainEngine(event_engine)

    # 添加交易接口
    main_engine.add_gateway(CtpGateway)

    # 添加应用模块（每个对应左侧一个图标）
    main_engine.add_app(CtaStrategyApp)
    main_engine.add_app(CtaBacktesterApp)
    main_engine.add_app(DataManagerApp)
    main_engine.add_app(ChartWizardApp)
    main_engine.add_app(RiskManagerApp)
    main_engine.add_app(AlgoTradingApp)
    main_engine.add_app(SpreadTradingApp)
    main_engine.add_app(PaperAccountApp)
    main_engine.add_app(DataRecorderApp)

    main_window = MainWindow(main_engine, event_engine)
    main_window.showMaximized()

    qapp.exec()


if __name__ == "__main__":
    main()
```

### Step 3: 重新启动

```bash
python script/run.py
```

启动后左侧工具栏应出现对应图标，菜单栏 **【功能】** 下也能看到已加载的模块。

## 四、连接 SimNow 仿真账号（CTP）

SimNow 是期货交易所提供的仿真交易平台，注册地址：<https://www.simnow.com.cn/>

> 注册后必须修改一次密码才能登录 CTP。

### 图形界面配置

启动后点击菜单栏 **【系统】→【连接CTP】**，填写：

| 参数 | 值 | 说明 |
|------|-----|------|
| 用户名 | 6位数字 | SimNow 的 InvestorID（不是注册手机号） |
| 密码 | 修改后的密码 | 注册后必须改一次密码 |
| 经纪商代码 | `9999` | SimNow 默认 |
| 交易服务器 | `180.168.146.187:10101` | 盘中测试环境 |
| 行情服务器 | `180.168.146.187:10111` | 盘中行情环境 |
| 产品名称 | `simnow_client_test` | 固定值 |
| 授权编码 | `0000000000000000` | 16个0 |

### JSON 配置文件方式

手动创建 `~/.vntrader/connect_ctp.json`：

```json
{
    "用户名": "000000",
    "密码": "xxxxxx",
    "经纪商代码": "9999",
    "交易服务器": "180.168.146.187:10101",
    "行情服务器": "180.168.146.187:10111",
    "产品名称": "simnow_client_test",
    "授权编码": "0000000000000000"
}
```

### 连接验证

连接成功后，主界面底部 **【日志】** 组件会输出登录信息，同时显示账号资金、持仓、合约等数据。

## 五、数据库配置

VeighNa 默认使用 **SQLite**（零配置），数据存在 `~/.vntrader/database.db`。

如需更换数据库，在菜单栏 **【配置】** 中修改对应字段，保存后重启生效。

### SQLite（默认，推荐新手）

| 字段 | 值 |
|------|-----|
| `database.name` | `sqlite` |
| `database.database` | `database.db` |

### MySQL

需先手动创建数据库：`CREATE DATABASE vnpy;`

| 字段 | 值 |
|------|-----|
| `database.name` | `mysql`（注意小写） |
| `database.host` | `localhost` |
| `database.port` | `3306` |
| `database.database` | `vnpy` |
| `database.user` | `root` |
| `database.password` | 你的密码 |

### PostgreSQL

| 字段 | 值 |
|------|-----|
| `database.name` | `postgresql` |
| `database.host` | `localhost` |
| `database.port` | `5432` |
| `database.database` | `vnpy` |
| `database.user` | `postgres` |
| `database.password` | 你的密码 |

### 其他支持

MongoDB、InfluxDB、DolphinDB、Arctic、LevelDB 等，详见 [官方数据库文档](https://www.vnpy.com/docs/cn/community/info/database.html)。

## 六、获取历史数据

### 方式A: 通过 CTA回测模块下载（推荐）

1. 菜单栏 **【功能】→【CTA回测】**
2. 在下载区域填写：
   - 本地代码：如 `IF888.CFFEX`、`rb2505.SHFE`
   - K线周期：`1m`（1分钟）、`1h`（1小时）、`d`（日线）、`w`（周线）、`tick`
   - 开始/结束日期
3. 点击 **【下载数据】**，数据自动存入本地数据库

### 方式B: 配置第三方数据服务（Datafeed）

在 **【配置】** 中设置 `datafeed` 相关字段：

| 字段 | 说明 |
|------|------|
| `datafeed.name` | 数据服务名（全小写英文） |
| `datafeed.username` | 用户名/license |
| `datafeed.password` | 密码/token |

支持的数据服务：

| 服务 | 数据类型 | 特点 |
|-----|---------|------|
| RQData（米筐） | 股票/期货/期权/基金 | 日线/小时线/分钟线/TICK，实时更新 |
| 迅投研 | 股票/期货/期权 | 性价比高，实时更新 |
| TuShare | 股票/期货 | 开源社区，盘后更新 |
| TQSDK（天勤） | 期货 | 免费分钟线，实时更新 |
| Wind | 期货 | 机构标配 |
| iFinD（同花顺） | 期货 | 机构用户 |
| UData（恒有数） | 股票/期货 | 不限次不限量，盘后更新 |

详见 [官方数据服务文档](https://www.vnpy.com/docs/cn/community/info/datafeed.html)。

### 方式C: 脚本方式下载

```python
from datetime import datetime
from vnpy.trader.constant import Exchange, Interval
from vnpy.trader.datafeed import get_datafeed
from vnpy.trader.object import HistoryRequest

datafeed = get_datafeed()

req = HistoryRequest(
    symbol="cu888",
    exchange=Exchange.SHFE,
    start=datetime(2023, 1, 1),
    end=datetime(2024, 1, 1),
    interval=Interval.DAILY
)

data = datafeed.query_bar_history(req)
```

### 方式D: 脚本直接操作数据库

```python
from datetime import datetime
from vnpy.trader.constant import Exchange, Interval
from vnpy.trader.database import get_database

database = get_database()

# 读取K线数据
bars = database.load_bar_data(
    symbol="cu888",
    exchange=Exchange.SHFE,
    interval=Interval.DAILY,
    start=datetime(2023, 1, 1),
    end=datetime(2024, 1, 1)
)

# 删除K线数据（谨慎操作）
database.delete_bar_data(
    symbol="cu888",
    exchange=Exchange.SHFE,
    interval=Interval.DAILY
)
```

## 七、CTA策略回测

### 图形化回测（CtaBacktester 模块）

1. 菜单栏 **【功能】→【CTA回测】**
2. 配置回测参数：
   - **策略品种**：选择策略名称、填写合约代码（如 `IF888.CFFEX`）
   - **数据范围**：回测起止日期
   - **交易成本**：滑点、百分比手续费、固定手续费
   - **合约属性**：合约乘数、价格跳动、回测资金、合约模式
3. 点击 **【开始回测】**
4. 查看结果：
   - 四张业绩图表：账户净值、净值回撤、每日盈亏、盈亏分布
   - 统计指标：夏普比率、收益回撤比、总收益率、最大回撤等
   - 详细信息：委托记录、成交记录、每日盈亏、K线图表

### 参数优化

1. 在回测界面设置优化参数的开始值、结束值、步进值
2. 选择优化目标函数（如总收益率、夏普比率）
3. 选择优化算法：
   - **穷举优化**：遍历所有参数组合
   - **遗传算法优化**：智能搜索最优参数
4. 点击 **【多进程优化】** 可利用多核 CPU 并行加速

## 八、CTA策略开发基础

策略文件放在运行时目录下的 `strategies/` 文件夹中：
- Windows: `C:\Users\<用户名>\strategies\`
- macOS/Linux: `~/.vntrader/strategies/`（或当前运行目录下的 `strategies/`）

### 核心组件

- **BarGenerator**: Tick 数据 → K线合成器，支持多周期
- **ArrayManager**: K线序列管理器，内置技术指标计算（SMA, RSI, CCI, Bollinger 等）

### 策略模板示例

```python
from vnpy_ctastrategy import CtaTemplate, BarGenerator, ArrayManager


class DoubleMaStrategy(CtaTemplate):
    """双均线策略"""
    author = "me"

    fast_window = 10
    slow_window = 20

    fast_ma = 0.0
    slow_ma = 0.0

    parameters = ["fast_window", "slow_window"]
    variables = ["fast_ma", "slow_ma"]

    def __init__(self, cta_engine, strategy_name, vt_symbol, setting):
        super().__init__(cta_engine, strategy_name, vt_symbol, setting)
        self.bg = BarGenerator(self.on_bar)
        self.am = ArrayManager()

    def on_init(self):
        """策略初始化"""
        self.write_log("策略初始化")
        self.load_bar(10)  # 加载10天历史数据用于初始化

    def on_start(self):
        """策略启动"""
        self.write_log("策略启动")

    def on_stop(self):
        """策略停止"""
        self.write_log("策略停止")

    def on_tick(self, tick):
        """Tick数据更新"""
        self.bg.update_tick(tick)

    def on_bar(self, bar):
        """K线数据更新"""
        self.cancel_all()

        self.am.update_bar(bar)
        if not self.am.inited:
            return

        self.fast_ma = self.am.sma(self.fast_window)
        self.slow_ma = self.am.sma(self.slow_window)

        if self.fast_ma > self.slow_ma:
            if self.pos == 0:
                self.buy(bar.close_price, 1)
            elif self.pos < 0:
                self.cover(bar.close_price, abs(self.pos))
                self.buy(bar.close_price, 1)
        elif self.fast_ma < self.slow_ma:
            if self.pos == 0:
                self.short(bar.close_price, 1)
            elif self.pos > 0:
                self.sell(bar.close_price, abs(self.pos))
                self.short(bar.close_price, 1)

        self.put_event()
```

### 多周期策略

```python
from vnpy.trader.constant import Interval

# 在 __init__ 中创建多个 BarGenerator
self.bg_5min = BarGenerator(self.on_bar, window=5, on_window_bar=self.on_5min_bar)
self.bg_1hour = BarGenerator(
    self.on_bar, window=1,
    on_window_bar=self.on_1hour_bar,
    interval=Interval.HOUR
)
```

### 策略关键方法速查

| 方法 | 说明 |
|------|------|
| `self.buy(price, volume)` | 买入开仓 |
| `self.sell(price, volume)` | 卖出平仓 |
| `self.short(price, volume)` | 卖出开仓 |
| `self.cover(price, volume)` | 买入平仓 |
| `self.cancel_all()` | 撤销所有活动委托 |
| `self.write_log(msg)` | 输出日志 |
| `self.load_bar(days)` | 加载历史K线用于初始化 |
| `self.put_event()` | 通知界面更新显示 |

### ArrayManager 常用指标

| 方法 | 说明 |
|------|------|
| `am.sma(n)` | 简单移动平均 |
| `am.ema(n)` | 指数移动平均 |
| `am.rsi(n)` | RSI相对强弱指标 |
| `am.cci(n)` | CCI顺势指标 |
| `am.atr(n)` | ATR真实波幅 |
| `am.boll(n, dev)` | 布林带（返回上轨、中轨、下轨） |
| `am.macd(fast, slow, signal)` | MACD指标 |
| `am.donchian(n)` | 唐奇安通道 |

## 九、常用工作流

```
安装模块 → 修改 run.py → 启动 VeighNa Trader
                              │
                    连接CTP（SimNow仿真）
                              │
                    订阅行情（输入合约代码+交易所）
                              │
            ┌─────────────────┼─────────────────┐
         下载数据          编写策略          手动交易
            │                │
         回测优化       加载策略实盘运行
```

## 十、常见问题

### Q: 左侧工具栏没有图标？

A: 需要 `pip install` 对应的应用模块，并在 `run.py` 中 `add_app()`。见第三节。

### Q: 连接CTP后日志没有任何输出？

A: 用 `telnet` 测试服务器端口是否可达：`telnet 180.168.146.187 10101`。SimNow 只在交易时段和盘后测试时段开放。

### Q: 如何订阅行情？

A: 在左上角交易组件中，交易所选择 `CFFEX`（中金所）、`SHFE`（上期所）等，代码填写合约代码（如 `IF2506`），按回车键订阅。

### Q: 回测时提示没有数据？

A: 需先通过CTA回测模块下载数据，或配置第三方数据服务。见第六节。

### Q: 策略文件放在哪里？

A: 放在运行目录下的 `strategies/` 文件夹中，VeighNa Trader 启动时会自动扫描加载。

## 参考文档

- [VeighNa 官方文档](https://www.vnpy.com/docs/cn/index.html)
- [VeighNa Trader 使用说明](https://www.vnpy.com/docs/cn/community/info/veighna_trader.html)
- [VeighNa Station](https://www.vnpy.com/docs/cn/community/info/veighna_station.html)
- [CTA策略模块](https://www.vnpy.com/docs/cn/community/app/cta_strategy.html)
- [CTA回测模块](https://www.vnpy.com/docs/cn/community/app/cta_backtester.html)
- [数据库配置](https://www.vnpy.com/docs/cn/community/info/database.html)
- [数据服务配置](https://www.vnpy.com/docs/cn/community/info/datafeed.html)
