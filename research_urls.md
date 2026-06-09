# 研究数据收集 SOP

**用法**：所有 URL 把 `{TICKER}` 替换成股票代码（大写）。
我（Claude）做 teardown 时按这个列表挨个 WebFetch / 调用 MCP，**不用你手动复制**。

**Stack 月成本：$0**

---

## 一、原始 SEC 文件（最权威）→ EdgarTools MCP

不用 URL——我直接调用 MCP：

| 我调用 | 给我什么 |
|---|---|
| `edgar_company(identifier="{TICKER}")` | 公司 profile + 财务 + 最近 filings + 持股结构 |
| `edgar_filing(identifier="{TICKER}", form="10-Q")` | 最新季报 metadata |
| `edgar_read(identifier="{TICKER}", form="10-Q", sections=["mda","risk_factors","financials"])` | MD&A 全文 + 风险段 + 报表 |
| `edgar_filing(identifier="{TICKER}", form="10-K")` | 年报（业务描述、moat 论据最完整）|
| `edgar_filing(identifier="{TICKER}", form="8-K")` | 重大事件 / 财报新闻稿 |
| `edgar_ownership(identifier="{TICKER}")` | 13F 机构持仓 + Form 4 内部人交易 |
| `edgar_proxy(identifier="{TICKER}")` | DEF 14A：高管薪酬 + 公司治理 |
| `edgar_trends(identifier="{TICKER}")` | 财务纵向趋势 |
| `edgar_compare([...])` | 跨公司对比（比如 INTU vs WDAY vs NOW）|

**覆盖范围**：10-K / 10-Q / 8-K / Form 4 / 13F / DEF 14A / S-1 等所有 SEC 文件，**永久免费、SEC 官方源**。

---

## 二、实时市场数据（价、市值、ratios）→ WebFetch

### 主选：stockanalysis.com
| URL 模板 | 给我什么 |
|---|---|
| `https://stockanalysis.com/stocks/{ticker}/` | 概览 + 实时价 + 简要财务 |
| `https://stockanalysis.com/stocks/{ticker}/statistics/` | P/E、市值、52w、ratios、beta、股息 |
| `https://stockanalysis.com/stocks/{ticker}/financials/` | 收入表（季度+年度）|
| `https://stockanalysis.com/stocks/{ticker}/financials/balance-sheet/` | 资产负债表 |
| `https://stockanalysis.com/stocks/{ticker}/financials/cash-flow-statement/` | 现金流量表 |
| `https://stockanalysis.com/stocks/{ticker}/financials/ratios/` | 详细 ratios |
| `https://stockanalysis.com/stocks/{ticker}/forecast/` | 卖方共识 EPS + 收入 + 目标价 |

### 备选：Yahoo Finance
| URL | 给我什么 |
|---|---|
| `https://finance.yahoo.com/quote/{TICKER}/` | 实时价 + 关键数字 |
| `https://finance.yahoo.com/quote/{TICKER}/key-statistics/` | 估值倍数（更全）|
| `https://finance.yahoo.com/quote/{TICKER}/analysis/` | 分析师 EPS/收入预期 + 趋势 |
| `https://finance.yahoo.com/quote/{TICKER}/profile/` | 公司简介 |

### 历史财务（10-30 年纵向）
| URL | 给我什么 |
|---|---|
| `https://www.macrotrends.net/stocks/charts/{TICKER}/{company-name-slug}/revenue` | 收入 30 年 |
| `https://www.macrotrends.net/stocks/charts/{TICKER}/{company-name-slug}/free-cash-flow` | FCF 30 年 |
| `https://www.macrotrends.net/stocks/charts/{TICKER}/{company-name-slug}/pe-ratio` | P/E 历史 |

**注**：Macrotrends URL 需要 company slug（INTU 是 `intuit`），WebSearch 一下即可。

---

## 三、财报电话原话（管理层 Q&A 全文）→ WebFetch

财报后 24-48 小时内**公开免费**：

| 源 | URL 模板 |
|---|---|
| **Motley Fool**（最快+稳）| `https://www.fool.com/earnings/call-transcripts/{YYYY}/{MM}/{DD}/{company-name}-q{N}-{YEAR}-earnings-call-tran/` |
| **AlphaStreet** | `https://www.alphastreet.com/news/category/transcripts/` → 搜 ticker |
| **公司 IR 页**（最权威）| `https://investors.{company}.com/news-events/events/` |

**操作**：财报日期已知后，我用 WebSearch `"{ticker} Q{N} 20XX earnings call transcript site:fool.com"` 拿到精确 URL，再 WebFetch。

---

## 四、机构动向 + 内部人（更易读的呈现）→ WebFetch

**EdgarTools 已经有原始数据，但下面的网站把它 pre-process 成更易读形式：**

