# AI 交易示例（OpenClaw · Hermes Agent）

在 **OpenClaw** 或 **Hermes Agent** 中配置 MCP 服务 **`rhths-trade`** 后，用自然语言即可驱动**本机已登录**的同花顺账户。

- **默认模拟下单**（`dry_run` / 模式 `simulate`），不会真实成交。  
- **实盘**须环境变量 `RHTHS_ALLOW_LIVE=1`，且每笔写操作带 **`confirm_live: true`**，风险自负。  
- 配置方法见 [MCP使用说明.md](./MCP使用说明.md)、[SKILL.md](./SKILL.md)。

---

## 使用前（建议先说给 AI）

| 你可以对 AI 说 | MCP 工具 |
|----------------|----------|
| 「先检查 RHTHS 和同花顺是否正常」 | `rh_trade_health` |
| 「现在是模拟还是实盘模式？」 | `rh_trade_mode_get` |

---

## 一、系统与模式

| 你可以对 AI 说 | MCP 工具 | 说明 / 常用参数 |
|----------------|----------|-----------------|
| 「网关和同花顺 API 就绪吗？」 | `rh_trade_health` | 返回网关版本、`ths_api` 是否可用 |
| 「列出网关支持哪些接口」 | `rh_trade_catalog` | 查看 HTTP 路由列表 |
| 「当前是模拟还是实盘？」 | `rh_trade_mode_get` | 与 GUI 共用 `%APPDATA%\RHTHS\settings.json` |
| 「切换到模拟模式」 | `rh_trade_mode_set` | `mode`: `simulate` |
| 「切换到实盘模式」（需谨慎） | `rh_trade_mode_set` | `mode`: `live`；实盘下单仍需 `confirm_live` |

**示例对话：**

> 用户：帮我看一下 RHTHS 是否正常，并告诉我现在是模拟还是实盘。  
> AI：调用 `rh_trade_health` → `rh_trade_mode_get`，用自然语言汇总结果。

---

## 二、账户与交易查询（只读）

| 你可以对 AI 说 | MCP 工具 | 说明 / 常用参数 |
|----------------|----------|-----------------|
| 「我有哪些资金账号？」 | `rh_trade_users` | 多账号时列出；可选 `user`、`rzrq`（融资融券） |
| 「查一下资金」 | `rh_trade_account` | 可用资金、总资产等 |
| 「帮我看下资金和持仓」 | `rh_trade_account` + `rh_trade_positions` | 常组合使用 |
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

---

## 三、下单与撤单（写操作 · 默认模拟）

| 你可以对 AI 说 | MCP 工具 | 说明 / 常用参数 |
|----------------|----------|-----------------|
| 「模拟买入 100 股 600000，最新价」 | `rh_trade_buy` | `code`, `qty`, `price`: `zxjg`，`dry_run`: `true` |
| 「模拟卖出 603919 100 股，对手价三档」 | `rh_trade_sell` | `price`: 常用 `dsj3`（卖） |
| 「撤销今天所有未成交委托（模拟）」 | `rh_trade_cancel` | `scope`: `all`，`dry_run`: `true` |
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

---

## 四、实盘（高风险 · 须双重确认）

| 你可以对 AI 说 | MCP 工具 | 必要条件 |
|----------------|----------|----------|
| 「实盘卖出 100 股 603919，我已确认风险」 | `rh_trade_sell` | 交易机 `RHTHS_ALLOW_LIVE=1`，`dry_run`: `false`，`confirm_live`: `true` |
| 「实盘买入…」 | `rh_trade_buy` | 同上 |
| 「实盘撤销某笔委托」 | `rh_trade_cancel` | 同上 |

> **标准版**：实盘买入+卖出合计每天最多 **10 次**；超限返回 `FREE_DAILY_LIMIT`。高级版在 GUI 激活后无此限制。

**示例对话：**

> 用户：我已在 GUI 打开实盘并确认风险，请实盘卖出 100 股 603919，对手价三档。  
> AI：确认环境后 `rh_trade_sell`，`dry_run`: `false`，`confirm_live`: `true`。

---

## 五、行情与问财

| 你可以对 AI 说 | MCP 工具 | 说明 / 常用参数 |
|----------------|----------|-----------------|
| 「603919 现在什么价？」 | `rh_market_quote` | `code`: 6 位代码 |
| 「603919 和 300033 现价多少？」 | `rh_market_price` | `codes`: 数组或多次 `code` |
| 「拉 600000 最近 30 根日 K」 | `rh_market_kline` | `period`: `1440`（日线），`length`: `30` |
| 「看 603919 的 15 秒 K 线」 | `rh_market_kline_sec` | `sec`: `15` 或 `30`，`length`: 条数 |
| 「问财：市盈率小于 20 的沪深 A 股」 | `rh_market_wencai` | `query` 或 `q`: 问财语句 |
| 「问财选市盈率小于 20 的票，只要代码列表」 | `rh_market_select_stocklist` | `query`: 问财语句，返回代码列表 |
| 「订阅 603919、300033 行情推送」 | `rh_market_reg_quote` | `codes` 或 `code` |
| 「订阅 600000 日 K 推送」 | `rh_market_reg_kline` | `code`, `period`, `length` |
| 「等一次行情推送更新」 | `rh_market_wait_update` | `timeout`: 秒，默认 `0.5` |
| 「取消 603919 行情订阅」 | `rh_market_unreg_quote` | `code` 或 `codes` |
| 「取消 600000 K 线订阅」 | `rh_market_unreg_kline` | `code` |

