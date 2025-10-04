# PyLighter - Lighter Protocol 智能交易机器人

专为 Lighter Protocol 去中心化交易所开发的 Python 智能交易机器人，集成网格交易和动态波动率收割策略，实现自动化高频交易。

## 快速开始

### 安装依赖
```bash
uv sync
```

### 环境配置
创建 `.env` 文件：
```bash
LIGHTER_KEY=0x... # 你的钱包地址  
LIGHTER_SECRET=... # 你的 API KEY
API_KEY_INDEX=1   # API KEY 索引，需要和 API KEY 匹配（可选，默认为1）
```

### 策略快速启动

#### 网格交易策略 (Grid Strategy)
```bash
# 模拟测试（推荐先运行）
uv run grid_strategy.py --dry-run --symbol TON

# 实盘交易
uv run grid_strategy.py --symbol ETH --max-orders 10 --order-amount 50.0 --grid-spacing 0.002 

# 高性能配置
uv run grid_strategy.py --symbol TON --dry-run --max-orders 20 --order-amount 60.0 --grid-spacing 0.0005
```

#### 动态波动率收割策略 (Reaper Strategy)
```bash
# 模拟测试
uv run reaper_strategy.py --dry-run --symbol BTC --enable-adaptive --performance-db data/reaper_performance.db

# 实盘交易（推荐配置）
uv run reaper_strategy.py --symbol BTC --enable-adaptive --total-notional 100 --trend-exit-mode cancel

# 高波动市场配置
uv run reaper_strategy.py --symbol ETH --enable-adaptive --atr-multiplier 0.4 --grid-layers 4 --adaptive-high-vol 0.02
```

### 命令行参数

#### 网格策略参数
```bash
# 查看所有参数
uv run grid_strategy.py --help

# 主要参数说明
--dry-run                   # 模拟模式，无真实交易
--symbol SYMBOL            # 交易符号 (默认: TON)
--max-orders N             # 单边最大订单数 (默认: 20)
--grid-spacing FLOAT       # 网格间距百分比 (默认: 0.0005 = 0.05%)
--order-amount AMOUNT      # 每单金额USD (默认: $60.0)
--price-threshold FLOAT    # 价格变动阈值 (默认: 0.0002 = 0.02%)
```

#### Reaper策略参数
```bash
# 查看所有参数
uv run reaper_strategy.py --help

# 核心参数说明
--symbol SYMBOL            # 交易符号 (默认: BTC)
--dry-run                  # 模拟模式，无真实交易
--indicator-period N       # ADX/ATR指标周期 (默认: 14)
--adx-threshold FLOAT      # 趋势判断阈值 (默认: 20.0)
--atr-multiplier FLOAT     # 网格间距倍数 (默认: 0.3)
--grid-layers N            # 网格层数 (默认: 3)
--total-notional AMOUNT    # 总资金分配 (默认: $1000)
--enable-adaptive          # 启用自适应模式 (默认: 开启)
--trend-exit-mode MODE     # 趋势退出模式: pause/cancel/flatten (默认: cancel)
--performance-db PATH      # 性能数据库路径
```

#### 🆕 价格阈值优化
**智能订单更新控制**，避免频繁无效交易：

```bash
# 高敏感度 (频繁调整，适合高波动)
uv run grid_strategy.py --dry-run --price-threshold 0.00001  # 0.001%

# 中等敏感度 (平衡模式，推荐)
uv run grid_strategy.py --dry-run --price-threshold 0.0005   # 0.05%

# 低敏感度 (稳定运行，适合低波动)
uv run grid_strategy.py --dry-run --price-threshold 0.002    # 0.2%
```

**价格阈值功能**：
- ✅ **减少API调用**：只有价格变动超过阈值才更新订单
- ✅ **提高稳定性**：避免微小波动导致的频繁调整
- ✅ **节省资源**：降低系统负载和网络消耗
- ✅ **保持响应**：价格真正变动时仍能快速响应

