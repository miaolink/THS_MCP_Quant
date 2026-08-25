# MCP 使用说明

通过 **MCP（Model Context Protocol）**，各类 AI Agent 客户端可以调用 THS 交易与行情能力。 
RHTHS 不绑定单一产品：**凡支持 MCP 的客户端均可使用**，配置结构相同。

> 在本机 THS 策略 / 下单环境中经 **`ths_api`** 提供能力（非读屏点选主路径）。第三方辅助工具，非 THS 官方产品。使用说明见 [README.md](./README.md#使用说明与合规提示)。Agent 速查见 [SKILL.md](./SKILL.md)。
---
## 一、适用客户端与两种部署
### 1.1 常见 MCP 客户端（均适用）

以下客户端只需在各自的 MCP 配置里合并 **`rhths-trade`** 节点（见下文），无需 RHTHS 单独适配。

**优先顺序（与购买页 / README 一致）：** WorkBuddy → Codex → Hermes Agent；亦可 OpenClaw / Cursor 等。

| 客户端 | 典型配置文件位置 |
|--------|------------------|
| **WorkBuddy**（首选） | WorkBuddy 设置中的 MCP / `mcpServers` |
| **Codex**（次选） | Codex MCP 配置 |
| **Hermes Agent**（第三优先） | Hermes 的 MCP 服务器配置 |
| **OpenClaw / Claw** | Claw 设置中的 MCP / `mcpServers` |
| **CodeBuddy** | CodeBuddy MCP 设置 |
| **Cursor** | `%USERPROFILE%\.cursor\mcp.json` |
| **Claude Desktop** | `%APPDATA%\Claude\claude_desktop_config.json` |
| **Windsurf** | Windsurf MCP 设置 |
| **Cline** | VS Code 扩展 MCP 配置 |
| **Continue** | `config.json` 中 `mcpServers` |
| **Cherry Studio** | 设置 → MCP 服务器 |
| **Zed** | 项目/用户 `settings.json` 中的 MCP |

凡支持 `command`+`stdio` 或 `url`+HTTP/SSE 的 Agent / AI 编程客户端，均可按本文配置。
**机读模板（本机 stdio）**：[mcp.server.rhths-trade.json](./mcp.server.rhths-trade.json) 
**工具清单（机读）**：[mcp.tools.json](./mcp.tools.json) 
**AI 一句话自动安装 MCP + Skill**：[AI自动安装MCP与SKill示例.md](./AI自动安装MCP与SKill示例.md)
### 1.2 本机 vs 局域网 / 远程
| 模式 | 何时使用 | 安装章节 |
|------|----------|----------|
| **本机 stdio**（推荐） | AI 客户端与**THS 在同一台 Windows** | [§ 二](#二本机安装stdio) |
| **局域网 / 远程 HTTP** | AI 在**另一台电脑**，交易机在家/办公室常开 | [§ 三](#三局域网--远程安装mcp-http) |
共同点：
- **THS、扩展服务、网关 `19312` 必须在「交易机」上运行。**
- `rhths-mcp.exe` 通过环境变量 `RHTHS_GATEWAY_URL=http://127.0.0.1:19312` 访问本机网关（交易机上的 MCP 进程始终连本机 19312，与 AI 在哪台电脑无关）。
### 1.3 使用前确认（交易机）
1. THS 已启动并**登录**交易账户 
2. 网关可用：`rhths.exe health` 返回正常，或浏览器打开 `http://127.0.0.1:19312/v1/system/health` 见 `ok: true` 
3. 已安装 `rhths-mcp.exe`（一般在 RHTHS 的 `dist` 目录）
---
## 二、本机安装（stdio）
**适用**：Cursor、OpenClaw、Hermes Agent、Claude Desktop 等运行在**与交易同一台 PC** 上。
**原理**：由 AI 客户端**启动** `rhths-mcp.exe stdio`，通过标准输入输出通信，**无需**单独开 HTTP 端口，也不会与 GUI 的 HTTP 守护抢 **19310**。
### 2.1 通用配置项
在客户端的 `mcpServers` 中增加服务器名 **`rhths-trade`**：
| 项 | 值 |
|----|-----|
| `command` | `rhths-mcp.exe` 的**绝对路径** |
| `args` | `["stdio"]` |
| `env.RHTHS_GATEWAY_URL` | `http://127.0.0.1:19312` |
| `env.RHTHS_ALLOW_LIVE` | `0`（建议，见「实盘」） |
将 [mcp.server.rhths-trade.json](./mcp.server.rhths-trade.json) 中 `__RHTHS_DIST__` 替换为 `dist\` 的绝对路径（含末尾 `\`）后，把 `mcpServers.rhths-trade` 整段合并进客户端配置。
**JSON 示例：**
```json
{

 "mcpServers": {

 "rhths-trade": {

 "command": "D:\\allinpol\\RHTHS\\dist\\rhths-mcp.exe",

 "args": ["stdio"],

 "env": {

 "RHTHS_GATEWAY_URL": "http://127.0.0.1:19312",

 "RHTHS_ALLOW_LIVE": "0"

 }

 }

 }
}
```
### 2.2 Cursor（本机）
**方式 A — 一键脚本**
```powershell
cd D:\allinpol\RHTHS
.\scripts\build-go.ps1
.\scripts\setup-cursor-mcp.ps1
```
写入 `%USERPROFILE%\.cursor\mcp.json`（只合并 `rhths-trade`）。保存后：**Settings → MCP** 刷新，或 **Reload Window**。
**方式 B — 手动** 
按 § 2.1 编辑 `%USERPROFILE%\.cursor\mcp.json`。
**自检：** `.\scripts\verify-mcp.ps1`
### 2.3 OpenClaw / Claw（本机）
1. 打开 [mcp.server.rhths-trade.json](./mcp.server.rhths-trade.json)，替换 `__RHTHS_DIST__`。 
2. 将 **`mcpServers.rhths-trade`** 合并进 Claw 的 MCP 配置（结构与 Cursor 相同）。 
3. 保持 `args: ["stdio"]`，`RHTHS_GATEWAY_URL` 指向 `127.0.0.1:19312`。 
4. 重启 Claw 或重新加载 MCP。
**不要**在本机同时用 Claw stdio + 单独 `rhths-mcp.exe http` 抢 **19310**（stdio 无此问题）。
### 2.4 Hermes Agent（本机）
1. 在 Hermes Agent 设置中找到 **MCP Servers**（或等价项）。 
2. 新增服务器，名称建议 **`rhths-trade`**。 
3. 类型选 **stdio / 本地命令**（名称因版本而异），填入： 

 - 命令：`D:\...\dist\rhths-mcp.exe` 

 - 参数：`stdio` 

 - 环境变量：`RHTHS_GATEWAY_URL=http://127.0.0.1:19312`，`RHTHS_ALLOW_LIVE=0` 
4. 若 Hermes 支持直接粘贴 JSON，可粘贴 § 2.1 整段 `rhths-trade` 节点。 
5. 保存后重启 Hermes 或重载 MCP，在对话中让其调用 `rh_trade_health` 验证。
### 2.5 Claude Desktop / Windsurf / Cline / 其它（本机）
与各客户端文档一致：在 `mcpServers`（或扩展提供的 MCP 配置）中加入 § 2.1 的 `rhths-trade` 节点，`command` 改为本机绝对路径。 
保存后重启客户端或 Reload MCP。
---
## 三、局域网 / 远程安装（MCP HTTP）
**适用**：
- 交易机：家里/办公室 Windows，**常开 THS + RHTHS** 
- 客户端：笔记本、公司电脑上的 **Cursor / OpenClaw / Hermes Agent** 等，与交易机在同一**局域网**（或 VPN 内网）
**原理**：在**交易机**上运行 `rhths-mcp.exe http`（监听 **19310**，默认绑定 `0.0.0.0`）；AI 客户端通过 **`http://交易机局域网IP:19310`** 连接，不再在客户端机器上安装 THS。
> **安全提示**：仅在内网使用；防火墙只放行可信网段；**切勿**把 19310 暴露到公网。
### 3.1 交易机：启动 MCP HTTP
1. 确认 THS、扩展服务、网关 `19312` 正常（`rhths health`）。 
2. 启动 MCP HTTP（任选其一）：
**方式 A — GUI（推荐）** 
运行 `rhths-gui.exe`，保持运行；GUI 会守护 **19310** 上的 MCP HTTP（**POST `/mcp`**）。远程客户端请用 `url`: `http://交易机IP:19310/mcp`。
**方式 B — 命令行**
```powershell
$env:RHTHS_GATEWAY_URL = "http://127.0.0.1:19312"
$env:RHTHS_ALLOW_LIVE = "0"
D:\allinpol\RHTHS\dist\rhths-mcp.exe http
```
可选 SSE（部分客户端需要）：
```powershell
D:\allinpol\RHTHS\dist\rhths-mcp.exe http --sse
```
3. 查交易机局域网 IP，例如 `192.168.1.100`（`ipconfig`）。 
4. 在**交易机**上自检：
```powershell
Invoke-RestMethod http://127.0.0.1:19310/health
# 或项目内： .\scripts\verify-mcp-http.ps1
```
5. **Windows 防火墙**：入站规则放行 **TCP 19310**（仅专用/局域网配置文件）。
| 环境变量 | 说明 |
|----------|------|
| `RHTHS_MCP_HTTP_PORT` | 默认 `19310` |
| `RHTHS_BIND_HOST` / `RHTHS_MCP_HTTP_HOST` | 监听地址，默认 `0.0.0.0`（局域网可连） |
| `RHTHS_GATEWAY_URL` | 在交易机上保持 `http://127.0.0.1:19312` |
### 3.2 客户端：连接交易机 MCP
在**另一台电脑**的 MCP 配置中，使用 **URL** 指向交易机（将 `192.168.1.100` 换成实际 IP）：
**SSE 模式（Claude Desktop、部分 Agent 推荐）**
```json
{

 "mcpServers": {

 "rhths-trade": {

 "url": "http://192.168.1.100:19310/sse"

 }

 }
}
```
> 交易机须以 `rhths-mcp.exe http --sse` 启动（**GUI 守护的是 POST `/mcp`**，不用 SSE；若客户端只认 SSE，请改用手动 `http --sse` 启动）。
**Streamable HTTP（POST `/mcp`，部分 Cursor 版本）**
```json
{

 "mcpServers": {

 "rhths-trade": {

 "url": "http://192.168.1.100:19310/mcp"

 }

 }
}
```
在客户端机器上先测试连通：
```powershell
Invoke-RestMethod http://192.168.1.100:19310/health
```
再在 AI 对话中调用 `rh_trade_health`。
### 3.3 OpenClaw / Hermes Agent（远程）
- **OpenClaw**：在 MCP 配置中使用 `url` 字段（上表），不要用 `command`+`stdio`（stdio 只能在交易机本地 spawn 进程）。 
- **Hermes Agent**：选择 **远程 / URL** 类型 MCP，填入 `http://192.168.1.100:19310/sse` 或 `/mcp`（与交易机启动方式一致）。
远程模式下 **客户端无需**安装 `rhths-mcp.exe`；但交易机必须保持 THS、网关、MCP HTTP 运行。
### 3.4 本机 stdio 与远程 HTTP 对比
| | 本机 stdio | 局域网 HTTP |
|---|------------|-------------|
| THS 位置 | 本机 | 交易机 |
| 客户端配置 | `command` + `args: ["stdio"]` | `url`: `http://IP:19310/sse` 或 `/mcp` |
| 客户端是否需 `rhths-mcp.exe` | 需要（由客户端启动） | 不需要 |
| 端口 | 无额外端口 | 交易机 **19310** |
| 典型场景 | 开发机即交易机 | 笔记本连家里台式机 |
---
## 四、验证 MCP 是否生效
1. 在 AI 对话中让其调用 **`rh_trade_health`**。 
2. 正常应返回网关版本、`ths_api` 是否就绪。 
3. 再试 **`rh_trade_account`** 或 **`rh_trade_positions`**。
若失败：
- **本机 stdio**：先 `rhths health`，再查 `command` 路径、`args` 是否为 `stdio`。 
- **远程 HTTP**：先在客户端 `curl`/浏览器访问 `http://交易机IP:19310/health`，再查防火墙与 IP。
---
## 五、工具说明（给 AI / 用户查阅）
### 5.1 系统与模式
| 工具 | 用途 |
|------|------|
| `rh_trade_health` | 检查网关、 THS `ths_api` 是否就绪 |
| `rh_trade_catalog` | 列出网关支持的路由 |
| `rh_trade_mode_get` | 查看当前 **模拟 / 实盘** 默认模式 |
| `rh_trade_mode_set` | 切换模式：`mode` = `simulate` 或 `live` |
模拟/实盘默认与 **rhths-gui** 共用配置（`%APPDATA%\RHTHS\settings.json`）。 
未传 `dry_run` 的买卖撤单将使用该默认。
### 5.2 交易查询（只读）
| 工具 | 用途 |
|------|------|
| `rh_trade_users` | 资金账号列表 |
| `rh_trade_account` | 资金（可选写日快照） |
| `rh_trade_account_daily` | 资金日快照历史 |
| `rh_trade_positions` | 持仓（含可卖数量） |
| `rh_trade_orders_today` | 当日委托 |
| `rh_trade_orders_history` | 历史委托（可 `start_date` / `end_date` / `code`） |
| `rh_trade_warmup` | 预热缓存（批量查询前可选） |
可选参数：`user`（资金账号）、`rzrq`（融资融券）。
### 5.3 下单与撤单（写操作）
| 工具 | 用途 |
|------|------|
| `rh_trade_buy` | 买入 |
| `rh_trade_sell` | 卖出 |
| `rh_trade_cancel` | 撤单 |
| `rh_trade_cancel_all` | 全部撤单 |
| `rh_trade_invoke` | 原始命令（高级） |
常用参数：`code`、`qty`、`price`（如 `zxjg` / `dsj3`）、`dry_run`、`confirm_live`。
### 5.4 行情与指标
| 工具 | 用途 |
|------|------|
| `rh_market_quote` | 单股行情 |
| `rh_market_price` | 批量现价 |
| `rh_market_kline` | K 线 |
| `rh_market_wencai` | 问财选股语句 |
| `rh_market_select_stocklist` | 问财结果代码列表 |
| `rh_indicator_list` / `rh_indicator_calc` | 指标列表与计算 |
| `rh_market_timeshare` / `rh_market_depth` | 分时 / 盘口 |
| `rh_market_kline_probe` / `rh_market_kline_15s` | K 线周期探测 / 15 秒 K |
| `rh_market_stock5d` / `rh_market_wencai_detail` | 5 日因子 / 问财详情 |
| `rh_market_moneyflow_probe` | 资金流探测 |
| `rh_market_block_list` / `rh_market_block_stocks` | 板块 |
订阅类：`rh_market_reg_quote`、`rh_market_reg_kline`、`rh_market_wait_update` 等。

### 5.5 条件单、py/call、进程、自动交易审核

| 工具 | 用途 |
|------|------|
| `rh_condition_*` | 条件单增删改查、`resume`（≈ 恢复策略） |
| `rh_py_call` | 统一分发：`condition.*`、`market.*`、`trade.*` |
| `rh_system_*` | 启动/停止 hexin、xiadan |
| `rh_autotrading_*` | 本地 JSON 审核配置与队列（**非 MySQL**） |

### 5.6 同花顺金融数据服务（官方标准 API）

重叠的**实时行情 / 交易**请优先用上面的 `rh_market_*` / `rh_trade_*`（本机已登录环境）。研究类数据通过 **同花顺金融数据服务** 的 **官方标准 API** 转发（[fuyao.aicubes.cn](https://fuyao.aicubes.cn/) · [Financial-API](https://github.com/HiThink-Tech/Financial-API)），**不替代下单**。

**Token（用户自备）：** 请到 <https://fuyao.aicubes.cn/admin/> **自行申请** API Key，在 `rhths-gui` → **「API」** 页保存（仅写入本机 `%APPDATA%\RHTHS\settings.json` 与环境变量 `HITHINK_FINANCE_API_KEY`）。RHTHS 不代申请、不托管他人密钥；用量与计费以官方账号为准。

| 工具 | 用途 |
|------|------|
| `rh_fuyao_ping` | 探测 Key 是否有效 |
| `rh_fuyao_meta_search` / `rh_fuyao_meta_list` | 标的名称消歧 / 代码表 |
| `rh_fuyao_prices_snapshot` / `rh_fuyao_prices_historical` | 云端行情快照 / 日 K（**优先**本地 `rh_market_quote` / `kline`） |
| `rh_fuyao_adjustment_factors` | 复权事件 |
| `rh_fuyao_financials_*` | 利润表 / 资产负债表 / 现金流量表 / 财务指标 |
| `rh_fuyao_valuations_snapshot` | 估值快照（PE/PB 等） |
| `rh_fuyao_calendar_trading_days` | 近一年交易日历 |
| `rh_fuyao_auction_snapshot` / `auction_benchmark` | 集合竞价快照 / 短期强弱基准 |
| `rh_fuyao_index_*` | 指数/板块目录、成分、行情 |
| `rh_fuyao_fund_*` | 基金资料、持仓、净值、收益、公司/经理、财务、资讯、历史持仓、场内行情 |
| `rh_fuyao_limit_up_*` / `limit_down_pool` / `limit_break_pool` / `hot_*` / `dragon_tiger` 等 | 涨停/跌停/炸板池、热榜、龙虎榜等特色数据 |
| `rh_fuyao_get` | 通用 `GET /api/*`（高级） |

**用法与避错：** 工作台用 Skill `rhths-fuyao`（`pool_agent/skills`）；产品侧对照 [fuyao-routing.md](./fuyao-routing.md)。运行时勿依赖开发仓路径。

CLI：`rhths fuyao ping | search | get`。

### 5.7 同花顺云端自选 / 动态板块

不依赖本机 hexin。Cookie 在 `rhths-gui` → **「自选」** 页手动粘贴并保存（`%APPDATA%\RHTHS\settings.json` 的 `ths_cookie`，或环境变量 `THS_COOKIE` / `RHTHS_THS_COOKIE`）。保存后需重启 MCP 进程。

| 工具 | 用途 |
|------|------|
| `rh_ths_favorite_list` | 列出自选分组；可选 `group` 只看一组 |
| `rh_ths_favorite_add` / `rh_ths_favorite_delete` | 自选分组增删股票。`group` 必填；`code` 或 `codes` |
| `rh_ths_block_list` | 列出动态板块/自定义分组 |
| `rh_ths_block_add` / `rh_ths_block_delete` | 板块增删股票 |

代码格式：`600519.SH` 或 6 位代码（按首位推断市场）。CLI：`rhths favorite list|show|add|delete`，`rhths block …`。

完整机读列表：[mcp.tools.json](./mcp.tools.json)。**不连接** thsQuant `:19090`，**不提供** MySQL 工具。

---
## 六、模拟下单与实盘
### 默认（建议）
- 环境变量 **`RHTHS_ALLOW_LIVE=0`** 
- MCP 调用买卖时 **`dry_run: true`** 或 `rh_trade_mode_set` 为 **`simulate`**
### 实盘（需人工确认，风险自负）
1. 在**交易机**上设置：`RHTHS_ALLOW_LIVE=1`（stdio 写在 `env`；HTTP 在启动 MCP 前设环境变量） 
2. MCP：`rh_trade_mode_set` → `{"mode":"live"}` 或每笔传 `dry_run: false` 
3. 每笔实盘须 **`confirm_live: true`** 
4. **标准版**：实盘买入+卖出合计每天最多 **10 次**；**高级版**无限制（GUI 激活）
---
## 七、常见错误
| 返回 / 现象 | 处理 |
|-------------|------|
| 连接失败 | THS 未开或未登录；重新部署扩展 后重启 xiadan |
| 远程 `health` 超时 | 防火墙、IP 错误、交易机未开 MCP HTTP |
| `LIVE_BLOCKED` | 未开 `RHTHS_ALLOW_LIVE` 或未 `confirm_live` |
| `FREE_DAILY_LIMIT` | 标准版当日实盘次数已满 |
| `FUYAO_ERROR` / 认证失败 | GUI「API」检查 Key；或访问 fuyao 管理页重新创建 |
| `THS_UGC_ERROR` / Cookie 未配置 | GUI「自选」填写 Cookie 并保存；然后重启 MCP |
| 工具找不到 | 检查路径 / URL、重建 `rhths-mcp.exe`；`rh_fuyao_*` 需新版 MCP |
| 19310 被占用 | 关闭多余 `rhths-mcp.exe`；本机优先用 stdio，远程才用 HTTP |
更多排障见 [快速开始.md](./快速开始.md)。CLI 仅本机使用见 [CLI使用说明.md](./CLI使用说明.md)。
