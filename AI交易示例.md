# AI 交易示例（WorkBuddy · Codex · Hermes Agent）

在 **WorkBuddy**、**Codex** 或 **Hermes Agent** 中配置 MCP 服务 **`rhths-trade`** 后，用自然语言即可驱动**本机已登录**的 THS 账户，并（可选）调用**同花顺金融数据服务**云端研究接口（亦可 OpenClaw / Cursor 等）。

- **默认模拟下单**（`dry_run` / 模式 `simulate`），不会真实成交。
- **实盘**须环境变量 `RHTHS_ALLOW_LIVE=1`，且每笔写操作带 **`confirm_live: true`**，风险自负。
- **云端研究**（涨停池/财报/估值/基金等）：到 [fuyao.aicubes.cn](https://fuyao.aicubes.cn/admin/) **自行申请 Token**，在 `rhths-gui` → **「API」** 页保存。
- **能力分工：** 实时行情/交易 → `rh_market_*` / `rh_trade_*`；云端特色与财务 → `rh_fuyao_*`；云端自选股/自选板块/动态板块 → `rh_ths_watch_*` / `rh_ths_favorite_*` / `rh_ths_block_*`（GUI「自选」页 Cookie）。
- 配置见 [MCP使用说明.md](./MCP使用说明.md)、[SKILL.md](./SKILL.md)、[fuyao-routing.md](./fuyao-routing.md)。

---

## 使用前（建议先说给 AI）

| 你可以对 AI 说 | MCP 工具 |
|----------------|----------|
| 「先检查 RHTHS 和 THS 是否正常」 | `rh_trade_health` |
| 「hexin 和 xiadan 在跑吗？」 | `rh_system_status` |
| 「现在是模拟还是实盘模式？」 | `rh_trade_mode_get` |
| 「官方数据 Key 通不通？」 | `rh_fuyao_ping` |
| 「我的同花顺自选股里有哪些？」 | `rh_ths_watch_list` |
| 「自选板块有哪些？」 | `rh_ths_favorite_list` |
| 「动态板块「5日线」里有哪些票？」 | `rh_ths_block_list`（`group`=`5日线`） |

---

## 一、系统与模式

| 你可以对 AI 说 | MCP 工具 | 说明 / 常用参数 |
|----------------|----------|-----------------|
| 「网关和 THS API 就绪吗？」 | `rh_trade_health` | 返回网关版本、`ths_api` 是否可用 |
| 「列出网关支持哪些接口」 | `rh_trade_catalog` | 查看 HTTP 路由列表（约 50+ 条） |
| 「hexin / xiadan 是否在运行？」 | `rh_system_status` | `hexin_running`、`xiadan_running`、配置路径 |
| 「当前是模拟还是实盘？」 | `rh_trade_mode_get` | 与 GUI 共用 `%APPDATA%\RHTHS\settings.json` |
| 「切换到模拟模式」 | `rh_trade_mode_set` | `mode`: `simulate` |
| 「切换到实盘模式」（需谨慎） | `rh_trade_mode_set` | `mode`: `live`；实盘下单仍需 `confirm_live` |

**示例对话：**

> 用户：帮我看一下 RHTHS 是否正常，并告诉我现在是模拟还是实盘。  
> AI：调用 `rh_trade_health` → `rh_trade_mode_get`，用自然语言汇总结果。

> 用户：THS 主程序和下单程序都在跑吗？  
> AI：`rh_system_status`，说明 hexin / xiadan 运行状态及 exe 路径。

---

## 二、账户与交易查询（只读）

| 你可以对 AI 说 | MCP 工具 | 说明 / 常用参数 |
|----------------|----------|-----------------|
| 「我有哪些资金账号？」 | `rh_trade_users` | 多账号时列出；可选 `user`、`rzrq`（融资融券） |
| 「查一下资金」 | `rh_trade_account` | 可用资金、总资产等（默认会写当日快照） |
| 「帮我看下资金和持仓」 | `rh_trade_account` + `rh_trade_positions` | 常组合使用 |
| 「查 1 月份每天的资金快照」 | `rh_trade_account_daily` | `start_date`/`end_date`: **`YYYYMMDD`**（不是带横杠的日期） |
| 「持仓里 603919 还能卖多少？」 | `rh_trade_positions` | 含 `hold_qty`、`sellable_qty`（T+1） |
| 「今天有哪些委托？」 | `rh_trade_orders_today` | 当日全部委托 |
| 「今天还没成交的委托有哪些？」 | `rh_trade_orders_today` | 让 AI 按返回字段筛选未完成 |
| 「查一下最近半个月的历史委托」 | `rh_trade_orders_history` | 默认近 15 日、不含今天 |
| 「查 600000 今年 1 月的委托」 | `rh_trade_orders_history` | `start_date`/`end_date`: `YYYYMMDD`，`code`: `600000` |
| 「批量查询前先预热缓存」 | `rh_trade_warmup` | 大量查询前可选 |

**示例对话：**

> 用户：帮我看下资金和持仓，重点看 603919 还能卖多少。  
> AI：`rh_trade_account` → `rh_trade_positions`，解读可卖数量。

> 用户：列出我今天所有买入委托。  
> AI：`rh_trade_orders_today`，筛选买入方向。

> 用户：对比一下最近一周每天的总资产变化。  
> AI：`rh_trade_account_daily`，`start_date`/`end_date` 设为近 7 日（`YYYYMMDD`），汇总 `total_asset`。

---

## 三、下单与撤单（写操作 · 默认模拟）

| 你可以对 AI 说 | MCP 工具 | 说明 / 常用参数 |
|----------------|----------|-----------------|
| 「模拟买入 100 股 600000，最新价」 | `rh_trade_buy` | `code`, `qty`, `price`: `zxjg`，`dry_run`: `true` |
| 「模拟卖出 603919 100 股，对手价三档」 | `rh_trade_sell` | `price`: 常用 `dsj3`（卖） |
| 「撤销今天所有未成交委托（模拟）」 | `rh_trade_cancel` | `scope`: `all`，`dry_run`: `true` |
| 「一键全部撤单（模拟）」 | `rh_trade_cancel_all` | `dry_run`: `true` |
| 「撤销合同号 xxx 的委托」 | `rh_trade_cancel` | `htbh`: 合同编号 |
| 「用原始命令模拟卖 603919」 | `rh_trade_invoke` | `cmd`: 如 `sell 603919 dsj3 100 -notip` |

**价格模式参考：**

| 场景 | 常用 `price` |
|------|----------------|
| 买入 | `zxjg`（最新价）等 |
| 卖出 | `dsj3`（对手价三档）等 |

**示例对话：**

> 用户：模拟买入 100 股贵州茅台，用最新价。  
> AI：`rh_trade_buy`，`code`: `600519`，`qty`: `100`，`price`: `zxjg`，`dry_run`: `true`。

> 用户：把我持仓里 603919 可卖数量的一半模拟卖掉。  
> AI：先 `rh_trade_positions` 读 `sellable_qty`，再 `rh_trade_sell`（模拟）。

> 用户：今天所有挂单先模拟全部撤掉。  
> AI：`rh_trade_cancel_all`，`dry_run`: `true`。

---

## 四、实盘（高风险 · 须双重确认）

| 你可以对 AI 说 | MCP 工具 | 必要条件 |
|----------------|----------|----------|
| 「实盘卖出 100 股 603919，我已确认风险」 | `rh_trade_sell` | 交易机 `RHTHS_ALLOW_LIVE=1`，`dry_run`: `false`，`confirm_live`: `true` |
| 「实盘买入…」 | `rh_trade_buy` | 同上 |
| 「实盘撤销某笔委托」 | `rh_trade_cancel` | 同上 |
| 「实盘全部撤单，我已确认」 | `rh_trade_cancel_all` | 同上 |

> **标准版**：实盘买入+卖出合计每天最多 **10 次**；超限返回 `FREE_DAILY_LIMIT`。高级版在 GUI 激活后无此限制。

**示例对话：**

> 用户：我已在 GUI 打开实盘并确认风险，请实盘卖出 100 股 603919，对手价三档。  
> AI：确认环境后 `rh_trade_sell`，`dry_run`: `false`，`confirm_live`: `true`。

---

## 五、本机行情与问财

| 你可以对 AI 说 | MCP 工具 | 说明 / 常用参数 |
|----------------|----------|-----------------|
| 「603919 现在什么价？」 | `rh_market_quote` | **单票必用**；`code`: 6 位 |
| 「603919 和 300033 现价多少？」 | `rh_market_price` | `codes` 数组 |
| 「拉 600000 最近 30 根日 K」 | `rh_market_kline` | `period`: **`1440`**=日线，`length`: `30` |
| 「看 603919 的 15 秒 K 线」 | `rh_market_kline_sec` | `sec`: `15` 或 `30` |
| 「603919 今天分时走势」 | `rh_market_timeshare` | `code` |
| 「603919 五档盘口」 | `rh_market_depth` | `code`；`max_levels` 默认 5 |
| 「问财：市盈率小于 20 的沪深 A 股」 | `rh_market_wencai` | `query`：中文问财话术（多回 codes） |
| 「问财同上，要完整字段明细」 | `rh_market_wencai_detail` | `query` |
| 「只要代码列表」 | `rh_market_select_stocklist` | `query` |
| 「300033 最近 5 日因子」 | `rh_market_stock5d` | `code` |
| 「列出所有板块」 | `rh_market_block_list` | 无参；**本机板块目录优先这个** |
| 「新能源板块有哪些成分股？」 | `rh_market_block_stocks` | `block`: 板块名称（与 list 一致） |

### 5.1 问财：搜个股 vs 搜板块/指数

问财**没有**单独的「资产类型」参数，差别写在 `query` 话术里；列板块/成分优先 `block_*`。

| 目标 | 推荐说法 | 工具 |
|------|----------|------|
| 筛股票 | 「今日涨停；非ST」「连续两日涨停」「市盈率<20；市值>100亿」 | `wencai` / `wencai_detail` / `select_stocklist` |
| 要字段表 | 「问财找…，把明细字段也给我」 | `wencai_detail` |
| 列本机板块 | 「列出板块」「半导体板块成分」 | `block_list` → `block_stocks` |
| 用问财聊板块 | 「涨幅前十的概念板块」「今天强势行业」 | `wencai`（话术点明板块/指数） |

**示例对话：**

> 用户：问财找市盈率小于 20、市值大于 100 亿的票，把代码列出来。  
> AI：`rh_market_select_stocklist`，`query`: `沪深A股;市盈率<20;市值>100亿`。

> 用户：今天涨停有哪些？给我明细。  
> AI：`rh_market_wencai_detail`，`query`: `今日涨停`。（若要**历史某日**涨停池，改走第五节云端 `rh_fuyao_limit_up_pool`。）

> 用户：半导体板块有哪些成分？别用问财瞎猜名字。  
> AI：`rh_market_block_list` 确认名称 → `rh_market_block_stocks`，`block`: `半导体`（以 list 返回名为准）。

> 用户：603919 现在价多少？再用问财搜一下同题材。  
> AI：先 `rh_market_quote`（单票）→ 再 `rh_market_wencai` 用题材话术；**禁止**只用问财当现价。

> 用户：看下 603919 最近 20 根日 K 和五档。  
> AI：`rh_market_kline`（`period`: `1440`，`length`: `20`）+ `rh_market_depth`。

**订阅类（高级，日常分析可不走）：** `reg_quote` / `reg_kline` → 循环 `wait_update` → `unreg_*`。

---

## 五点五、同花顺云端自选股 / 自选板块 / 动态板块

> 需 GUI「自选」页粘贴 Cookie 并保存（会重启 MCP）。不依赖本机 hexin。代码：`600519.SH` 或 6 位代码。  
> **自选股**走 `t.10jqka.com.cn`；**自选板块 / 动态板块**走 `ugc.10jqka.com.cn` selfgroup（`types=0,1`），以 `type` 与 `attrs.question` 区分。

| 你可以对 AI 说 | MCP 工具 | 参数要点 |
|----------------|----------|----------|
| 「列出自选股」 | `rh_ths_watch_list` | 返回「我的自选」；`group` 可省略 |
| 「把 600519 加到自选股」 | `rh_ths_watch_add` | `group`=`我的自选`（可省略）；`code`=`600519.SH` |
| 「从自选股删掉 600519」 | `rh_ths_watch_delete` | 同上 |
| 「列出自选板块」 | `rh_ths_favorite_list` | 可选 `group` 只看一个板块（如 `买点`） |
| 「把 000001 加到买点板块」 | `rh_ths_favorite_add` | `group`=`买点`（type=0 手动自选板块） |
| 「有哪些动态板块？」 | `rh_ths_block_list` | type=1 且含问句，如 `5日线`、`今日首板` |
| 「看看 5日线 动态板块有哪些票」 | `rh_ths_block_list` | `group`=`5日线` |

**示例对话：**

> 用户：看看我同花顺自选股和「买点」自选板块里有哪些票，把 600519.SH 加进买点。  
> AI：`rh_ths_watch_list` → `rh_ths_favorite_list`（`group`: `买点`）→ `rh_ths_favorite_add`（`group`: `买点`，`code`: `600519.SH`）。

---

## 六、云端研究数据（同花顺金融数据服务 · `rh_fuyao_*`）

> 需 GUI「API」Token。成功时业务在返回 JSON 的 `data`；另可能有 `item_count` / `pagination` / `hint`。  
> 完整映射：[fuyao-routing.md](./fuyao-routing.md)。工作台 Skill：`rhths-fuyao`。

### 6.1 准备与消歧

| 你可以对 AI 说 | MCP 工具 | 参数要点 |
|----------------|----------|----------|
| 「官方 Key 通不通？」 | `rh_fuyao_ping` | 无参 |
| 「贵州茅台的 thscode？」 | `rh_fuyao_meta_search` | `q` 必填；可选 `asset_type` |
| 「列出 A 股代码表前一页」 | `rh_fuyao_meta_list` | `asset_type`=`a-share`；`limit`/`offset` |
| 「近一年有哪些交易日？」 | `rh_fuyao_calendar_trading_days` | 无参（固定近一年，不可自定义十年） |

**示例对话：**

> 用户：帮我确认「凯莱英」的标准代码，再查估值。  
> AI：`rh_fuyao_meta_search`（`q`: `凯莱英`）→ 得到 `002821.SZ` → `rh_fuyao_valuations_snapshot`（`thscodes`: `002821.SZ`）。  
> 现价：`rh_market_quote`（`code`: `002821`）。

> 用户：搜一下名字带「半导体」的基金。  
> AI：`rh_fuyao_meta_search`，`q`: `半导体`，`asset_type` 可试 `fund-otc,fund-etf`；再按结果选 `rh_fuyao_fund_*`。

### 6.2 财务 · 估值 · 复权（单票研究）

| 你可以对 AI 说 | MCP 工具 | 参数要点 |
|----------------|----------|----------|
| 「600519 估值 PE/PB」 | `rh_fuyao_valuations_snapshot` | `thscodes` 逗号串，最多约 100；**仅最新** |
| 「茅台最近利润表」 | `rh_fuyao_financials_income` | `thscode`；`period`=`annual`/`quarterly` |
| 「资产负债表 / 现金流量表」 | `_balance` / `_cashflow` | 同上 |
| 「2024 年三季报财务指标」 | `rh_fuyao_financials_indicators` | `report`=`2024-3`（`yyyy-{1\|2\|3\|4}`） |
| 「茅台 2020 至今分红送转」 | `rh_fuyao_adjustment_factors` | `from`/`to`=`YYYY-MM-DD`（事件流，非每日因子表） |

**示例对话：**

> 用户：帮我查贵州茅台当前估值和最近几期利润表。  
> AI：`meta_search` → `valuations_snapshot` + `financials_income`；现价用 `rh_market_quote`。

> 用户：对比 600519 和 000858 的 PE/PB。  
> AI：一次 `rh_fuyao_valuations_snapshot`，`thscodes`: `600519.SH,000858.SZ`。

> 用户：给 600519 算一下 2023 年年报关键的成长/盈利类指标。  
> AI：`rh_fuyao_financials_indicators`，`thscode`: `600519.SH`，`report`: `2023-4`。（无行业排名/点评。）

### 6.3 特色数据：涨停 · 跌停 · 炸板 · 连板 · 异动 · 热榜 · 龙虎榜 · 竞价

| 你可以对 AI 说 | MCP 工具 | 参数要点 |
|----------------|----------|----------|
| 「今天涨停池」 | `rh_fuyao_limit_up_pool` | 省略日期=服务器今日；周末常为空 |
| 「上周五涨停有哪些？」 | `rh_fuyao_limit_up_pool` | **`trade_date`=`YYYY-MM-DD`**（或 `date`/`date_ms`）；默认尽量一页 size=200 |
| 「今天跌停池 / 炸板池」 | `rh_fuyao_limit_down_pool` / `limit_break_pool` | 日期别名同涨停池；炸板看 `open_times`（开板次数≠连板） |
| 「近 30 日连板天梯」 | `rh_fuyao_limit_up_ladder` | **无日期入参**；从返回 `item[].date` 取目标日 |
| 「上周五连板高度谁最高？」 | `ladder` + 取对应 `date` | 勿给 ladder 传 `trade_date` |
| 「这几只集合竞价」 | `rh_fuyao_auction_snapshot` | `thscodes` 必填≤100；`stage=live/final` |
| 「今天竞价强弱基准」 | `rh_fuyao_auction_benchmark` | `date`=`YYYY-MM-DD`；省略=上海当日 |
| 「今天全市场异动」 | `rh_fuyao_anomaly_list` | **仅当日**；历史勿用 |
| 「这几只今天异动原因」 | `rh_fuyao_anomaly_stock` | `thscodes`=`600519.SH,000001.SZ` |
| 「上周五异动/涨停原因」 | **涨停池** `limit_up_reason` | **禁止**对历史日调 `anomaly_*` |
| 「当前飙升榜 / 热股榜」 | `skyrocket_list` / `hot_stock_list` | `period`=`day`/`hour`；非历史 |
| 「上周五热股榜」 | `rh_fuyao_hot_stock_history` | `date`=`YYYY-MM-DD` |
| 「某票热榜走势」 | `rh_fuyao_hot_stock_trend` | `thscode`；`start_date`/`end_date` |
| 「今天龙虎榜机构」 | `rh_fuyao_dragon_tiger` | `board_type`=`org`；可选 `date` |

**示例对话：**

> 用户：上周五的涨停池、连板、个股异动帮我整理成 md。  
> AI：  
> 1. `rh_fuyao_limit_up_pool`，`trade_date`: `YYYY-MM-DD`（上周五）  
> 2. `rh_fuyao_limit_up_ladder`，从矩阵取该日 boards  
> 3. 「异动/原因」用池内 `limit_up_reason`（**不调** `anomaly_*`）  
> 4. 用 `Write` 写入会话 `out/` 整理稿  

> 用户：今天涨停几家？按连板高度说说。  
> AI：`limit_up_pool`（今日）+ 可选 `limit_up_ladder` 看当日梯队；汇总 `continue_day_cnt`。

> 用户：给我看下 2026-08-07 的热股历史榜，不要用当前榜冒充。  
> AI：`rh_fuyao_hot_stock_history`，`date`: `2026-08-07`。

> 用户：今天龙虎榜游资席位。  
> AI：`rh_fuyao_dragon_tiger`，`board_type`: `hot_money`。

### 6.4 指数 / 概念板块（云端）

| 你可以对 AI 说 | MCP 工具 | 参数要点 |
|----------------|----------|----------|
| 「同花顺有哪些概念板块？」 | `rh_fuyao_index_catalog` | `tag`=`cn_concept`（另有 `industry`/`region`/`tszs`）；结果大宜落盘 |
| 「某概念成分股」 | `rh_fuyao_index_constituents` | `thscode` 如 `886042.TI` |
| 「这些指数最新行情」 | `rh_fuyao_index_prices_snapshot` | `thscodes` |
| 「沪深300 近一年日 K」 | `rh_fuyao_index_prices_historical` | `thscode`=`000300.SH`；`start`/`end` 日期或毫秒 |

**示例对话：**

> 用户：找「商业航天」相关同花顺概念代码，再拉成分，对前 5 只查本机现价。  
> AI：`index_catalog`（`tag`: `cn_concept`）匹配名称 → `index_constituents` → 有限只数 `rh_market_quote` / `rh_market_price`。  
> （本机板块名也可用 `rh_market_block_*`，与云端 `.TI` 概念是两套目录。）

### 6.5 基金

| 你可以对 AI 说 | MCP 工具 | 参数要点 |
|----------------|----------|----------|
| 「查某基金资料」 | `rh_fuyao_fund_profile` | **`fund_type`**=`otc`/`exchange`/`reits` + `thscode` |
| 「基金重仓股」 | `rh_fuyao_fund_holdings` | 同上；披露口径非实时 |
| 「近一年净值」 | `rh_fuyao_fund_nav` | 用 **`range`**=`year` 等，勿乱填自定义起止 |
| 「区间收益」 | `rh_fuyao_fund_returns` | `fund_type` + `thscode` |
| 「持有人结构」 | `rh_fuyao_fund_holders` | 可选 `merge_scope` |
| 「ETF 场内价」 | `rh_fuyao_fund_market_snapshot` | ETF/LOF；勿对场外 |
| 「ETF 日线」 | `rh_fuyao_fund_market_historical` | **仅 ETF** |
| 「基金公司 / 经理」 | `fund_company` / `fund_manager_*` | `company_id` / `manager_id` 来自 profile |
| 「行业配置 / 资产配置 / 回撤」 | `industry_allocation` / `asset_allocation` / `drawdowns` | `fund_type`+`thscode` |
| 「历史持仓报告期」 | `fund_stock_report_dates` / `bond_report_dates` | 先发现报告期，再 `*_history` |
| 「在售基金 / 资讯」 | `fund_offerings` / `fund_news` | `subscribe=active/upcoming`；资讯 `offset` 为游标 |

**示例对话：**

> 用户：510300 这只 ETF 的资料和近一年净值。  
> AI：`meta_search` 确认 `thscode` 与类型 → `fund_profile`（`fund_type`: `exchange`）→ `fund_nav`（`range`: `year`）→ 场内价可用 `fund_market_snapshot` 或本机 `rh_market_quote`。

> 用户：某场外基金重仓股。  
> AI：确认 `fund_type`=`otc` 后 `rh_fuyao_fund_holdings`；说明为定期披露。

> 用户：查这只基金经理履历和风格。  
> AI：`fund_profile` 取 `manager_info[].manager_id` → `fund_manager_detail` + `fund_manager_style`。

### 6.6 云端避错（用户也可提醒 AI）

1. 只有名称 → 先 `meta_search`，带齐 `.SH/.SZ/.TI/.OF` 后缀。  
2. 历史涨停/跌停/炸板必须带 `trade_date`；周末空池 ≠ 接口坏了。  
3. 异动/当前热榜 **无历史**；历史原因用涨停池字段。  
4. 连板天梯不传日期。  
5. 竞价快照须带 `thscodes`；基准 `date` 非交易日不自动回退。  
6. 实时价优先本机 `rh_market_*`。  
7. 大列表让 AI 落盘，对话只留摘要。

---

## 七、技术指标

| 你可以对 AI 说 | MCP 工具 | 说明 / 常用参数 |
|----------------|----------|-----------------|
| 「支持哪些技术指标？」 | `rh_indicator_list` | 返回可用指标名 |
| 「算一下 603919 的 MACD」 | `rh_indicator_calc` | `code`, `indicator`: `MACD`；日线 `period`: `1440` |
| 「600000 日线 RSI」 | `rh_indicator_calc` | 可选 `length`: `100` |

**示例对话：**

> 用户：603919 的 MACD 金叉了吗？帮我算 MACD 并简单解读。  
> AI：`rh_indicator_calc`（`indicator`: `MACD`），结合返回说明（不构成投资建议）。

---

## 八、THS 进程（system）

| 你可以对 AI 说 | MCP 工具 | 说明 |
|----------------|----------|------|
| 「hexin 和 xiadan 在不在跑？」 | `rh_system_status` | 只读 |
| 「启动 THS 主程序」 | `rh_system_start_hexin` | 已在跑则跳过 |
| 「停止 hexin」 | `rh_system_stop_hexin` | |
| 「启动下单程序 xiadan」 | `rh_system_start_xiadan` | 路径来自 GUI |
| 「停止 xiadan」 | `rh_system_stop_xiadan` | |

> 路径在 `%AppData%\RHTHS\settings.json`。**停止 xiadan 后再部署扩展**，部署完需重启 xiadan。

**示例对话：**

> 用户：帮我启动 xiadan，然后检查网关是否就绪。  
> AI：`rh_system_start_xiadan` → `rh_trade_health`。

> 用户：收盘后帮我停掉 hexin 和 xiadan。  
> AI：`rh_system_stop_xiadan` → `rh_system_stop_hexin` → `rh_system_status`。

---

## 九、条件单与 py/call

| 你可以对 AI 说 | MCP 工具 | 说明 / 常用参数 |
|----------------|----------|-----------------|
| 「条件单字段和支持的路由有哪些？」 | `rh_condition_schema` | 只读 schema |
| 「给 000001 加一个条件单测试」 | `rh_condition_add` | `codelist`, `condition`, `action_code` |
| 「恢复 / 暂停 / 删除条件单」 | `resume` / `pause` / `delete` | `signal_id` |
| 「提交 Pine/UI 信号单 JSON」 | `rh_condition_run_json` | `payload` |
| 「用 py/call 恢复策略」 | `rh_py_call` | `action`: `condition.resume` |

**示例对话：**

> 用户：给 600000 加一个简单条件：价格大于 0 就 pass，看看 signal_id。  
> AI：`rh_condition_add`，`codelist`: `600000`，`condition`: `quote[code]['price'] > 0`，`action_code`: `pass`。

> 用户：把 signal_id 20260524_00000001 的策略恢复运行。  
> AI：`rh_condition_resume`，或 `rh_py_call` + `condition.resume`。

---

## 十、自动交易审核（本地 JSON）

| 你可以对 AI 说 | MCP 工具 | 说明 / 常用参数 |
|----------------|----------|-----------------|
| 「自动交易审核配置是什么？」 | `rh_autotrading_config_get` | 只读 |
| 「改成人工审核 / 自动执行」 | `rh_autotrading_config_set` | `execution_mode`: `review` / `auto` |
| 「有哪些待审？」 | `rh_autotrading_review_list` | `status`: `pending` |
| 「通过 / 拒绝某条」 | `approve` / `reject` | `review_id`，可选 `note` |
| 「看审计日志」 | `rh_autotrading_audit_logs` | `limit` |

> 数据在 `PythonLog\rhths\data\`。**无 MySQL**；与 thsQuant `:19090` 无关。

---

## 十一、组合场景（推荐话术）

### 11.1 开盘前检查

> 「检查 RHTHS 健康 → hexin/xiadan → 资金与持仓 → 今日委托，全部只读。」

`rh_trade_health` → `rh_system_status` → `rh_trade_account` → `rh_trade_positions` → `rh_trade_orders_today`

### 11.2 选股 + 模拟下单

> 「问财选 5 只低市盈率股票，对第一只拉 stock5d 和 MACD，再模拟买入 100 股最新价。」

`rh_market_select_stocklist` → `rh_market_stock5d` + `rh_indicator_calc` → `rh_trade_buy`（`dry_run`: `true`）

### 11.3 持仓复盘

> 「列出持仓，对每只拉日 K 和 MACD，用文字总结。」

`rh_trade_positions` → 每只 `rh_market_kline` + `rh_indicator_calc`

### 11.4 上周五涨停结构复盘（云端）

> 「整理上周五涨停池、连板高度、涨停原因，写成 md；不要调历史异动接口。」

`rh_fuyao_limit_up_pool(trade_date=…)` → `rh_fuyao_limit_up_ladder` → 用 `limit_up_reason` → `Write` 到 `out/`

### 11.5 单票基本面快照（本机价 + 云端财报）

> 「茅台：现价、估值、最近利润表、近一年分红事件。」

`rh_market_quote` + `rh_fuyao_meta_search` → `valuations_snapshot` + `financials_income` + `adjustment_factors`

### 11.6 概念板块 → 成分 → 本机行情

> 「找商业航天相关同花顺概念，拉成分，对前 10 只查现价。」

`rh_fuyao_index_catalog` → `index_constituents` → `rh_market_price`（有限只数）

### 11.7 ETF 研究

> 「510300：确认代码 → 资料 → 近一年净值 → 本机现价。」

`meta_search` → `fund_profile` + `fund_nav(range=year)` → `rh_market_quote`

### 11.8 委托清理（模拟）

> 「撤销今天所有未成交委托，先模拟。」

`rh_trade_orders_today` → `rh_trade_cancel_all`（`dry_run`: `true`）

### 11.9 自动交易审核流

> 「看待审列表 → 说明方向 → 我确认后通过或拒绝。」

`rh_autotrading_review_list` → approve/reject → `audit_logs`

### 11.10 本机板块 vs 云端概念

> 「先列本机板块里有没有『半导体』；若没有，再查同花顺云端概念目录。」

`rh_market_block_list` / `block_stocks`；不足再 `rh_fuyao_index_catalog(tag=cn_concept)`

---

## 十二、工具速查（全部 MCP）

| 分类 | 工具名 |
|------|--------|
| 系统 | `rh_trade_health`、`rh_trade_catalog`、`rh_system_*` |
| 模式 | `rh_trade_mode_get`、`rh_trade_mode_set` |
| 交易查询 | `rh_trade_users`、`account`、`account_daily`、`positions`、`orders_today`、`orders_history`、`warmup` |
| 交易写入 | `buy`、`sell`、`cancel`、`cancel_all`、`invoke` |
| 本机行情 | `rh_market_quote`、`price`、`kline`、`kline_sec`、`kline_15s`、`timeshare`、`depth`、`stock5d`、`wencai*`、`select_stocklist`、`block_*`、`reg_*`/`wait`/`unreg_*` |
| 指标 | `rh_indicator_list`、`rh_indicator_calc` |
| 条件单 | `rh_condition_*`、`rh_py_call` |
| 自动交易 | `rh_autotrading_*` |
| **云端 fuyao** | `rh_fuyao_ping`、`meta_*`、`prices_*`、`adjustment_factors`、`financials_*`、`valuations_snapshot`、`calendar_trading_days`、`auction_*`、`limit_up_*`、`limit_down_pool`、`limit_break_pool`、`anomaly_*`、`skyrocket_list`、`hot_stock_*`、`dragon_tiger`、`index_*`、`fund_*`、`rh_fuyao_get` |
| **云端自选股/自选板块/动态板块** | `rh_ths_watch_*`；`rh_ths_favorite_*`；`rh_ths_block_*` |

机读摘要：[mcp.tools.json](./mcp.tools.json) · 云端路由：[fuyao-routing.md](./fuyao-routing.md)

---

## 十三、常见错误

| 现象 | 处理 |
|------|------|
| 工具调用失败 / 连接错误 | THS 是否已登录；`rh_trade_health`；扩展是否已部署 |
| `rh_system_*` 找不到 exe | GUI 配置 hexin / xiadan 路径 |
| `LIVE_BLOCKED` | 未设 `RHTHS_ALLOW_LIVE=1` 或未传 `confirm_live` |
| `FREE_DAILY_LIMIT` | 标准版当日实盘次数已满 |
| 问财/行情为空 | THS 行情是否可用；话术是否合法；单票改用 `quote` |
| 用问财查历史涨停 | 改 `rh_fuyao_limit_up_pool(trade_date=…)` |
| 涨停池周末为空 | 传具体交易日；或属非交易日正常空 |
| 异动历史为空 / `skipped` | 异动仅当日；用涨停池 `limit_up_reason` |
| `FUYAO_ERROR` / Key 无效 | GUI「API」保存 Token；`rh_fuyao_ping` |
| `THS_UGC_ERROR` / Cookie 未配置 | GUI「自选」粘贴 Cookie 并保存（会重启 MCP） |
| 基金报类型错误 | `meta_search` 后设对 `fund_type` |
| `account_daily` 无数据 | 日期须 **`YYYYMMDD`** |
| autotrading 列表为空 | 看 `execution_mode`；数据在 `PythonLog\rhths\data\` |

更多配置与排障：[MCP使用说明.md](./MCP使用说明.md) · [快速开始.md](./快速开始.md) · [SKILL.md](./SKILL.md)