**订阅类典型流程：**

1. `rh_market_reg_quote` 或 `rh_market_reg_kline`  
2. 循环 `rh_market_wait_update` 获取推送  
3. 结束用 `rh_market_unreg_quote` / `rh_market_unreg_kline`

**示例对话：**

> 用户：问财找市盈率小于 20、市值大于 100 亿的票，把代码列出来。  
> AI：`rh_market_select_stocklist`，`query`: `沪深A股;市盈率<20;市值>100亿`。

> 用户：看下 603919 最近 20 根日 K 大概走势。  
> AI：`rh_market_kline`，`code`: `603919`，`period`: `1440`，`length`: `20`。

---

## 六、技术指标

| 你可以对 AI 说 | MCP 工具 | 说明 / 常用参数 |
|----------------|----------|-----------------|
| 「支持哪些技术指标？」 | `rh_indicator_list` | 返回可用指标名 |
| 「算一下 603919 的 MACD」 | `rh_indicator_calc` | `code`, `indicator`: `MACD` |
| 「600000 日线 RSI」 | `rh_indicator_calc` | 可选 `period`: `1440`，`length`: `100` |

**示例对话：**

> 用户：603919 的 MACD 金叉了吗？帮我算 MACD 并简单解读。  
> AI：`rh_indicator_calc`（`indicator`: `MACD`），结合返回数据说明（不构成投资建议）。

---

## 七、组合场景（推荐话术）

### 7.1 开盘前检查

> 「检查 RHTHS 健康 → 看资金与持仓 → 看今日已有委托，全部用只读接口。」

工具链：`rh_trade_health` → `rh_trade_account` → `rh_trade_positions` → `rh_trade_orders_today`

### 7.2 选股 + 模拟下单

> 「问财选 5 只低市盈率股票，选第一只模拟买入 100 股最新价。」

工具链：`rh_market_select_stocklist` → `rh_trade_buy`（`dry_run`: `true`）

### 7.3 持仓复盘

> 「列出持仓，对每只拉日 K 和 MACD，用文字总结。」

工具链：`rh_trade_positions` → 对每只 `rh_market_kline` + `rh_indicator_calc`

### 7.4 委托清理（模拟）

> 「撤销今天所有未成交委托，先模拟。」

工具链：`rh_trade_orders_today` → `rh_trade_cancel`（`scope`: `all`，`dry_run`: `true`）

---

## 八、工具速查（全部 MCP）

| 分类 | 工具名 |
|------|--------|
| 系统 | `rh_trade_health`、`rh_trade_catalog` |
| 模式 | `rh_trade_mode_get`、`rh_trade_mode_set` |
| 交易查询 | `rh_trade_users`、`rh_trade_account`、`rh_trade_positions`、`rh_trade_orders_today`、`rh_trade_orders_history`、`rh_trade_warmup` |
| 交易写入 | `rh_trade_buy`、`rh_trade_sell`、`rh_trade_cancel`、`rh_trade_invoke` |
| 行情 | `rh_market_quote`、`rh_market_price`、`rh_market_kline`、`rh_market_kline_sec`、`rh_market_wencai`、`rh_market_select_stocklist` |
| 订阅 | `rh_market_reg_quote`、`rh_market_reg_kline`、`rh_market_wait_update`、`rh_market_unreg_quote`、`rh_market_unreg_kline` |
| 指标 | `rh_indicator_list`、`rh_indicator_calc` |

机读参数摘要：[mcp.tools.json](./mcp.tools.json)

---

## 九、常见错误

| 现象 | 处理 |
|------|------|
| 工具调用失败 / 连接错误 | 同花顺是否已登录；`rh_trade_health`；Hook 是否部署 |
| `LIVE_BLOCKED` | 未设 `RHTHS_ALLOW_LIVE=1` 或未传 `confirm_live` |
| `FREE_DAILY_LIMIT` | 标准版当日实盘次数已满，改模拟或升级高级版 |
| 问财/行情为空 | 同花顺行情模块是否可用；语句是否合法 |

更多配置与排障：[MCP使用说明.md](./MCP使用说明.md) · [快速开始.md](./快速开始.md)
