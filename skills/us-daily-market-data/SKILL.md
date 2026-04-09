---
name: us-daily-market-data
description: 获取美股每日市场数据，包括三大指数（S&P500/Nasdaq/Dow）、宏观指标（VIX/利率/DXY）以及板块表现。这是投研流程的第一步。
user-invocable: true
version: 1.0.0
---

# 📊 美股每日市场数据获取

## 触发条件
当用户请求生成每日投研报告、或需要获取最新市场数据时调用此技能。

## 执行步骤

### Step 1：获取三大指数数据
使用 yfinance 获取以下指数的最新交易日数据：

```python
import yfinance as yf

indices = {
    "S&P 500": "^GSPC",
    "Nasdaq": "^IXIC",
    "Dow Jones": "^DJI"
}

for name, ticker in indices.items():
    data = yf.Ticker(ticker)
    hist = data.history(period="5d")
    # 获取：收盘价、涨跌幅、成交量
```

### Step 2：获取宏观指标数据
```python
macro_tickers = {
    "VIX": "^VIX",
    "10Y Treasury": "^TNX",
    "DXY": "DX-Y.NYB"
}

for name, ticker in macro_tickers.items():
    data = yf.Ticker(ticker)
    hist = data.history(period="5d")
    # 获取：当前值、变化幅度、趋势方向
```

### Step 3：获取板块 ETF 表现
```python
sectors = {
    "科技": "XLK",
    "医疗": "XLV",
    "金融": "XLF",
    "能源": "XLE",
    "消费": "XLY",
    "工业": "XLI",
    "材料": "XLB",
    "公共事业": "XLU",
    "房地产": "XLRE",
    "通讯": "XLC",
    "必需消费": "XLP",
    "半导体": "SMH",
    "AI/云计算": "CLOU"
}
```

## 输出格式

```json
{
  "date": "YYYY-MM-DD",
  "indices": [
    {
      "name": "S&P 500",
      "close": 0.00,
      "change_pct": 0.00,
      "volume": 0,
      "trend": "up|down|flat"
    }
  ],
  "macro": [
    {
      "name": "VIX",
      "value": 0.00,
      "change": 0.00,
      "signal": "risk-on|risk-off|neutral"
    }
  ],
  "sectors": [
    {
      "name": "科技",
      "ticker": "XLK",
      "change_pct": 0.00,
      "relative_strength": "strong|neutral|weak"
    }
  ]
}
```

## 数据验证
- 确认数据日期为最新交易日
- 确认所有字段非空
- 检查涨跌幅是否在合理范围内（单日超过±10%需标注异常）
- 如果 yfinance 返回空数据，必须明确报告并标注"数据缺失"

## 注意事项
- 禁止编造任何数据，所有数值必须来自 yfinance
- 如果非交易日，使用最近交易日的数据并注明
- 数据保留2位小数
