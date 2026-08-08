# AI 交易示例（OpenClaw · Hermes Agent）

在 **OpenClaw** 或 **Hermes Agent** 中配置 MCP 服务 **`rhths-trade`** 后，用自然语言即可驱动**本机已登录**的 THS 账户。

- **默认模拟下单**（`dry_run` / 模式 `simulate`），不会真实成交。 
- **实盘**须环境变量 `RHTHS_ALLOW_LIVE=1`，且每笔写操作带 **`confirm_live: true`**，风险自负。 
- 配置方法见 [MCP使用说明.md](./MCP使用说明.md)、[SKILL.md](./SKILL.md)。

---

## 使用前（建议先说给 AI）

| 你可以对 AI 说 | MCP 工具 |
|----------------|----------|
| 「先检查 RHTHS 和 THS 是否正常」 | `rh_trade_health` |
| 「hexin 和 xiadan 在跑吗？」 | `rh_system_status` |
| 「现在是模拟还是实盘模式？」 | `rh_trade_mode_get` |

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

> 用户： THS 主程序和下单程序都在跑吗？ 
> AI：`rh_system_status`，说明 hexin / xiadan 运行状态及 exe 路径。

---

## 二、账户与交易查询（只读）

| 你可以对 AI 说 | MCP 工具 | 说明 / 常用参数 |
|----------------|----------|-----------------|
| 「我有哪些资金账号？」 | `rh_trade_users` | 多账号时列出；可选 `user`、`rzrq`（融资融券） |
| 「查一下资金」 | `rh_trade_account` | 可用资金、总资产等（默认会写当日快照） |
| 「帮我看下资金和持仓」 | `rh_trade_account` + `rh_trade_positions` | 常组合使用 |
| 「查 1 月份每天的资金快照」 | `rh_trade_account_daily` | `start_date`/`end_date`: `YYYYMMDD` |
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
> AI：`rh_trade_account_daily`，`start_date`/`end_date` 设为近 7 日，汇总 `total_asset` 等字段。

---

## 三、下单与撤单（写操作 · 默认模拟）

| 你可以对 AI 说 | MCP 工具 | 说明 / 常用参数 |
|----------------|----------|-----------------|
| 「模拟买入 100 股 600000，最新价」 | `rh_trade_buy` | `code`, `qty`, `price`: `zxjg`，`dry_run`: `true` |
| 「模拟卖出 603919 100 股，对手价三档」 | `rh_trade_sell` | `price`: 常用 `dsj3`（卖） |
| 「撤销今天所有未成交委托（模拟）」 | `rh_trade_cancel` | `scope`: `all`，`dry_run`: `true` |
| 「一键全部撤单（模拟）」 | `rh_trade_cancel_all` | `dry_run`: `true`（等价于 cancel scope=all） |
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

## 五、行情与问财

| 你可以对 AI 说 | MCP 工具 | 说明 / 常用参数 |
|----------------|----------|-----------------|
| 「603919 现在什么价？」 | `rh_market_quote` | `code`: 6 位代码 |
| 「603919 和 300033 现价多少？」 | `rh_market_price` | `codes`: 数组或多次 `code` |
| 「拉 600000 最近 30 根日 K」 | `rh_market_kline` | `period`: `1440`（日线），`length`: `30` |
| 「看 603919 的 15 秒 K 线」 | `rh_market_kline_sec` | `sec`: `15` 或 `30`，`length`: 条数 |
| 「拉 603919 的 15 秒 K（专用接口）」 | `rh_market_kline_15s` | `code`, `length`（默认 960 根） |
| 「这只票支持哪些 K 线周期？」 | `rh_market_kline_probe` | `code`, `length`（探测用） |
| 「603919 今天分时走势」 | `rh_market_timeshare` | `code` |
| 「603919 五档盘口」 | `rh_market_depth` | `code` |
| 「问财：市盈率小于 20 的沪深 A 股」 | `rh_market_wencai` | `query` 或 `q`: 问财语句 |
| 「问财同上，要完整字段明细」 | `rh_market_wencai_detail` | `query` |
| 「问财选市盈率小于 20 的票，只要代码列表」 | `rh_market_select_stocklist` | `query`: 问财语句 |
| 「300033 最近 5 日因子（OHLCV+MACD+资金流）」 | `rh_market_stock5d` | `code`, `kline_length` |
| 「stock5d 返回哪些字段？」 | `rh_market_stock5d_schema` | 无参数 |
| 「探测 603919 资金流接口是否可用」 | `rh_market_moneyflow_probe` | `code` |
| 「列出所有板块」 | `rh_market_block_list` | 无参数 |
| 「新能源板块有哪些成分股？」 | `rh_market_block_stocks` | `block_name` 或 `block_id` |
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

> 用户：300033 最近 5 个交易日的量价和 MACD 因子帮我拉一下。 
> AI：`rh_market_stock5d`，`code`: `300033`。

