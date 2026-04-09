---
name: us-daily-equity-research
description: 执行美股个股多因子分析，包括基本面（收入/盈利/估值）、技术面（MA/RSI/支撑阻力）分析，并输出选股评分。这是因子层和决策层的核心技能。
user-invocable: true
version: 1.0.0
---

# 📈 美股个股多因子研究

## 触发条件
在市场数据和新闻数据获取完成后，执行个股因子分析和选股决策。

## 执行步骤

### Step 1：候选池构建
从以下来源筛选候选股票（20-30只）：
- 当日成交量异常放大的股票（对比20日均量 > 150%）
- 当日波动率排名靠前的股票
- 新闻催化涉及的股票
- 近期发布财报的股票
- 板块联动中领涨/领跌的股票

```python
import yfinance as yf

# 示例：获取候选股数据
def get_candidate_data(ticker):
    stock = yf.Ticker(ticker)
    hist = stock.history(period="6mo")
    info = stock.info
    return hist, info
```

### Step 2：基本面因子（Fundamental）

对每只候选股计算：

#### 2.1 收入增长
```python
# 从 stock.financials 获取
# revenue_growth = (latest_revenue - prev_revenue) / prev_revenue
```
- 评分：> 20% = 强 | 10-20% = 中 | < 10% = 弱

#### 2.2 盈利变化
```python
# 从 stock.financials 获取 Net Income
# earnings_change = (latest_earnings - prev_earnings) / abs(prev_earnings)
```
- 评分：正增长 = 正面 | 负增长 = 负面 | 扭亏 = 强正面

#### 2.3 估值水平
```python
# PE = stock.info.get('trailingPE') 或 'forwardPE'
# EV/EBITDA = stock.info.get('enterpriseToEbitda')
```
- 评分标准（相对行业）：
  - 低估（< 行业均值 0.8x）= 有吸引力
  - 合理（0.8x - 1.2x）= 中性
  - 高估（> 1.2x）= 需警惕

### Step 3：技术面因子（Technical）

#### 3.1 均线系统
```python
hist = stock.history(period="1y")
ma20 = hist['Close'].rolling(20).mean()
ma50 = hist['Close'].rolling(50).mean()
ma200 = hist['Close'].rolling(200).mean()

# 判断：
# 多头排列（MA20 > MA50 > MA200）= 强势
# 空头排列（MA20 < MA50 < MA200）= 弱势
# 交叉 = 信号
```

#### 3.2 RSI（14日）
```python
delta = hist['Close'].diff()
gain = delta.where(delta > 0, 0).rolling(14).mean()
loss = (-delta.where(delta < 0, 0)).rolling(14).mean()
rs = gain / loss
rsi = 100 - (100 / (1 + rs))

# RSI > 70 = 超买
# RSI < 30 = 超卖
# 40-60 = 中性
```

#### 3.3 支撑与阻力
```python
# 基于近20日高低点
support = hist['Low'].rolling(20).min().iloc[-1]
resistance = hist['High'].rolling(20).max().iloc[-1]
current = hist['Close'].iloc[-1]

# 距支撑位比例 = (current - support) / current
# 距阻力位比例 = (resistance - current) / current
```

### Step 4：多因子评分与选股

综合三大因子打分（满分100）：

| 因子类别 | 权重 | 评分维度 |
|---------|------|---------|
| 基本面 | 35% | 收入增长 + 盈利变化 + 估值 |
| 情绪面 | 30% | 新闻热度 + 催化剂强度（来自 news-brief 技能）|
| 技术面 | 35% | 均线 + RSI + 支撑阻力 |

选出综合得分 Top 5 的股票。

## 输出格式

```json
{
  "analysis_date": "YYYY-MM-DD",
  "candidates_screened": 25,
  "top_5": [
    {
      "rank": 1,
      "ticker": "AAPL",
      "company": "Apple Inc.",
      "sector": "Technology",
      "score": 85,
      "signal_strength": "strong|moderate|weak",
      "fundamental": {
        "revenue_growth": "15.2%",
        "earnings_change": "8.5%",
        "pe_ratio": 28.5,
        "ev_ebitda": 22.1,
        "assessment": "合理估值，盈利稳健"
      },
      "technical": {
        "price": 185.50,
        "ma20": 182.30,
        "ma50": 178.60,
        "ma200": 170.20,
        "ma_alignment": "多头排列",
        "rsi_14": 62.5,
        "rsi_signal": "中性偏强",
        "support": 175.00,
        "resistance": 190.00
      },
      "catalyst": "来自 news-brief 的催化剂摘要",
      "reason": "选股理由（综合多因子）"
    }
  ]
}
```

## 数据验证
- 基本面数据必须来自 yfinance `.info` 和 `.financials`
- 技术面数据必须通过计算得出，不得估算
- 如果某只股票缺失关键数据字段，降低其排名或排除
- 选股理由必须引用具体数据点

## 注意事项
- 禁止根据"感觉"或"经验"选股
- 每个 Top 5 股票必须有完整的三因子分析
- 如果数据不足以支撑结论，必须明确标注
