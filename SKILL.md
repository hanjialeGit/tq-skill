# 天启数据云 A 股数据 Skill

> 免鉴权，零依赖，直接用 curl 调用，支持 Claude / OpenAI / 通义千问等所有 Agent。

## 安装（两步，无需 pip）

```bash
mkdir -p ~/.claude/skills/tianqi-data
curl -o ~/.claude/skills/tianqi-data/SKILL.md \
  https://raw.githubusercontent.com/tianqi-data/tianqi-skill/main/SKILL.md
```

重启 Claude Code 后自动生效。

---

## 基础用法

所有接口均为 HTTP GET，用 curl 直接调用：

```bash
BASE="http://192.168.1.52:9001/blade-dataplatform/open/data"

# 查单只股票行情
curl -s "$BASE/quote?symbol=000001"

# 查股票基本信息（含 PE/PB/市值）
curl -s "$BASE/stock?symbol=000001"
```

---

## 工具列表

### 1. 实时行情 `quote`

查单只股票最新涨跌、量价。

```bash
curl -s "$BASE/quote?symbol=688017"
# 返回: symbol, name, trade_date, open, high, low, close,
#       pre_close, change, pct_chg, volume, amount
```

**示例问题**：「688017 今天涨了多少？」「茅台现在什么价格？」

---

### 2. 批量行情 `quotes`

最多同时查 10 只股票。

```bash
curl -s "$BASE/quotes?symbols=000001,600519,000858"
```

**示例问题**：「帮我看下茅台、五粮液、平安银行今天的涨跌」

---

### 3. 日K历史 `daily`

最近 N 条日K，最多 30 条。

```bash
curl -s "$BASE/daily?symbol=000001&limit=30"
# 返回: trade_date, open, high, low, close, volume, amount, pct_chg
```

**示例问题**：「平安银行最近 30 天走势」

---

### 4. 股票基本信息 `stock`

行业、市值、PE、PB、上市日期。

```bash
curl -s "$BASE/stock?symbol=688017"
# 返回: symbol, name, market, industry, pe, pb, total_mv, circ_mv
```

**示例问题**：「688017 的估值怎么样？PE 是多少？」

---

### 5. 股票搜索 `stocks`

按名称/代码关键词搜索，支持行业过滤。

```bash
curl -s "$BASE/stocks?q=银行&industry=银行&limit=20"
curl -s "$BASE/stocks?q=688017"
```

**示例问题**：「帮我找所有银行股」「搜索新能源相关股票」

---

### 6. 财务数据 `financial`

ROE、营收、净利润等，最多 4 期。

```bash
curl -s "$BASE/financial?symbol=000001&limit=4"
# 返回: report_period, report_type, roe, revenue, net_profit,
#       grossprofit_margin, eps, bps, debt_to_assets
```

**示例问题**：「平安银行的 ROE 怎么样？」「最近 4 期净利润趋势」

---

### 7. 最近新闻 `news`

最多 10 条，最多 168 小时内。

```bash
curl -s "$BASE/news?symbol=000001&hours=72&limit=10"
# 返回: title, publish_time, source, url, sentiment
```

**示例问题**：「平安银行最近有什么新闻？」「茅台近 3 天有没有利好？」

---

### 8. 十大股东 `holders`

最新期十大股东 / 十大流通股东。

```bash
curl -s "$BASE/holders?symbol=000001&holderCategory=top10_float"
# 返回: holder_name, hold_amount, hold_ratio, hold_change, holder_type
```

**示例问题**：「平安银行十大流通股东是哪些机构？」

---

### 9. ST 状态 `st`

是否 ST / 退市风险，正常股返回 null。

```bash
curl -s "$BASE/st?symbol=000001"
```

**示例问题**：「这只股票有退市风险吗？」

---

### 10. 宏观指标历史 `macro`

GDP / CPI / PPI / PMI，最多 12 条。

```bash
curl -s "$BASE/macro?type=CPI&limit=12"
# 返回: period, value, yoy_growth, mom_growth
```

**示例问题**：「最近 12 个月 CPI 走势」

---

### 11. 宏观指标最新值 `macro/latest`

```bash
curl -s "$BASE/macro/latest?type=PMI"
# 返回: period, value, unit, yoy_growth
```

**示例问题**：「现在 PMI 是多少？」「最新 GDP 增速」

---

### 12. 公司公告 `announcements`

按股票查最近公告（标题、AI 摘要、Markdown 完整正文、公告日期、类型、链接），最多 5 条。

```bash
curl -s "$BASE/announcements?symbol=000001&limit=5"
# 返回: title, summary, content, ann_date, publish_time, category,
#       importance, sentiment, keywords, source, url
# 说明: content 为 Markdown 格式公告全文，单条可能较长，故上限 5 条
```

**示例问题**：「平安银行最近发布了哪些公告？」「茅台最新公告的具体内容是什么？」

---

## 综合分析示例

问「帮我分析一下 688017 的估值」时，依次调用：

```bash
BASE="http://192.168.1.52:9001/blade-dataplatform/open/data"
curl -s "$BASE/stock?symbol=688017"       # PE/PB/市值
curl -s "$BASE/quote?symbol=688017"       # 当前股价
curl -s "$BASE/financial?symbol=688017"   # ROE/净利润
curl -s "$BASE/daily?symbol=688017&limit=30"  # 近期走势
```

---

## 注意事项

- 所有接口**只读、免鉴权**，无需注册或 token
- symbol 统一用 **6 位数字代码**（688017），不带交易所后缀
- 单次请求建议超时 10 秒
- 数据来源：天启数据云，与 A 股数据落库周期同步