> 用户：603919 现在分时和五档盘口一起看下。 
> AI：`rh_market_timeshare` + `rh_market_depth`，`code`: `603919`。

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

## 七、 THS 进程（system）

| 你可以对 AI 说 | MCP 工具 | 说明 |
|----------------|----------|------|
| 「hexin 和 xiadan 在不在跑？」 | `rh_system_status` | 只读，返回进程状态与 exe 路径 |
| 「启动 THS 主程序」 | `rh_system_start_hexin` | 已在跑则跳过 |
| 「停止 hexin」 | `rh_system_stop_hexin` | 结束所有 hexin 进程 |
| 「启动下单程序 xiadan」 | `rh_system_start_xiadan` | 路径来自 GUI 设置 |
| 「停止 xiadan」 | `rh_system_stop_xiadan` | 结束所有 xiadan 进程 |

> 路径配置在 `%AppData%\RHTHS\settings.json`，GUI 可修改。**停止 xiadan 后再部署扩展**，部署完需重启 xiadan。

**示例对话：**

> 用户：帮我启动 xiadan，然后检查网关是否就绪。 
> AI：`rh_system_start_xiadan` → `rh_trade_health`，汇报 xiadan 与 `ths_api` 状态。

> 用户：收盘后帮我停掉 hexin 和 xiadan。 
> AI：`rh_system_stop_xiadan` → `rh_system_stop_hexin`，再 `rh_system_status` 确认。

---

## 八、条件单与 py/call

| 你可以对 AI 说 | MCP 工具 | 说明 / 常用参数 |
|----------------|----------|-----------------|
| 「条件单字段和支持的路由有哪些？」 | `rh_condition_schema` | 只读 schema |
| 「给 000001 加一个条件单测试」 | `rh_condition_add` | `codelist`, `condition`, `action_code` |
| 「恢复策略 / 条件单 resume」 | `rh_condition_resume` | `signal_id`（add 或 run_json 返回） |
| 「暂停条件单」 | `rh_condition_pause` | `signal_id` |
| 「删除条件单」 | `rh_condition_delete` | `signal_id` |
| 「提交 Pine/UI 信号单 JSON」 | `rh_condition_run_json` | `payload` 对象 |
| 「用 py/call 恢复策略」 | `rh_py_call` | `action`: `condition.resume`，`params`: `{signal_id}` |
| 「py/call 拉 stock5d」 | `rh_py_call` | `action`: `market.stock5d`，`params`: `{code}` |
| 「py/call 查持仓」 | `rh_py_call` | `action`: `trade.positions` |

**示例对话：**

> 用户：给 600000 加一个简单条件：价格大于 0 就 pass，看看返回的 signal_id。 
> AI：`rh_condition_add`，`codelist`: `600000`，`condition`: `quote[code]['price'] > 0`，`action_code`: `pass`。

> 用户：把 signal_id 20260524_00000001 的策略恢复运行。 
> AI：`rh_condition_resume`，`signal_id`: `20260524_00000001`。 
> 或等价：`rh_py_call`，`action`: `condition.resume`，`params`: `{"signal_id":"20260524_00000001"}`。

> 用户：用 py/call 查一下 300033 的 5 日因子。 
> AI：`rh_py_call`，`action`: `market.stock5d`，`params`: `{"code":"300033"}`。

---

## 九、自动交易审核（本地 JSON）

| 你可以对 AI 说 | MCP 工具 | 说明 / 常用参数 |
|----------------|----------|-----------------|
| 「自动交易审核配置是什么？」 | `rh_autotrading_config_get` | 只读 |
| 「改成人工审核模式」 | `rh_autotrading_config_set` | `execution_mode`: `review` |
| 「改成自动执行模式」（谨慎） | `rh_autotrading_config_set` | `execution_mode`: `auto` |
| 「有哪些待审的自动交易？」 | `rh_autotrading_review_list` | `status`: `pending`（默认） |
| 「通过 review_id xxx 的审核」 | `rh_autotrading_review_approve` | `review_id`, 可选 `note` |
| 「拒绝 review_id xxx」 | `rh_autotrading_review_reject` | `review_id`, 可选 `note` |
| 「看最近 100 条审计日志」 | `rh_autotrading_audit_logs` | `limit`: `100` |
| 「自动交易面板用的今日委托」 | `rh_autotrading_exchange_orders` | `limit` 可选 |