## 项目结构

```
pylighter/
├── pylighter/                    # 核心 SDK
│   ├── client.py                 # 主要客户端类
│   ├── httpx.py                  # HTTP 客户端
│   ├── websocket_manager.py      # WebSocket管理器
│   ├── order_manager.py          # 订单管理器
│   └── market_utils.py           # 市场工具
├── examples/                     # 示例代码
├── docs/                         # 文档
├── strategies/                   # 策略框架
│   ├── base_strategy.py          # 基础策略类
│   ├── mock_client.py            # 模拟客户端
│   ├── price_simulator.py        # 价格模拟器
│   └── backtest_runner.py        # 回测框架
├── utils/                        # 工具模块
│   └── logger_config.py          # 日志配置
├── grid_strategy.py              # 简化网格策略
├── reaper_strategy.py            # 动态波动率收割策略
├── reaper.md                     # Reaper策略详细文档
└── main.py                       # 主入口
```

## 核心功能

### 🤖 网格交易策略 (Grid Strategy)

**简化版高频网格交易机器人**，基于 pylighter SDK 工具重构：

#### 核心特性
- ✅ **简化代码架构**: 使用 pylighter SDK 工具，代码简洁但功能完整
- ✅ **智能订单管理**: 集成 BatchOrderManager 和 OrderSyncManager
- ✅ **价格阈值优化**: 0.02% 价格变动阈值，减少无效交易
- ✅ **动态持仓阈值**: 基于账户总价值的50%设定持仓限制
- ✅ **双向持仓管理**: 支持同时做多/做空，独立管理
- ✅ **风险控制**: 10倍杠杆，单边最大20订单，自动清理
- ✅ **官方API集成**: 使用官方账户API获取实时持仓和统计信息

#### 策略参数
- **网格间距**: 0.05% (可配置)
- **每单金额**: $60 USD (可配置)
- **最大杠杆**: 10x
- **持仓阈值**: 账户总价值的50%
- **订单刷新间隔**: 20秒
- **价格更新阈值**: 0.02%

#### 智能特性
- **装死模式**: 持仓超过阈值时只下止盈单
- **动态止盈**: 根据对冲比例动态调整止盈价格
- **库存风险控制**: 双向持仓过大时自动市价平仓
- **官方统计集成**: 实时显示账户总资产、保证金、盈亏等

### 🌾 动态波动率收割策略 (Reaper Strategy)

**高级波动率自适应网格策略**，专为零费用环境优化：

#### 核心特性
- ✅ **市场状态过滤**: 使用ADX指标区分震荡与趋势市场
- ✅ **动态网格间距**: 基于ATR波动率自动调整网格参数
- ✅ **自适应参数**: 根据市场波动率智能调节网格层数和间距
- ✅ **全局止损保护**: 设置主止损价格，防止黑天鹅事件
- ✅ **趋势退出模式**: 多种趋势处理方式 (pause/cancel/flatten)
- ✅ **性能追踪**: SQLite数据库记录完整的性能指标
- ✅ **风险管理**: 最大持仓限制和自动减仓机制

#### 策略原理
1. **市场状态识别**: ADX < 20 为震荡市，启动网格交易
2. **动态参数调整**: 基于ATR比率调整网格间距和层数
3. **对称网格部署**: 在当前价格上下对称布置买卖订单
4. **智能订单管理**: 低买高卖，持续收割波动利润
5. **风险控制**: 趋势来临时自动暂停或平仓

#### 自适应机制
- **低波动**: 缩小网格间距，增加交易频率
- **高波动**: 扩大网格间距，减少交易风险
- **参数范围**: 
  - ATR倍数: 0.25-0.5
  - 网格层数: 2-4层
  - ADX阈值: 20-30

