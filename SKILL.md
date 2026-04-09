# 股票技术分析技能 v1.8.0

专业的股票/基金技术分析工具，支持 A股、港股、美股、基金（ETF/LOF/开放式）的实时技术分析。

## 功能概览

- **多市场覆盖**：A股（沪深）、港股、美股
- **基金支持**：ETF、LOF、开放式基金
- **多数据源容错**：主源失败自动切换备用源
- **完整技术指标**：MA、MACD、RSI、KDJ、布林带、ATR、成交量分析、支撑压力位、缺口检测
- **MACD背离检测**：顶背离/底背离自动识别
- **量化评分系统**：0-100 综合评分（6维度）+ 5 档趋势判断
- **批量并发分析**：多线程并发，支持混合类型（股票+基金同时分析）
- **多种输出格式**：JSON / 终端表格 / ASCII模式 / 文件输出
- **离线测试**：`--test` 模式使用模拟数据，无需网络

## 项目结构

```
stock-technical-analysis/
├── _meta.json              # 技能元数据
├── SKILL.md                # 本文档
├── requirements.txt        # Python 依赖
└── scripts/
    └── stock_analysis.py   # 主脚本
```

## 快速开始

### 安装依赖

```bash
# 核心依赖（必需）
pip install pandas numpy requests

# 可选依赖（扩展数据源）
pip install akshare    # A股/基金备用数据源
pip install yfinance   # 港美股数据源
```

### 基本用法

```bash
# A股（自动识别沪深，默认表格输出）
python scripts/stock_analysis.py 600519

# 港股
python scripts/stock_analysis.py 00700 -m hkstock

# 美股
python scripts/stock_analysis.py AAPL -m usstock

# ETF 基金
python scripts/stock_analysis.py 159934 -t fund

# 开放式基金
python scripts/stock_analysis.py 001316 -t fund

# JSON 输出（供程序调用）
python scripts/stock_analysis.py 600519 --json --pretty
```

### 批量分析

```bash
# 批量股票
python scripts/stock_analysis.py -b 600036,600900,510310

# 混合类型（股票 + 基金）
python scripts/stock_analysis.py -b 001316:fund,600036:stock
```

### 其他选项

```bash
# 离线测试（模拟数据，无需网络）
python scripts/stock_analysis.py 600519 --test --pretty

# ASCII 模式（避免终端中文乱码）
python scripts/stock_analysis.py 600519 --ascii --pretty

# 查看版本
python scripts/stock_analysis.py --version

# 自定义数据天数
python scripts/stock_analysis.py 600519 -d 120
```

## 命令行参数

| 参数 | 缩写 | 说明 | 默认值 |
|------|------|------|--------|
| `stock_code` | - | 股票/基金代码（位置参数） | - |
| `--market` | `-m` | 市场：`auto` / `ashare` / `hkstock` / `usstock` | `auto` |
| `--type` | `-t` | 资产类型：`stock` / `fund` | `stock` |
| `--days` | `-d` | 历史数据天数 | `60` |
| `--batch` | `-b` | 批量分析（逗号分隔，支持 `code:type` 格式） | - |
| `--json` | `-j` | JSON 输出（默认为终端表格模式） | `false` |
| `--pretty` | `-p` | 格式化 JSON 输出（需配合 `--json`） | `false` |
| `--table` | `-T` | 终端表格输出（默认模式，可省略） | `true` |
| `--test` | - | 离线测试模式（模拟数据） | `false` |
| `--ascii` | - | ASCII 模式（中文替换为英文标签，表格和 JSON 均生效） | `false` |
| `--output` | `-o` | 输出到文件（如 `result.json`） | - |
| `--check` | - | 环境自检（检查依赖和数据源） | - |
| `--verbose` | - | 详细输出（显示 INFO 级别日志） | `false` |
| `--quiet` | - | 静默模式（仅输出错误） | `false` |
| `--watchlist` | `-w` | 使用持仓列表进行分析 | `false` |
| `--add` | - | 添加到持仓列表（如 `600519` 或 `159934:fund`） | - |
| `--remove` | - | 从持仓列表删除 | - |
| `--list` | - | 查看当前持仓列表 | - |
| `--version` | `-v` | 显示版本号 | - |

## 数据源架构

### A股（沪深）

```
历史数据:  新浪财经 API (主) → akshare (备用，3次重试)
实时行情:  新浪财经 API (主) → akshare (备用)
```

### 港股 / 美股