> 数据在 `PythonLog\rhths\data\` 本地 JSON / jsonl。**无 MySQL 工具**；与 thsQuant `:19090` 无连接。

**示例对话：**

> 用户：看一下自动交易是不是人工审核模式，并列出现有待审条目。 
> AI：`rh_autotrading_config_get` → `rh_autotrading_review_list`，`status`: `pending`。

> 用户：通过 review_id abc123，备注「已人工确认」。 
> AI：`rh_autotrading_review_approve`，`review_id`: `abc123`，`note`: `已人工确认`。

> 用户：今天自动交易相关的委托和审计记录帮我汇总。 
> AI：`rh_autotrading_exchange_orders` + `rh_autotrading_audit_logs`，文字归纳。

---

## 十、组合场景（推荐话术）

### 10.1 开盘前检查

> 「检查 RHTHS 健康 → hexin/xiadan 状态 → 看资金与持仓 → 看今日已有委托，全部只读。」

工具链：`rh_trade_health` → `rh_system_status` → `rh_trade_account` → `rh_trade_positions` → `rh_trade_orders_today`

### 10.2 选股 + 模拟下单

> 「问财选 5 只低市盈率股票，对第一只拉 stock5d 和 MACD，再模拟买入 100 股最新价。」

工具链：`rh_market_select_stocklist` → `rh_market_stock5d` + `rh_indicator_calc` → `rh_trade_buy`（`dry_run`: `true`）

### 10.3 持仓复盘

> 「列出持仓，对每只拉日 K 和 MACD，用文字总结。」

工具链：`rh_trade_positions` → 对每只 `rh_market_kline` + `rh_indicator_calc`

### 10.4 委托清理（模拟）

> 「撤销今天所有未成交委托，先模拟。」

工具链：`rh_trade_orders_today` → `rh_trade_cancel_all`（`dry_run`: `true`）

### 10.5 策略恢复

> 「列出 catalog 里 condition 路由 → 恢复 signal_id xxx → 再查 health。」

工具链：`rh_trade_catalog` → `rh_condition_resume` → `rh_trade_health`

### 10.6 自动交易审核流

> 「看待审列表 → 对每条说明买卖方向 → 我确认后通过或拒绝。」

工具链：`rh_autotrading_review_list` →（用户确认）→ `rh_autotrading_review_approve` / `rh_autotrading_review_reject` → `rh_autotrading_audit_logs`

---

## 十一、工具速查（全部 MCP）

| 分类 | 工具名 |
|------|--------|
| 系统 | `rh_trade_health`、`rh_trade_catalog`、`rh_system_status`、`rh_system_start_hexin`、`rh_system_stop_hexin`、`rh_system_start_xiadan`、`rh_system_stop_xiadan` |
| 模式 | `rh_trade_mode_get`、`rh_trade_mode_set` |
| 交易查询 | `rh_trade_users`、`rh_trade_account`、`rh_trade_account_daily`、`rh_trade_positions`、`rh_trade_orders_today`、`rh_trade_orders_history`、`rh_trade_warmup` |
| 交易写入 | `rh_trade_buy`、`rh_trade_sell`、`rh_trade_cancel`、`rh_trade_cancel_all`、`rh_trade_invoke` |
| 行情 | `rh_market_quote`、`rh_market_price`、`rh_market_kline`、`rh_market_kline_sec`、`rh_market_kline_15s`、`rh_market_kline_probe`、`rh_market_timeshare`、`rh_market_depth`、`rh_market_wencai`、`rh_market_wencai_detail`、`rh_market_select_stocklist`、`rh_market_stock5d`、`rh_market_stock5d_schema`、`rh_market_moneyflow_probe`、`rh_market_block_list`、`rh_market_block_stocks` |
| 订阅 | `rh_market_reg_quote`、`rh_market_reg_kline`、`rh_market_wait_update`、`rh_market_unreg_quote`、`rh_market_unreg_kline` |
| 条件单 | `rh_condition_schema`、`rh_condition_add`、`rh_condition_pause`、`rh_condition_resume`、`rh_condition_delete`、`rh_condition_run_json`、`rh_py_call` |
| 自动交易审核 | `rh_autotrading_config_get`、`rh_autotrading_config_set`、`rh_autotrading_review_list`、`rh_autotrading_review_approve`、`rh_autotrading_review_reject`、`rh_autotrading_audit_logs`、`rh_autotrading_exchange_orders` |
| 指标 | `rh_indicator_list`、`rh_indicator_calc` |

机读参数摘要：[mcp.tools.json](./mcp.tools.json)

---

## 十二、常见错误

| 现象 | 处理 |
|------|------|
| 工具调用失败 / 连接错误 | THS 是否已登录；`rh_trade_health`；扩展是否已部署 |
| `rh_system_*` 找不到 exe | GUI 设置里配置 hexin / xiadan 路径 |
| 条件单无 `signal_id` | 先 `rh_condition_add` 或 `run_json`，从返回里取 id |
| `LIVE_BLOCKED` | 未设 `RHTHS_ALLOW_LIVE=1` 或未传 `confirm_live` |
| `FREE_DAILY_LIMIT` | 标准版当日实盘次数已满，改模拟或升级高级版 |
| 问财/行情为空 | THS 行情模块是否可用；语句是否合法 |
| autotrading 列表为空 | 确认 `execution_mode`；数据在 `PythonLog\rhths\data\` |

更多配置与排障：[MCP使用说明.md](./MCP使用说明.md) · [快速开始.md](./快速开始.md)