#### 性能监控
- **实时权益**: 总资产价值追踪
- **盈亏统计**: 已实现/未实现盈亏分离
- **ROI计算**: 基于基准权益的收益率
- **历史记录**: 支持程序重启后继续累计

### 📚 基础 SDK 功能

```python
from pylighter.client import Lighter

# 初始化客户端
lighter = Lighter(key="your_key", secret="your_secret")
await lighter.init_client()

# 下限价单
tx_info, tx_hash, error = await lighter.limit_order(
    ticker="SUI",
    amount=3.0,  # 正数=买入/做多，负数=卖出/做空
    price=4.25,
    tif='GTC'
)

# 获取账户信息
account_info = await lighter.get_account_info()

# 查看持仓
positions = await lighter.get_positions()

# 取消所有订单
await lighter.cancel_all_orders()
```

## 使用指南

### 🚀 策略启动流程

1. **环境准备**
```bash
# 克隆项目
git clone <repository_url>
cd pylighter

# 安装依赖
uv sync

# 配置环境变量
echo "LIGHTER_KEY=0x..." > .env
echo "LIGHTER_SECRET=..." >> .env
echo "API_KEY_INDEX=1" >> .env
```

2. **策略测试**

#### 网格策略测试
```bash
# 模拟模式测试 (无风险)
uv run grid_strategy.py --dry-run --symbol TON

# 自定义网格参数测试
uv run grid_strategy.py --dry-run --symbol TON \
    --max-orders 20 \
    --order-amount 60.0 \
    --grid-spacing 0.0005 \
    --price-threshold 0.0002

# 高频策略测试
uv run grid_strategy.py --dry-run --symbol TON \
    --max-orders 25 \
    --order-amount 30.0 \
    --grid-spacing 0.0003 \
    --price-threshold 0.0001

# 检查日志
tail -f log/grid_strategy.log
```

#### Reaper策略测试
```bash
# 基础模拟测试
uv run reaper_strategy.py --dry-run --symbol BTC --enable-adaptive

# 自定义参数测试
uv run reaper_strategy.py --dry-run --symbol BTC \
    --total-notional 2000 \
    --atr-multiplier 0.4 \
    --grid-layers 4 \
    --adx-threshold 18

# 高波动市场配置
uv run reaper_strategy.py --dry-run --symbol ETH \
    --enable-adaptive \
    --adaptive-high-vol 0.02 \
    --adaptive-multiplier-max 0.6 \
    --trend-exit-mode flatten

# 检查性能数据库
sqlite3 data/reaper_performance.db "SELECT * FROM performance_snapshots ORDER BY timestamp DESC LIMIT 10;"
```

3. **实盘部署**

#### 网格策略实盘
```bash
# 启动实盘交易 (需要输入 YES 确认)
uv run grid_strategy.py --symbol TON

# 保守实盘策略 (推荐新手)
uv run grid_strategy.py --symbol TON \
    --max-orders 15 \
    --order-amount 30.0 \
    --price-threshold 0.0003

# 中等风险策略
uv run grid_strategy.py --symbol TON \
    --max-orders 20 \
    --order-amount 60.0 \
    --grid-spacing 0.0005 \
    --price-threshold 0.0002

# 高频策略 (需要充足资金)
uv run grid_strategy.py --symbol SUI \
    --max-orders 25 \
    --order-amount 40.0 \
    --grid-spacing 0.0003 \
    --price-threshold 0.0001
```

#### Reaper策略实盘
```bash
# 基础实盘配置
uv run reaper_strategy.py --symbol BTC --enable-adaptive --total-notional 1000

# 保守配置 (低波动市场)
uv run reaper_strategy.py --symbol BTC \
    --enable-adaptive \
    --total-notional 1000 \
    --atr-multiplier 0.25 \
    --grid-layers 2 \
    --adaptive-low-vol 0.002 \
    --trend-exit-mode cancel

# 积极配置 (高波动市场)
uv run reaper_strategy.py --symbol ETH \
    --enable-adaptive \
    --total-notional 2000 \
    --atr-multiplier 0.5 \
    --grid-layers 4 \
    --adaptive-high-vol 0.015 \
    --trend-exit-mode flatten

# 自定义性能追踪
uv run reaper_strategy.py --symbol BTC \
    --enable-adaptive \
    --performance-db data/btc_performance.db \
    --performance-interval 60
```