```
历史数据:  yfinance (主) → akshare (备用，3次重试)
实时行情:  yfinance (主) → akshare (备用)
```

### 基金

```
ETF/LOF:   新浪ETF接口 → 东财 akshare → 东财直接API
开放式基金: fund_open_fund_info_em → 东财直接API → LOF历史接口(回退)
```

## 代码自动识别规则

| 条件 | 市场 | 类型 |
|------|------|------|
| 6位数字，6/5/7/9 开头 | A股（上海） | 股票 |
| 6位数字，其他开头 | A股（深圳） | 股票 |
| 6位数字，15/51/58 开头 | A股 | ETF 基金 |
| 6位数字，16 开头 | A股 | LOF 基金 |
| 6位数字，00/01/11/18 开头 | A股 | 开放式基金 |
| 含 `HK.` 或 `港.` 前缀 | 港股 | 股票 |
| 纯字母 / 含 `US`/`NASDAQ`/`NYSE` | 美股 | 股票 |

## 技术指标

### MA 移动平均线

计算 MA5、MA10、MA20、MA60 四条均线，用于判断趋势方向和支撑压力。

### MACD 指数平滑异同移动平均线

- **DIF**：快线（EMA12 - EMA26）
- **DEA**：慢线（DIF 的 EMA9）
- **MACD 柱**：(DIF - DEA) × 2
- **信号**：金叉（DIF 上穿 DEA）/ 死叉（DIF 下穿 DEA）/ 中性
- **背离**：顶背离（价格新高但DIF降低）/ 底背离（价格新低但DIF升高）

### RSI 相对强弱指数

计算 RSI6、RSI12、RSI24 三个周期：
- RSI > 70：超买区域
- RSI < 30：超卖区域

### KDJ 随机指标

- **K**：RSV 的平滑值（参数 N=9, M1=3）
- **D**：K 的平滑值（参数 M2=3）
- **J**：3K - 2D
- **信号**：超买（J>100）/ 超卖（J<0）/ 多头 / 空头

### 布林带 (BOLL)

- **上轨**：MA20 + 2×标准差
- **中轨**：MA20
- **下轨**：MA20 - 2×标准差
- **bb_position**：价格在布林带中的位置百分比（0~1）
- **bb_width**：布林带宽度百分比
- **squeeze**：布林带收窄检测（暗示即将变盘）

### ATR 真实波幅

- **ATR**：N日（默认14日）真实波幅的移动平均
- **ATR_percent**：ATR 占当前价格的百分比，衡量波动强度

### 成交量分析

- **VMA5 / VMA10**：5日/10日成交量均线
- **量比**：当日成交量 / 过去5日平均成交量
- **信号**：显著放量(>=2.0) / 放量(>=1.5) / 正常 / 缩量(<=0.8) / 显著缩量(<=0.5)

### 支撑压力位

- **支撑位**：局部低点（前后5日最低，排除自身）
- **压力位**：局部高点（前后5日最高，排除自身）
- **缺口检测**：向上跳空 / 向下跳空
- **基金**：开放式基金（open=high=low=close）自动跳过支撑压力位计算

## 评分算法

综合评分基于 6 个维度，基础分 50 分：

| 维度 | 分值范围 | 规则 |
|------|---------|------|
| **均线趋势** | ±20 | 完美多头+20，短期多头+12，完美空头-20，短期空头-12（MA60缺失时仅评短期） |
| **价格与均线** | ±8 | 价格>MA5 ±5，价格>MA20 ±3 |
| **MACD** | ±15 | 金叉+10，死叉-10，MACD柱方向±5 |
| **RSI** | ±15 | <20 超卖+15，<30 +10，>80 超买-15，>70 -10 |
| **布林带** | ±10 | 突破下轨+10，突破上轨-10，接近下轨+5，接近上轨-5 |
| **KDJ** | ±8 | 超卖+8，超买-8，多头+4，空头-4 |
| **成交量** | ±5 | 显著放量+5，放量+3，显著缩量-3，缩量-1 |

### 评分映射

| 分数 | 趋势 | 技术面评估 |
|------|------|-----------|
| ≥75 | 强势上涨 | 技术面强势，多指标共振偏多 |
| 60-74 | 上涨趋势 | 技术面偏多，短期趋势向上 |
| 40-59 | 震荡整理 | 技术面中性，方向不明确 |
| 25-39 | 下跌趋势 | 技术面偏空，短期趋势向下 |
| <25 | 强势下跌 | 技术面弱势，多指标共振偏空 |

