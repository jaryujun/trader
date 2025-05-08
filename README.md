# Trader - 多功能量化交易平台

## 项目介绍

Trader是一个高性能、可扩展的量化交易系统，采用Python和C++混合架构设计，充分发挥两种语言的优势。系统支持多市场、多策略并行执行，具备完整的回测和实盘交易功能。核心特性包括：

- **高性能交易执行**：C++实现的交易执行层，支持微秒级响应
- **灵活策略开发**：Python策略引擎，支持多种交易策略并行运行
- **完整回测系统**：基于历史数据的策略回测和性能评估
- **风险管理**：实时风控检查和资金管理
- **数据处理**：高效的市场数据获取、清洗和存储
- **可视化分析**：交易绩效分析和可视化报告

## 关键概念解释

### 量化交易核心概念

- **策略信号** - 由策略生成的交易指令，通常是买入、卖出或持有的决策
- **因子模型** - 使用多种因子（如价值、成长、动量等）来预测资产收益的模型
- **回测** - 在历史数据上模拟策略执行过程，评估其有效性和风险
- **滑点** - 预期价格与实际执行价格之间的差异，是交易成本的一部分
- **撮合引擎** - 负责在回测或模拟环境中匹配买卖订单的系统组件
- **标记价格** - 用于计算未实现盈亏的价格，特别是在期货和杠杆交易中
- **风险控制** - 监控和管理交易策略风险的过程，包括设置止损和资金分配
- **资金管理** - 决定每笔交易使用多少资金，通常基于风险和回报比率

### 性能指标术语

- **夏普比率** - 衡量风险调整后收益的指标，高值表示更好的风险收益平衡
- **最大回撤** - 从峰值到低谷的最大损失百分比，衡量策略风险
- **胜率** - 盈利交易占总交易的百分比
- **盈亏比** - 平均盈利交易与平均亏损交易的比率
- **阿尔法** - 相对于基准指数的超额收益
- **贝塔** - 策略相对于市场波动性的敏感度
- **索提诺比率** - 类似夏普比率，但只考虑下行风险
- **卡玛比率** - 年化收益率除以最大回撤，衡量单位风险的回报

## 支持的市场

TODO

## 安装与快速开始指南

### 系统要求

- Python 3.10+
- C++17兼容编译器 (GCC 8+, Visual Studio 2019+, Clang 7+)
- 64位操作系统 (Linux推荐，也支持Windows和macOS)
- 最小8GB RAM (建议16GB+用于生产环境)
- PostgreSQL 12+
- Redis 6+

### 基本安装

1. **克隆项目**

```bash
git clone https://github.com/jaryujun/trader.git
cd trader
```

2. **使用pip安装依赖**

```bash
pip install -r requirements/dev.txt
```

3. **创建配置文件**

```bash
cp config/config.example.yaml config/config.yaml
# 编辑配置文件以匹配您的环境
```

4. **设置数据库**

```bash
# 启动PostgreSQL和Redis
docker-compose -f docker/dev/docker-compose.yml up -d

# 初始化数据库
python scripts/setup/init_db.py
```

### Docker快速启动

如果您已安装Docker和Docker Compose，可以使用以下命令快速启动完整开发环境：

```bash
# 启动所有服务
docker-compose -f docker/dev/docker-compose.yml up -d

# 检查服务状态
docker-compose -f docker/dev/docker-compose.yml ps
```

### 运行示例策略

```bash
# 运行回测示例
python examples/strategies/simple_moving_average.py

# 查看回测结果
python -m src.analysis.reports.show_report --report-path ./output/reports/latest.html
```

## 系统架构

系统采用微服务架构，主要组件包括：

```
                ┌────────────────────┐
                │    API Gateway     │
                │    (FastAPI)       │
                └─────────▲──────────┘
                          │
           ┌──────────────┼──────────────┐
           │              │              │
┌──────────▼─────┐ ┌──────▼──────┐ ┌────▼──────────┐
│  策略引擎服务   │ │ 交易执行服务 │ │  数据服务     │
│  (Python)      │ │   (C++)     │ │   (Python)    │
└──────────▲─────┘ └──────▲──────┘ └────▲──────────┘
           │              │              │
           └──────────────┼──────────────┘
                          │
                ┌─────────▼──────────┐
                │  消息队列 (Kafka)   │
                └─────────▲──────────┘
                          │
                ┌─────────┴──────────┐
                │   数据存储层        │
                │ (PostgreSQL, Redis) │
                └────────────────────┘
```

