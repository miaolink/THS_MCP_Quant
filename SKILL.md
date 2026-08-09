---
name: rhths-trade
description: >-
  Operate THS PC trading via RHTHS gateway using MCP tools
  (rhths-mcp) or CLI (rhths.exe). Use for OpenClaw or Hermes Agent with
  rhths-trade MCP, rh_trade_*/rh_market_* locally first; rh_fuyao_* for
  HiThink Financial-API (fuyao) cloud data when local data is missing.
  Prefer dry-run; require confirm_live for live trades.
disable-model-invocation: false
---

# RHTHS — MCP + CLI Agent Skill

## Product (one paragraph)

**RHTHS** bridges **THS PC** to MCP/CLI via local gateway `http://127.0.0.1:19312`. 
Primary path: in-process Python **`ths_api`** (same class of usage as strategy scripts), not screen scraping / click automation. 
Optional cloud data: GUI **API** tab → fuyao Key → MCP `rh_fuyao_*` / CLI `rhths fuyao` (prefer local for overlapping quotes/trades).
Install RHTHS, deploy extension, log in to THS (strategy/trading Python env must be available), then **MCP** (OpenClaw / Hermes Agent) or **CLI** (scripts).

**Notice:** MCP/CLI unified entry on your PC. Cloud research data uses **Tonghuashun Financial Data Service** official standard API ([fuyao.aicubes.cn](https://fuyao.aicubes.cn/)); **users apply and keep their own API token**. Trading uses your own logged-in THS account/environment. Follow service terms, 券商 rules, and local law. See [README.md § 使用说明与合规提示](./README.md#使用说明与合规提示).

**AI trading (priority):** OpenClaw or Hermes Agent → MCP `rhths-trade` → `rhths-mcp.exe`. 
**Quant scripts:** `rhths.exe` on the **same Windows machine** as THS.

---

## Choose MCP vs CLI

| Use | When |
|-----|------|
| **MCP** | User talks to OpenClaw or Hermes Agent; natural-language trading assistant |
| **CLI** | PowerShell, cron, Python `subprocess`, batch files on **trading PC** |
| **MCP HTTP** | AI on **another PC** on LAN; trading PC runs `rhths-mcp.exe http` |

Both share the same gateway and semantics. Default orders are **simulate** (`dry_run` / `--dry-run`).

---

## Preconditions (always check first)

1. Windows + THS (or xiadan) **running and logged in**
2. RHTHS extension deployed (`rhths-gui.exe` → 安装/更新扩展)
3. Health OK:
 - MCP: call `rh_trade_health`
 - CLI: `rhths health` → JSON `ok: true`, `data.ready: true`

If health fails: start THS , redeploy extension, restart xiadan.

---

## MCP setup (summary)

**Server name:** `rhths-trade`

**Local (stdio)** — AI and THS on same PC:

```json
{
 "mcpServers": {
 "rhths-trade": {
 "command": "D:\\path\\to\\dist\\rhths-mcp.exe",
 "args": ["stdio"],
 "env": {
 "RHTHS_GATEWAY_URL": "http://127.0.0.1:19312",
 "RHTHS_ALLOW_LIVE": "0"
 }
 }
 }
}
```

**LAN (HTTP)** — AI on laptop, trading PC at home:

- Trading PC: `rhths-mcp.exe http` (or `rhths-gui.exe` keeps HTTP on `:19310`)
- Client config: `"url": "http://192.168.x.x:19310/mcp"` (GUI uses POST `/mcp`; some clients need `http --sse` + `/sse`)

Templates: [mcp.server.rhths-trade.json](./mcp.server.rhths-trade.json) · Full doc: [MCP使用说明.md](./MCP使用说明.md)

---

## MCP tools ↔ CLI (quick map)

| Task | MCP tool | CLI equivalent |
|------|----------|----------------|
| Health / routes | `rh_trade_health` / `rh_trade_catalog` | `rhths health` / `rhths catalog` |
| hexin / xiadan | `rh_system_*` | `rhths system status` / `start-xiadan` … |
| Accounts | `rh_trade_users` | `rhths users` |
| Balance / daily | `rh_trade_account` / `rh_trade_account_daily` | `rhths account` / `account-daily` |
| Positions | `rh_trade_positions` | `rhths positions` |
| Today / history orders | `rh_trade_orders_today` / `rh_trade_orders_history` | `rhths orders today` / `history` |
| Mode | `rh_trade_mode_get` / `rh_trade_mode_set` | GUI `settings.json` |
| Buy / sell / cancel | `rh_trade_buy` / `sell` / `cancel` | `rhths buy` / `sell` / `cancel` |
| Cancel all | `rh_trade_cancel_all` | `rhths cancel-all` |
| Condition resume | `rh_condition_resume` | `rhths condition resume` |
| Py dispatch | `rh_py_call` | `rhths py call --action …` |
| Quote / K / wencai | `rh_market_*` | `rhths market …` |
| Cloud finance / special（fuyao） | `rh_fuyao_*` | `rhths fuyao ping` / `search` / `get` |
| Autotrading review | `rh_autotrading_*` | `rhths autotrading …` |
| Indicator | `rh_indicator_calc` | `rhths indicator calc CODE MACD` |

**Routing:** prefer local `rh_trade_*` / `rh_market_*` for trading and live quotes. Use `rh_fuyao_*` only for data not available locally (financials, valuations, calendar, limit-up, funds, meta search, etc.). Configure Key in GUI **API** tab (`HITHINK_FINANCE_API_KEY`).

**Not in scope:** MySQL sync, thsQuant `:19090`. Gateway only: `http://127.0.0.1:19312`.

Full tool list: [mcp.tools.json](./mcp.tools.json) · CLI: [CLI使用说明.md](./CLI使用说明.md)

---

## Trading rules (agent must follow)

1. **Default simulate** — `dry_run: true` or omit with mode `simulate`; never live unless user explicitly confirms risk.
2. **Live requires:**
 - Env `RHTHS_ALLOW_LIVE=1` on trading machine
 - MCP: `confirm_live: true` on each write; or `rh_trade_mode_set` → `live`
 - CLI: `--live --confirm` and `$env:RHTHS_ALLOW_LIVE = "1"`
3. **Standard edition:** live buy+sell combined **≤ 10 per day**; error `FREE_DAILY_LIMIT` → use simulate or upgrade (GUI 激活).
4. **Read before write** — prefer `rh_trade_account` / `rh_trade_positions` before suggesting orders.
5. **Codes** — 6-digit A-share codes; prices often `zxjg` (buy), `dsj3` (sell).

---

## Example agent flows

**Check portfolio (MCP):**

1. `rh_trade_health`
2. `rh_trade_account`
3. `rh_trade_positions`

**Simulate buy (MCP):**

```json
{ "code": "600000", "qty": 100, "price": "zxjg", "dry_run": true }
```

→ tool `rh_trade_buy`

**Same via CLI (trading PC shell):**

```powershell
rhths buy 600000 --qty 100 --price zxjg --dry-run
```

**OpenClaw / Hermes:** merge `rhths-trade` from `mcp.server.rhths-trade.json`; reload MCP; verify with `rh_trade_health`.

---

## Errors

| Code / symptom | Action |
|----------------|--------|
| Connection failed | THS not running / not logged in; redeploy extension |
| `LIVE_BLOCKED` | `RHTHS_ALLOW_LIVE=0` or missing `confirm_live` |
| `FREE_DAILY_LIMIT` | Standard edition daily live cap |
| Remote MCP timeout | Firewall TCP 19310; ping `http://IP:19310/health` |

---

## Docs index

| File | Purpose |
|------|---------|
| [README.md](./README.md) | Product intro, THS editions, no external API |
| [快速开始.md](./快速开始.md) | Install checklist |
| [MCP使用说明.md](./MCP使用说明.md) | OpenClaw / Hermes / LAN HTTP |
| [CLI使用说明.md](./CLI使用说明.md) | All `rhths` subcommands |

**Online:** https://www.miaolink.cn/rhths/index.php

---

## 安装为 Agent Skill（OpenClaw · Hermes Agent 标准）

本文件为 **AgentSkills 兼容** 的 `SKILL.md`（YAML frontmatter + 说明正文）。**MCP 工具连接**与 **Skill 知识库**是两套配置，需分别完成。

### OpenClaw（目录 + `mcpServers`）

遵循 [OpenClaw Skills](https://documentation.openclaw.ai/tools/skills)（AgentSkills 目录规范）：

1. 创建技能目录，例如：
 - 本机共享：`~/.openclaw/skills/rhths-trade/`
 - 或工作区：`<你的工作区>/skills/rhths-trade/`
2. 将本文件保存为该目录下的 **`SKILL.md`**（保留顶部 `name` / `description` frontmatter）。
3. **MCP：** 把 [mcp.server.rhths-trade.json](./mcp.server.rhths-trade.json) 里的 **`mcpServers.rhths-trade`** 合并进 OpenClaw 的 MCP 配置（标准 `mcpServers` JSON，键名 `rhths-trade`）。
4. 交易机已安装 RHTHS、扩展已部署、 THS 已登录；将 `__RHTHS_DIST__` 换为 `rhths-mcp.exe` 绝对路径。
5. 重载 MCP / 重启 OpenClaw；在对话中验证 **`rh_trade_health`**。

可选：在 frontmatter 增加单行 `metadata`（OpenClaw 用于依赖检查），例如要求本机存在 `rhths-mcp.exe`：

```yaml
metadata: {"openclaw":{"requires":{"bins":["rhths-mcp.exe"]},"os":["win32"]}}
```

### Hermes Agent（`~/.hermes/skills` + `mcp_servers`）

遵循 [Hermes MCP 配置参考](https://hermes-agent.nousresearch.com/docs/reference/mcp-config-reference)：

**① Skill（供 Agent 阅读的交易规范）**

```text
~/.hermes/skills/rhths-trade/SKILL.md ← 复制本文件
```

**② MCP（注册 `rhths-trade` 工具，编辑 `~/.hermes/config.yaml`）**

本机 stdio（AI 与交易在同一台 Windows）：

```yaml
mcp_servers:
 rhths-trade:
 command: "D:/path/to/dist/rhths-mcp.exe"
 args: ["stdio"]
 env:
 RHTHS_GATEWAY_URL: "http://127.0.0.1:19312"
 RHTHS_ALLOW_LIVE: "0"
 timeout: 120
 connect_timeout: 60
```

局域网 HTTP（AI 在其它电脑，交易机常开 `rhths-mcp.exe http`）：

```yaml
mcp_servers:
 rhths-trade:
 url: "http://192.168.1.100:19310/mcp"
 timeout: 120
 connect_timeout: 60
```

> Hermes 规定：每个 server **只能**使用 `command`（stdio）**或** `url`（HTTP），不能同时填写。

3. 保存配置后执行 **`/reload-mcp`** 或重启 Hermes；验证 **`rh_trade_health`**。

### 对照

| 项目 | OpenClaw | Hermes Agent |
|------|----------|----------------|
| Skill 路径 | `~/.openclaw/skills/rhths-trade/SKILL.md` | `~/.hermes/skills/rhths-trade/SKILL.md` |
| MCP 配置 | OpenClaw `mcpServers` JSON | `~/.hermes/config.yaml` → `mcp_servers` |
| 机读 MCP 片段 | [mcp.server.rhths-trade.json](./mcp.server.rhths-trade.json) | 同上（改为 YAML `mcp_servers` 块） |
| 重载 | 重启或重载 MCP | `/reload-mcp` |