## 输出格式

### 单个分析

```json
{
  "stock_info": {
    "code": "600519",
    "name": "贵州茅台",
    "market": "ashare",
    "asset_type": "stock",
    "current_price": 1850.50,
    "change_pct": 1.25,
    "update_time": "2026-04-09 14:30:00"
  },
  "technical_indicators": {
    "ma": { "MA5": 1845.2, "MA10": 1838.1, "MA20": 1820.5, "MA60": 1790.3 },
    "macd": { "DIF": 8.52, "DEA": 5.31, "MACD": 6.42, "signal": "金叉" },
    "rsi": { "RSI6": 65.2, "RSI12": 58.3, "RSI24": 52.1 },
    "bollinger": { "upper": 1880.0, "middle": 1820.5, "lower": 1761.0, "position": "高位", "bb_position": 0.78 }
  },
  "key_levels": {
    "support": [1800.0, 1780.5, 1761.0],
    "resistance": [1880.0, 1900.0],
    "gaps": [{ "type": "向上跳空", "gap_start": 1855.0, "gap_end": 1860.0, "size": 0.27 }]
  },
  "analysis": {
    "score": 72,
    "trend": "上涨趋势",
    "recommendation": "技术面偏多，短期趋势向上",
    "summary": "贵州茅台现价1850.5，上涨1.25%；均线多头排列；MACD金叉；评分72"
  }
}
```

### 批量分析

```json
{
  "results": [...],
  "summary": {
    "强势上涨": 1,
    "上涨趋势": 2,
    "震荡整理": 1,
    "下跌趋势": 0,
    "强势下跌": 0,
    "total": 4,
    "valid": 4,
    "failed_count": 0,
    "avg_score": 62.3
  },
  "failed": null,
  "timestamp": "2026-04-09 14:30:00"
}
```

## 触发关键词

- 股票分析 / 分析股票 / 股票走势 / 技术分析
- 基金分析 / 基金净值 / 基金走势
- 诊股 / 看盘 / 盯盘

## 注意事项

1. **数据延迟**：免费数据源可能有 15-20 分钟延迟
2. **基金净值**：开放式基金每日更新一次，无实时行情
3. **仅供参考**：分析结果不构成投资建议
4. **网络要求**：核心功能需要网络连接（`--test` 模式除外）

## 更新日志

### v1.8.0 (2026-04-09)

**新功能：配置文件与持仓管理**
- 新增配置文件支持：`~/.stock-analysis/config.json`，自动创建
- 新增 `--add CODE[:TYPE]` 添加到持仓列表
- 新增 `--remove CODE` 从持仓列表删除
- 新增 `--list` 查看当前持仓列表
- 新增 `--watchlist` / `-w` 使用持仓列表进行批量分析
- 支持配置文件中的默认参数（market/days/asset_type），未显式指定时自动读取

### v1.7.0 (2026-04-09)

**网络可靠性加固：**
- 新浪财经主数据源增加重试机制（3 次重试，间隔 1s）
- yfinance 主数据源增加重试机制（3 次重试，间隔 2s）
- 批量模式失败项自动重试一轮（偶发网络问题自动恢复）
- 错误提示增强：区分"依赖缺失"、"网络超时"和"无数据"

**体验优化：**
- 参数互斥校验：`--batch` + `stock_code`、`--json` + `--table` 同时传入时警告
- `--ascii` 模式现在同时影响表格和 JSON 输出（GBK 终端下表格也可用）
- 新增 `--verbose` 参数（显示 INFO 级别日志，方便调试）
- 新增 `--quiet` 参数（仅输出错误，适合脚本调用）
- 修正 requests 依赖标记为"必需"（实际是 A 股数据核心依赖）

### v1.6.1 (2026-04-09)

**新增功能：**
- 新增 `--check` 环境自检命令：检测依赖安装状态（pandas/numpy/requests/akshare/yfinance/wcwidth）+ 数据源连通性（新浪/yfinance/akshare），输出清晰的状态报告

**Bug 修复：**
- 修复 `--test` 模式下 `-d` 参数被静默忽略的问题（`analyze_with_mock_data` 现在接受 `days` 参数）

### v1.6.0 (2026-04-09)

**体验优化（PM 审查改进）：**
- 默认输出改为终端表格模式（原默认为 JSON），新增 `--json` / `-j` 参数用于 JSON 输出
- 投资建议措辞弱化为纯技术面描述（如"强烈建议买入" → "技术面强势，多指标共振偏多"），消除合规风险
- 修复终端表格中文对齐错位问题（使用 wcwidth 计算中文字符显示宽度）
- `--output` 支持表格模式输出到文件

