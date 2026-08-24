# RHTHS · `rh_fuyao_*` 路由与用法

> **受众：** RHTHS 安装包 / OpenClaw / Hermes 等**产品侧**文档。  
> **工作台：** 用 Skill `rhths-fuyao`（`~/.cursor-local-assistant-v2/pool_agent/skills/rhths-fuyao/`），勿读本文件或开发仓 `Financial-API/`。
>
> 对照官方 MCP capability-map（meta / a-share / index / fund / special-data；官方快照约 55 工具）的 RHTHS 映射。  
> 对齐上游 **2026.08.17.1**：集合竞价、跌停/炸板池、公募基金扩展。  
> 实时行情/交易优先本地 `rh_market_*` / `rh_trade_*`。需 GUI「API」Token。

## 与官方 MCP 的关系

官方 4 个托管服务 → 本机统一 `CallMcpTool(server="rhths-trade", tool_name=rh_fuyao_*)`。  
成功时业务在 `data`（已解包）；另有顶层 `item_count` / `pagination` / `hint` / `skipped`。

## 意图 → 工具（全量速查）

| 用户目标 | RHTHS 工具 | 关键参数 |
| --- | --- | --- |
| 探测 Token | `rh_fuyao_ping` | 无 |
| 消歧 | `rh_fuyao_meta_search` | `q`；基金可加 `asset_type=fund-otc,fund-etf,…` |
| 代码表 | `rh_fuyao_meta_list` | `limit`/`offset` |
| A 股快照/日 K | `rh_fuyao_prices_*` | **优先本地**；日 K 的 start/end 支持日期 |
| 复权事件 | `rh_fuyao_adjustment_factors` | `from`/`to` |
| 三表+指标 | `rh_fuyao_financials_*` | `report` 格式专用 |
| 估值 | `rh_fuyao_valuations_snapshot` | `thscodes` |
| 交易日历 | `rh_fuyao_calendar_trading_days` | 无参、近一年 |
| 集合竞价快照 | `rh_fuyao_auction_snapshot` | `thscodes` 必填≤100；`stage=live/final`（默认 final） |
| 竞价短期基准 | `rh_fuyao_auction_benchmark` | `date`=`YYYY-MM-DD`；省略=上海当日 |
| 涨停池 | `rh_fuyao_limit_up_pool` | `trade_date`；默认 size=200 |
| 跌停池 | `rh_fuyao_limit_down_pool` | 同涨停池日期别名；默认 size=200 |
| 炸板池 | `rh_fuyao_limit_break_pool` | 同涨停池；`open_times`=开板次数≠连板 |
| 连板天梯 | `rh_fuyao_limit_up_ladder` | 无日期 |
| 当日异动 | `rh_fuyao_anomaly_*` | 无历史 |
| 飙升/热股（当前） | `rh_fuyao_skyrocket_list` / `hot_stock_list` | 非历史 |
| 历史热股 | `rh_fuyao_hot_stock_history` | `date` |
| 热榜走势 | `rh_fuyao_hot_stock_trend` | `thscode`；`start_date`/`end_date` |
| 龙虎榜 | `rh_fuyao_dragon_tiger` | `board_type`；可选 date |
| 指数目录/成分/行情 | `rh_fuyao_index_*` | tag / thscode |
| 基金资料/重仓/净值/收益/持有人 | `rh_fuyao_fund_profile` / `holdings` / `nav` / `returns` / `holders` | `fund_type` 必填；净值用 `range` |
| ETF/LOF 场内 | `rh_fuyao_fund_market_snapshot` / `market_historical` | 快照可 ETF/LOF；日线**仅 ETF** |
| 基金公司 | `rh_fuyao_fund_company` | `company_id`（来自 profile） |
| 行业配置 / 大类资产 | `rh_fuyao_fund_industry_allocation` / `asset_allocation` | `fund_type`+`thscode` |
| 历史业绩指标 / 回撤 | `rh_fuyao_fund_indicators_historical` / `drawdowns` | 指标需 `start`/`end`（≤5 年） |
| 前十大持有人 / 分红 / 诊断 | `rh_fuyao_fund_holders_top` / `dividends` / `diagnostics` | `fund_type`+`thscode` |
| 基金三表+指标 | `rh_fuyao_fund_financials_*` | 与 A 股财务工具勿混用 |
| 基金经理 | `rh_fuyao_fund_manager_*` | `manager_id`（来自 profile.`manager_info`） |
| 基金资讯 | `rh_fuyao_fund_news` | `offset` 为游标；无 `total`，看 `has_more` |
| 在售/待售 | `rh_fuyao_fund_offerings` | `subscribe=active/upcoming` |
| 历史股/债持仓 | `rh_fuyao_fund_stock_*` / `bond_*` | 先 `*_report_dates` 再 `*_history` |
| 任意 GET | `rh_fuyao_get` | path+query |

## 服务端已做的归一化（减少踩坑）

- `trade_date`/`date`/`YYYYMMDD` → 涨停/跌停/炸板池 `date_ms`；热榜历史/龙虎榜/竞价基准 → `date`
- `thscode` ↔ `thscodes`（按端点单码/多码；竞价快照走多码）
- 日 K / 财务 / 基金历史业绩指标 `start`/`end`：`YYYY-MM-DD` → 毫秒（结束日含当日）
- 涨停/跌停/炸板池误传 `limit`/`offset` → 尝试映射 `size`/`page`；默认 size=200
- 对 anomaly / 当前热榜 / 飙升榜传历史日期 → `skipped`+`hint` 短路，不打官方
- 基金净值剥离错误的自定义 start/end；默认 `nav_type=unit,adj`
- 竞价快照默认 `stage=final`；`timestamp` 是响应组装时间，不是上游竞价发生时刻

## 维护

产品文档与工作台 Skill **同源意图**；运行时工作台只加载 `pool_agent/skills/rhths-fuyao`。  
官方细节变更时，优先改 MCP `NormalizeQuery` + 工作台 Skill，勿让 Agent 去翻开发目录。
