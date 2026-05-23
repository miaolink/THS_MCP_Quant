---
name: rhths-trade
description: >-
  Operate Tonghuashun (同花顺) PC trading via RHTHS gateway using MCP tools
  (rhths-mcp) or CLI (rhths.exe). Use when the user asks about RHTHS, 同花顺
  AI trading, OpenClaw/Hermes/Cursor MCP rhths-trade, rh_trade_* tools,
  rhths CLI commands, positions, orders, dry-run/live, or 无需 API 授权.
disable-model-invocation: false
---

# RHTHS — MCP + CLI Agent Skill

## Product (one paragraph)

**RHTHS** bridges **同花顺 PC** to MCP/CLI via local gateway `http://127.0.0.1:19312`.  
**Not WinGUI automation** — no simulated mouse/keyboard or window scraping. It uses **native in-process `ths_api`** (same as 同花顺 strategy scripts): **fast, stable, no third-party API quota**.  
**No external API authorization** required. Install RHTHS, deploy Hook, log in to 同花顺, then **MCP** (OpenClaw/Hermes/Cursor) or **CLI** (scripts).

**Compliance:** No reverse-engineering or cracking of 同花顺; uses **standard in-process `ths_api` calls** only (same class as official strategy scripts). User must follow 同花顺/券商 terms. See [README.md § 技术与合规说明](./README.md#技术与合规说明合法使用).

**AI trading (priority):** OpenClaw, Hermes Agent, Cursor → MCP server `rhths-trade` → `rhths-mcp.exe`.  
**Quant scripts:** `rhths.exe` on the **same Windows machine** as 同花顺.

---

## Choose MCP vs CLI

| Use | When |
|-----|------|
| **MCP** | User talks to OpenClaw / Hermes / Cursor; natural-language trading assistant |
| **CLI** | PowerShell, cron, Python `subprocess`, batch files on **trading PC** |
| **MCP HTTP** | AI on **another PC** on LAN; trading PC runs `rhths-mcp.exe http` |

Both share the same gateway and semantics. Default orders are **simulate** (`dry_run` / `--dry-run`).

---

## Preconditions (always check first)

1. Windows + 同花顺 (or xiadan) **running and logged in**
2. RHTHS Hook deployed (`rhths-gui.exe` → 安装/更新 Hook)
3. Health OK:
   - MCP: call `rh_trade_health`
   - CLI: `rhths health` → JSON `ok: true`, `data.ready: true`

If health fails: start 同花顺, redeploy Hook, restart xiadan.

---

## MCP setup (summary)

**Server name:** `rhths-trade`

**Local (stdio)** — AI and 同花顺 on same PC:

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
| Health | `rh_trade_health` | `rhths health` |
| Accounts | `rh_trade_users` | `rhths users` |
| Balance | `rh_trade_account` | `rhths account` |
| Positions | `rh_trade_positions` | `rhths positions` |
| Today orders | `rh_trade_orders_today` | `rhths orders today` |
| History orders | `rh_trade_orders_history` | `rhths orders history` |
| Mode | `rh_trade_mode_get` / `rh_trade_mode_set` | (GUI settings.json; MCP preferred) |
| Buy | `rh_trade_buy` | `rhths buy CODE --qty N --dry-run` |
| Sell | `rh_trade_sell` | `rhths sell CODE --qty N --dry-run` |
| Cancel | `rh_trade_cancel` | `rhths cancel --dry-run` |
| Quote | `rh_market_quote` | `rhths market quote CODE` |
| Wencai | `rh_market_wencai` | `rhths market wencai "query"` |
| Indicator | `rh_indicator_calc` | `rhths indicator calc CODE MACD` |

Full tool params: [mcp.tools.json](./mcp.tools.json) · CLI detail: [CLI使用说明.md](./CLI使用说明.md)

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
| Connection failed | 同花顺 not running / not logged in; redeploy Hook |
| `LIVE_BLOCKED` | `RHTHS_ALLOW_LIVE=0` or missing `confirm_live` |
| `FREE_DAILY_LIMIT` | Standard edition daily live cap |
| Remote MCP timeout | Firewall TCP 19310; ping `http://IP:19310/health` |

---

## Docs index

| File | Purpose |
|------|---------|
| [README.md](./README.md) | Product intro, 同花顺 editions, no external API |
| [快速开始.md](./快速开始.md) | Install checklist |
| [MCP使用说明.md](./MCP使用说明.md) | OpenClaw / Hermes / LAN HTTP |
| [CLI使用说明.md](./CLI使用说明.md) | All `rhths` subcommands |

**Online:** https://www.miaolink.cn/rhths/index.php

---

## Copy as Cursor skill

Copy this file to project `.cursor/skills/rhths-trade/SKILL.md` or user `~/.cursor/skills/rhths-trade/SKILL.md` so agents auto-load RHTHS context.