| URL | 给我什么 |
|---|---|
| `https://whalewisdom.com/stock/{ticker}` | 顶级基金（Berkshire、Pershing 等）持仓变动 |
| `https://www.openinsider.com/screener?s={TICKER}` | CEO/CFO/董事 90 天买卖 |
| `https://hedgefollow.com/stocks/{TICKER}` | 对冲基金持仓汇总 |

---

## 五、行业 / 宏观 / 顶级投资人 reasoning → WebFetch

### Hedge fund letters（季度 reasoning 范本）
| 经理 | URL |
|---|---|
| **Howard Marks (Oaktree)** | `https://www.oaktreecapital.com/insights/memos` |
| **Warren Buffett (Berkshire)** | `https://www.berkshirehathaway.com/letters/letters.html` |
| **Bill Ackman (Pershing Square)** | `https://pershingsquareholdings.com/company-reports/` |
| **David Einhorn (Greenlight)** | `https://www.greenlightcapital.com/` （季报区）|
| **Mohnish Pabrai (Pabrai Funds)** | `https://www.pabraifunds.com/` |

### 宏观
| 来源 | URL |
|---|---|
| **Torsten Slok (Apollo)** | `https://www.apolloacademy.com/the-daily-spark/` |
| **Ben Thompson (Stratechery)** 免费 | `https://stratechery.com/` |
| **Marc Rubinstein (Net Interest)** 免费部分 | `https://www.netinterest.co/` |

### 市场实时
- WebSearch：`"S&P 500 today"`、`"VIX"`、`"10Y Treasury yield"`、`"hyperscaler capex 2026"` 等

---

## 六、我的持仓 + 下单建议 → IBKR MCP

| 我调用 | 给我什么 |
|---|---|
| `get_account_positions` | 当前持仓 + 成本 + 当前价 |
| `get_account_balances` | 现金 + margin + 净值 |
| `get_account_trades` | 历史成交 |
| `get_price_snapshot` | 实时报价（你账号 entitlement 内）|
| `create_order_instruction` | 起草限价单**草稿**——你在 IBKR 端 approve 才会真下 |

**用途**：财报后做"从今天往前看"胜率赔率时，对照你的实际仓位 + 成本；触发 pre-commit 限价时帮你起草订单。

---

## 七、综合搜索（trial 期间）→ Bigdata.com MCP

trial 在 ~13 天后退化到 Free（功能受限）。期间用作：

| 我调用 | 给我什么 |
|---|---|
| `bigdata_search` smart mode | 跨源 semantic 搜索（财报 + 新闻 + 研报 + 转录）|
| `bigdata_company_tearsheet` | 综合 tearsheet（输出大，慎用）|
| `bigdata_events_calendar` | 财报日历 |

**trial 到期后**：能搜的内容缩水到 Free 公开内容——届时主力切回 EdgarTools + WebFetch。

---

## 标准 Teardown 工作流（我的执行顺序）

```
1. 锚定（30 秒）
   - WebFetch stockanalysis.com/stocks/{ticker}/statistics/
   - WebFetch stockanalysis.com/stocks/{ticker}/forecast/
   → 拿到：现价、P/E、市值、52w、共识目标

2. 业务理解（5 分钟）
   - edgar_company(ticker, include=["profile","financials","filings"])
   - edgar_read(ticker, form="10-K", sections=["business","risk_factors"])
   → 拿到：5 句话讲清生意 + 壁垒原文 + 风险段

3. 最新季度跟踪（5 分钟）
   - edgar_filing(ticker, form="10-Q")  
   - edgar_read(ticker, form="10-Q", sections=["mda","financials"])
   - WebSearch "{ticker} Q{N} earnings call transcript site:fool.com"
   - WebFetch 命中的 Motley Fool URL
   → 拿到：上季度财务 + CEO 原话 + Q&A

4. 共识 vs 我（3 分钟）
   - WebFetch finance.yahoo.com/quote/{ticker}/analysis/
   - WebSearch "{ticker} analyst downgrade upgrade 2026"
   → 拿到：卖方共识 + 主要分歧点

5. 持仓状态 + 仓位（2 分钟）
   - get_account_positions
   → 对照框架格子结构 + add zone

6. 综合 + 决策（输出）
   - 套用 CLAUDE.md 三原则
   - 用 Marks 框架做 gap-check
   - 写进 finance repo decision journal
```

**总 friction**：每次 teardown ~15-20 分钟数据收集 + 我推理时间。

---

## 维护

这个文件**不固定**——遇到新的好 URL 或工具就加进来。每季度回顾一次：
- 删掉没用过的源
- 加入新发现
- 检查所有 URL 是否还有效（产品改版会 break URL）