# 优雅停止 (Ctrl+C)
# 自动取消订单并保留持仓 (网格策略)
# 自动平仓并清理 (Reaper策略)

### 📊 监控和管理

#### 网格策略监控
```bash
# 实时监控日志
tail -f log/grid_strategy.log

# 查看策略运行状态
grep "📋 Orders" log/grid_strategy.log | tail -10

# 检查错误和警告
grep -E "(ERROR|WARNING)" log/grid_strategy.log | tail -5

# 查看账户统计信息
grep "账户统计信息" log/grid_strategy.log | tail -5

# 监控持仓变化
grep "持仓更新" log/grid_strategy.log | tail -10
```

#### Reaper策略监控
```bash
# 实时监控日志
tail -f log/reaper_strategy.log

# 查看策略状态切换
grep "Regime transition" log/reaper_strategy.log | tail -10

# 监控指标更新
grep "Indicator update" log/reaper_strategy.log | tail -5

# 查看性能快照
grep "Performance snapshot" log/reaper_strategy.log | tail -5

# 检查数据库记录
sqlite3 data/reaper_performance.db "SELECT timestamp, equity, roi FROM performance_snapshots ORDER BY timestamp DESC LIMIT 20;"
```

## ⚠️ 重要提醒

### 🔐 安全风险
- **真实资金交易**: 请先小额测试，熟悉策略后再增加资金
- **私钥安全**: 妥善保管私钥，使用 `.env` 文件，不要提交到代码库
- **网络风险**: 确保网络连接稳定，避免在不稳定网络环境下运行

### 📋 交易风险
- **市场风险**: 网格策略适合震荡行情，单边行情可能导致亏损
- **杠杆风险**: 5x 杠杆会放大收益和损失，请谨慎使用
- **技术风险**: 程序故障可能导致意外损失，建议监控运行状态

### 🛠️ 技术要求
- **Python 版本**: 需要 Python ≥3.13
- **依赖管理**: 使用 `uv` 包管理器
- **API 访问**: 需要有效的 Lighter Protocol 账户和 API 密钥

## 🔧 故障排除

### 网格策略常见问题

**Q: 如何调整价格阈值优化策略表现？**
```bash
# 高波动市场 - 使用较小阈值，更频繁调整
uv run grid_strategy.py --dry-run --symbol TON --price-threshold 0.00005  # 0.005%

# 低波动市场 - 使用较大阈值，减少无效调整
uv run grid_strategy.py --dry-run --symbol TON --price-threshold 0.001     # 0.1%

# 查看价格阈值触发日志
grep "💡 价格变动超过阈值" log/grid_strategy.log | tail -10
```

**Q: 如何调整订单数量控制风险？**
```bash
# 保守策略 - 少量订单
uv run grid_strategy.py --symbol TON --max-orders 15

# 积极策略 - 更多订单（需要充足资金）
uv run grid_strategy.py --symbol TON --max-orders 50

# 查看当前订单状态
grep "Active orders:" log/grid_strategy.log | tail -5
```

**Q: 订单达到限制无法下单？**
```bash
# 检查当前订单计数
grep "Max.*orders reached" log/grid_strategy.log | tail -5

# 程序会自动在5分钟内同步并恢复下单
# 或手动调整限制参数重启
uv run grid_strategy.py --symbol TON --max-orders 100
```

### Reaper策略常见问题

