---
name: us-daily-news-brief
description: 获取美股相关新闻并进行情绪分析，识别市场情绪（risk-on/risk-off）、热门主题、新闻催化剂。这是情绪因子的核心数据来源。
user-invocable: true
version: 1.0.0
---

# 📰 美股每日新闻与情绪分析

## 触发条件
在市场数据获取完成后执行，为因子分析提供情绪面数据支撑。

## 执行步骤

### Step 1：获取新闻数据

#### 1.1 宏观新闻
使用 yfinance 获取市场相关新闻：
```python
import yfinance as yf

# 获取指数相关新闻
sp500 = yf.Ticker("^GSPC")
news_sp500 = sp500.news

# 获取重点板块新闻
for sector_etf in ["XLK", "SMH", "XLF", "XLE"]:
    sector = yf.Ticker(sector_etf)
    sector_news = sector.news
```

#### 1.2 个股新闻
```python
# 获取热门个股新闻
watchlist = ["AAPL", "MSFT", "NVDA", "TSLA", "GOOG", "AMZN", "META"]
for ticker in watchlist:
    stock = yf.Ticker(ticker)
    stock_news = stock.news
    # 提取：标题、发布时间、来源、摘要
```

### Step 2：新闻分类与标记

将新闻按以下维度分类：

#### 2.1 主题分类
- **宏观政策**：美联储、利率、通胀、就业
- **地缘政治**：贸易战、制裁、国际冲突
- **行业动态**：AI、半导体、新能源、医药
- **公司事件**：财报、并购、管理层变动、产品发布
- **市场情绪**：机构观点、资金流向、技术信号

#### 2.2 情绪标记
对每条新闻进行情绪打分：
- **正面 (+1)**：利好消息（盈利超预期、政策支持、新产品等）
- **中性 (0)**：信息性新闻（人事变动、行业数据等）
- **负面 (-1)**：利空消息（业绩下滑、监管风险、诉讼等）

### Step 3：宏观情绪判断

基于新闻聚合分析整体市场情绪：

```
总情绪得分 = Σ(单条新闻情绪分 × 新闻重要性权重)

if 总情绪得分 > 阈值:
    market_sentiment = "risk-on"（风险偏好）
elif 总情绪得分 < -阈值:
    market_sentiment = "risk-off"（风险规避）
else:
    market_sentiment = "neutral"（中性）
```

### Step 4：热门主题提取

识别当日 Top 3 热门主题：
- 统计新闻关键词频率
- 识别跨多条新闻的共同主题
- 评估主题对市场的影响程度

### Step 5：催化剂识别

为候选股票标注新闻催化剂：
- 是否有财报发布/预告
- 是否有机构评级变动
- 是否有重大公司事件
- 是否受板块联动影响

## 输出格式

```json
{
  "date": "YYYY-MM-DD",
  "market_sentiment": {
    "overall": "risk-on|risk-off|neutral",
    "score": 0.65,
    "confidence": "high|medium|low",
    "key_driver": "描述主要驱动因素"
  },
  "hot_topics": [
    {
      "rank": 1,
      "topic": "AI 芯片需求",
      "sentiment": "positive",
      "related_tickers": ["NVDA", "AMD", "AVGO"],
      "impact": "high|medium|low",
      "summary": "简要描述"
    }
  ],
  "key_news": [
    {
      "headline": "新闻标题",
      "source": "来源",
      "time": "发布时间",
      "category": "宏观政策|行业动态|公司事件",
      "sentiment": "+1|0|-1",
      "related_tickers": ["AAPL"],
      "significance": "high|medium|low"
    }
  ],
  "catalysts": [
    {
      "ticker": "NVDA",
      "type": "earnings|rating|product|policy",
      "description": "催化剂描述",
      "expected_impact": "positive|negative|mixed"
    }
  ]
}
```

## 数据验证
- 所有新闻必须来自 yfinance 或可验证的公开来源
- 禁止捏造新闻标题或内容
- 情绪判断必须基于新闻内容本身，不得主观臆断
- 如果新闻数据不足，必须明确标注"数据有限"

## 注意事项
- 新闻时效性：优先使用最近24小时内的新闻
- 重复新闻需去重，保留信息最完整的版本
- 区分事实报道与观点评论