## 配置说明

系统配置使用YAML格式，主要配置文件包括：

### 核心配置 (config/config.yaml)

```yaml
# 数据库连接
database:
  postgres:
    host: localhost
    port: 5432
    user: trader
    password: password
    database: trader_db
  
  redis:
    host: localhost
    port: 6379
    db: 0

# 系统参数
system:
  log_level: INFO
  timezone: Asia/Shanghai
  default_currency: CNY
  
# 数据源配置
data_sources:
  - name: binance
    type: ccxt
    api_key: your_api_key
    secret: your_secret_key
    testnet: true
    
  - name: tushare
    type: api
    token: your_token
```

### 策略配置 (config/strategies/strategy_name.yaml)

```yaml
# 策略基本信息
name: dual_moving_average
description: "双均线交叉策略"
version: "1.0.0"

# 策略参数
parameters:
  fast_period: 10
  slow_period: 20
  risk_ratio: 0.02
  
# 交易产品
symbols:
  - exchange: binance
    symbol: BTC/USDT
    timeframe: 1h
    
# 资金管理
capital:
  initial: 10000
  leverage: 1
```

## 策略开发指南

### 策略实现基本结构

所有策略应继承自`BaseStrategy`类，并实现以下主要方法：

```python
from src.strategy.templates.base_strategy import BaseStrategy
from src.models.data_types import Bar, OrderRequest, Position

class MyStrategy(BaseStrategy):
    """
    自定义策略示例
    """
    
    def __init__(self, config):
        """初始化策略参数"""
        super().__init__(config)
        # 从配置中获取参数
        self.fast_period = self.config.get('parameters.fast_period', 10)
        self.slow_period = self.config.get('parameters.slow_period', 20)
        
    async def on_init(self, context):
        """策略初始化时调用"""
        # 创建技术指标
        self.fast_ma = self.create_indicator("MA", period=self.fast_period)
        self.slow_ma = self.create_indicator("MA", period=self.slow_period)
        
    async def on_bar(self, bar: Bar, context):
        """每个新K线到来时调用"""
        # 更新指标
        fast_value = self.fast_ma.update(bar.close)
        slow_value = self.slow_ma.update(bar.close)
        
        position = self.get_position(bar.symbol)
        
        # 交易逻辑：快线上穿慢线买入，快线下穿慢线卖出
        if fast_value > slow_value and fast_value <= slow_value:
            # 没有持仓时买入
            if position is None or position.amount == 0:
                self.buy(bar.symbol, bar.close, 1.0)
                
        elif fast_value < slow_value and fast_value >= slow_value:
            # 有持仓时卖出
            if position is not None and position.amount > 0:
                self.sell(bar.symbol, bar.close, position.amount)
    
    async def on_order_update(self, order, context):
        """订单状态更新时调用"""
        self.logger.info(f"订单状态更新: {order.id}, 状态: {order.status}")
    
    async def on_stop(self):
        """策略停止时调用"""
        self.logger.info("策略停止运行")
```

### 回测示例

```python
from src.backtest.engine import BacktestEngine
from src.analysis.performance import PerformanceAnalyzer

# 创建回测引擎
engine = BacktestEngine(
    strategy_class=MyStrategy,
    config_path="config/strategies/my_strategy.yaml",
    start_time="2022-01-01",
    end_time="2022-12-31"
)

# 运行回测
results = engine.run()

# 分析结果
analyzer = PerformanceAnalyzer(results)
report = analyzer.generate_report()
report.save("./output/my_strategy_report.html")
```

## 性能指标

TODO

## 工程目录结构