### v1.5.1 (2026-04-09)

**Bug 修复：**
- 修复 `--table` 模式在 GBK 编码终端下 UnicodeEncodeError 崩溃（Unicode 字符替换为 ASCII）

### v1.5.0 (2026-04-09)

**新增技术指标：**
- KDJ 随机指标（K/D/J + 超买超卖信号）
- ATR 真实波幅（ATR + 波幅百分比）
- 成交量分析（VMA5/VMA10 + 量比 + 量能信号）
- 布林带宽度（bb_width）和收窄检测（squeeze）
- MACD 背离检测（顶背离/底背离，最近20日）

**评分体系扩展：**
- 从 4 维度扩展到 6 维度（+KDJ ±8，+成交量 ±5）
- 均线评分分层处理（MA60 缺失时仍评短期趋势）

**新增功能：**
- `--table` 终端表格输出（人类可读格式，单只+批量）
- `--output` 输出到文件
- 批量并发请求（ThreadPoolExecutor，8线程）

### v1.4.0 (2026-04-09)

**Bug 修复：**
- P0: 修复支撑压力位局部极值算法逻辑错误（包含自身比较）
- P0: 修复 yfinance history() period 参数兼容性
- P0: 修复 akshare A股 symbol 参数格式错误
- P1: NaN 安全检查（current_price/bollinger/RSI）
- P1: 缺口检测除零保护
- P1: 重试后返回脏 DataFrame
- P1: 均线评分 MA60 缺失时全跳过
- P1: analyze_batch days 参数未传递
- P1: 新浪 HTTP 状态码未检查

**数据一致性：**
- 新浪数据源补充 amount 列
- yfinance 实时行情补充 open/prev_close/amount 字段
- 场外基金名称解析（通过 fund_name_em 查询）
- LOF 基金名称解析增强

**代码质量：**
- 基金类型跳过无意义的支撑压力位计算
- ASCII 映射表扩展覆盖建议/摘要常用词

**功能增强：**
- `--output` 参数支持输出到文件
- 港股识别优先级优化（HK. 前缀优先于基金规则）
- 基金代码识别规则收窄（00/01 开头不再自动识别为基金）

### v1.3.0 (2026-04-09)

**代码规范化：**
- 移除所有 DEBUG 日志，改用标准 `logging` 模块
- 修复所有 bare `except` 为具体异常类型
- 统一版本号为 `v1.3.0`
- 规范导入顺序（标准库 → 第三方 → 可选）
- 魔法数字提取为模块级常量
- 新增 `_build_sina_symbol()` / `_build_yfinance_code()` 辅助方法消除重复代码
- 新增 `--version` 命令行参数
- ASCII 映射表提取为模块级常量 `ASCII_REPLACE_MAP`

**Bug 修复：**
- 修复 `--test` 模式下基金代码解析失败（模拟数据直接生成 `pct_change`，不依赖实时行情）
- 修复布林带中位档评分：中位 (bb_position 0.3~0.7) 不加减分，提升评分区分度
- 修复 `fetch_realtime_quote_sina` 缺少 `amount` / `open` / `prev_close` 字段
- 修复 RSI 计算中 loss=0 时的除零错误
- 修复 `fetch_realtime_quote_yfinance` 的 `pct_change` 百分比换算

**项目结构：**
- 删除 `FEEDBACK.md` 和 `INTEGRATION.md`（开发过程文档）
- 新增 `requirements.txt` 依赖管理文件
- 更新 `_meta.json` 元数据

### v1.2.2-fix (2026-04-08)

- 修复 `fund_open_fund_daily_em` 参数问题，改用 `fund_open_fund_info_em(symbol=code)`

### v1.2.1 (2026-04-08)

- 修复基金接口 `fund_zh_a_hist` 不存在的问题
- 批量分析增加 `failed` 失败列表
- 支持混合类型批量 `code:type` 格式

### v1.2.0 (2026-04-08)

- 新增基金分析功能（ETF/LOF/开放式）
- 港美股新增 yfinance 数据源
- 新增批量分析功能
- 新增 `--ascii` 模式

### v1.1.0 (2026-04-08)

- 新浪财经作为 A 股主数据源
- akshare 改为可选依赖
- 修复东财接口不稳定问题