**Q: 如何优化ADX阈值参数？**
```bash
# 更早的趋势检测 (更保守)
uv run reaper_strategy.py --dry-run --symbol BTC --adx-threshold 18

# 更晚的趋势检测 (更积极)
uv run reaper_strategy.py --dry-run --symbol BTC --adx-threshold 25

# 查看市场状态切换
grep "Regime transition" log/reaper_strategy.log | tail -10
```

**Q: ATR倍数如何影响网格间距？**
```bash
# 紧密网格 (高频交易)
uv run reaper_strategy.py --dry-run --symbol BTC --atr-multiplier 0.2

# 宽松网格 (低频大利润)
uv run reaper_strategy.py --dry-run --symbol BTC --atr-multiplier 0.6

# 查看网格间距调整
grep "spacing=" log/reaper_strategy.log | tail -5
```

**Q: 趋势退出模式如何选择？**
```bash
# 保持现有订单 (适合震荡行情)
uv run reaper_strategy.py --symbol BTC --trend-exit-mode pause

# 取消所有订单 (推荐配置)
uv run reaper_strategy.py --symbol BTC --trend-exit-mode cancel

# 取消订单并平仓 (最保守)
uv run reaper_strategy.py --symbol BTC --trend-exit-mode flatten
```

**Q: 自适应模式参数如何调整？**
```bash
# 低波动环境配置
uv run reaper_strategy.py --symbol BTC \
    --adaptive-low-vol 0.001 \
    --adaptive-high-vol 0.01 \
    --adaptive-layers-max 5

# 高波动环境配置
uv run reaper_strategy.py --symbol BTC \
    --adaptive-low-vol 0.003 \
    --adaptive-high-vol 0.02 \
    --adaptive-layers-min 2
```

### 通用技术问题

**Q: WebSocket 连接频繁断开**
```bash
# 检查网络连接稳定性
ping mainnet.zklighter.elliot.ai

# 查看 WebSocket 重连日志
grep "Retrying WebSocket" log/grid_strategy.log
```

**Q: 订单无法成交**
```bash
# 检查市场流动性和价格设置
grep "Order placed" log/grid_strategy.log | tail -5

# 查看订单同步状态
grep "📋 Orders" log/grid_strategy.log | tail -10
```

**Q: 程序意外退出**
```bash
# 查看错误日志
grep "ERROR" log/grid_strategy.log | tail -10

# 检查 API 密钥配置
cat .env
```

## 📊 技术架构

### 核心组件
- **`pylighter/client.py`**: 主要 API 客户端，封装 Lighter Protocol REST API
- **`pylighter/httpx.py`**: HTTP 客户端，处理网络请求和错误重试
- **`pylighter/websocket_manager.py`**: WebSocket 价格流管理器
- **`pylighter/order_manager.py`**: 订单同步和批量管理器
- **`pylighter/market_utils.py`**: 市场数据管理和约束处理
- **`grid_strategy.py`**: 简化网格交易策略核心实现
- **`reaper_strategy.py`**: 动态波动率收割策略核心实现

### 设计特点
- **异步架构**: 全异步设计，支持高并发 WebSocket 和 API 调用
- **错误恢复**: 自动重连和错误重试机制
- **状态管理**: 精确的订单和持仓状态跟踪
- **日志系统**: 详细的运行日志，便于监控和调试
- **模块化设计**: 策略与SDK工具分离，便于扩展和维护
- **性能追踪**: SQLite数据库支持完整的性能历史记录

## 📖 更多资源

### 参考文档
- [Lighter Protocol 官方文档](https://docs.lighter.xyz/)
- [项目内文档](docs/) - API 参考和策略指南
- [示例代码](examples/) - 实用示例和测试脚本

### 社区支持
- 查看 [issues](https://github.com/your-repo/issues) 获取帮助
- 参考 [CLAUDE.md](CLAUDE.md) 了解开发信息

---

**免责声明**: 本工具仅供学习和研究使用，请自行承担交易风险。开发者不对使用本工具造成的任何损失负责。