```
trader/
├── src/                  # 源代码目录
│   ├── data/             # 数据处理模块
│   │   ├── collectors/   # 数据采集器
│   │   ├── processors/   # 数据处理器
│   │   └── storage/      # 数据存储
│   ├── strategy/         # 策略模块
│   │   ├── factors/      # 因子库
│   │   ├── indicators/   # 技术指标
│   │   └── templates/    # 策略模板
│   ├── backtest/         # 回测模块
│   │   ├── engine/       # 回测引擎
│   │   └── analyzers/    # 回测分析器
│   ├── risk/             # 风险管理模块
│   │   ├── rules/        # 风控规则
│   │   └── monitors/     # 风险监控
│   ├── execution/        # 执行模块
│   │   ├── gateways/     # 交易网关
│   │   ├── adapters/     # 交易所适配器
│   │   └── algos/        # 执行算法
│   ├── analysis/         # 分析模块
│   │   ├── performance/  # 绩效分析
│   │   └── reports/      # 报告生成
│   ├── api/              # API服务
│   │   ├── rest/         # REST API
│   │   └── websocket/    # WebSocket API
│   ├── utils/            # 工具函数
│   │   ├── logger/       # 日志工具
│   │   └── config/       # 配置管理
│   │
│   └── models/           # 数据模型
├── tests/                # 测试代码
│   ├── unit/             # 单元测试
│   └── integration/      # 集成测试
├── docs/                 # 文档
│   ├── api/              # API文档
│   └── guides/           # 使用指南
├── scripts/              # 脚本工具
│   ├── setup/            # 环境设置
│   └── tools/            # 辅助工具
├── examples/             # 示例代码
│   ├── strategies/       # 策略示例
│   └── notebooks/        # Jupyter笔记本
├── requirements/         # 依赖管理
│   ├── base.txt          # 基础依赖
│   ├── dev.txt           # 开发依赖
│   └── prod.txt          # 生产依赖
├── docker/               # Docker配置
│   ├── dev/              # 开发环境
│   └── prod/             # 生产环境
├── .github/              # CI/CD配置
├── README.md             # 项目说明
└── LICENSE               # 许可证
```

## 技术栈

### 编程语言
- **Python 3.10+**: 策略研发、数据处理、API服务
- **C++ 17**: 高性能交易执行、订单路由

### 基础设施
- **Docker & Docker Compose**: 容器化部署
- **Git**: 版本控制
- **GitHub Actions**: CI/CD流程

### 数据层
- **PostgreSQL**: 结构化数据存储
- **Redis**: 缓存和高速数据访问
- **Parquet/Feather**: 高性能时间序列存储
- **Pandas/NumPy/Polars**: 数据处理
- **Kafka**: 消息队列和事件流

### 策略与回测
- **Backtrader**: 回测引擎
- **TA-Lib**: 技术分析库
- **Alphalens**: 因子分析
- **Pyfolio**: 投资组合分析

### API服务
- **FastAPI**: Web API框架
- **Pydantic**: 数据验证
- **WebSocket**: 实时数据推送

### 监控与分析
- **Prometheus & Grafana**: 系统监控
- **ELK Stack**: 日志管理
- **Plotly/Matplotlib**: 数据可视化

### 交易接口
- **CCXT**: 数字货币交易所API
- **IB API**: 国际市场接口
- **CTP**: 国内期货市场接口

## 开发计划

### 第一阶段：基础架构
- [x] 项目结构设计
- [ ] 核心数据模型定义
- [ ] 基础组件封装
- [ ] 数据存储层实现
- [ ] 基本市场数据获取

### 第二阶段：回测系统
- [ ] 回测引擎开发
- [ ] 基础指标库实现
- [ ] 策略模板设计
- [ ] 样例策略开发
- [ ] 回测性能分析工具

### 第三阶段：实盘交易
- [ ] 交易执行引擎
- [ ] 风控系统实现
- [ ] 交易所适配器
- [ ] 订单管理系统
- [ ] 实时监控模块

### 第四阶段：API与用户界面
- [ ] REST API开发
- [ ] WebSocket服务
- [ ] Web管理界面
- [ ] 报表与分析工具
- [ ] 用户权限系统

### 第五阶段：优化与扩展
- [ ] 性能优化
- [ ] 策略库扩展
- [ ] 多市场支持
- [ ] 机器学习模块
- [ ] 分布式部署支持

## 如何贡献

1. Fork本项目
2. 创建功能分支 (`git checkout -b feature/amazing-feature`)
3. 提交更改 (`git commit -m 'Add some amazing feature'`)
4. 推送到分支 (`git push origin feature/amazing-feature`)
5. 创建Pull Request

## 许可证

[MIT License](LICENSE)

## 联系方式

- 项目维护者: [jaryujun@gmail.com](mailto:jaryujun@gmail.com)
- 项目主页: [GitHub](https://github.com/jaryujun/trader)
