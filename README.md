# THS_MCP_Quant

**同花顺 PC 用户的 AI 交易与量化网关** — 适配**免费版、金融大师、远航版、券商定制版、独立下单（xiadan）**等常见 PC 客户端；**无需开通任何 API**，**默认安装后即可使用**。专为 **OpenClaw**、**Hermes Agent** 及 Cursor 等提供查仓、下单；同时支持 **CLI** 量化脚本。

在线文档：[https://www.miaolink.cn/rhths/index.php](https://www.miaolink.cn/rhths/index.php) · MCP 配置详见 [MCP使用说明.md](./MCP使用说明.md)

---

## AI 交易（首选：OpenClaw · Hermes Agent）

在 **OpenClaw** 或 **Hermes Agent** 里配置 RHTHS 的 MCP 服务 **`rhths-trade`** 后，用自然语言即可驱动同花顺账户（默认**模拟下单**，实盘须显式确认）：

| 你可以对 AI 说 | MCP 背后能力 |
|----------------|--------------|
| 「帮我看下资金和持仓」 | `rh_trade_account`、`rh_trade_positions` |
| 「今天有哪些委托」 | `rh_trade_orders_today` |
| 「模拟买入 100 股某某，最新价」 | `rh_trade_buy`（`dry_run`） |
| 「问财选市盈率小于 20 的票」 | `rh_market_wencai`、`rh_market_select_stocklist` |
| 「切换到实盘模式」（需谨慎） | `rh_trade_mode_set` + `confirm_live` |

**推荐客户端（配置方式相同，合并 `rhths-trade` 即可）：**

1. **OpenClaw / Claw** — 自主 Agent，本机 stdio 或局域网 HTTP 连交易机  
2. **Hermes Agent** — MCP Agent，本机命令或远程 URL  
3. **Cursor**、Claude Desktop、Windsurf、Cline、Continue、Cherry Studio 等

> 机读配置模板：[mcp.server.rhths-trade.json](./mcp.server.rhths-trade.json) · 工具列表：[mcp.tools.json](./mcp.tools.json)

```text
你 ──对话──► OpenClaw / Hermes Agent ──MCP──► rhths-mcp.exe ──► 同花顺 (ths_api)
                                              查仓 · 行情 · 下单（可先模拟）
```

**RHTHS**（**R**onghui **T**ong**H**ua**S**hun Gateway）在本机同花顺进程内对接 `ths_api`，对外暴露统一网关；**AI 交易走 MCP**，**量化脚本走 CLI**，二者共用同一套交易语义。

---

## 适配哪些同花顺？安装即用

以下 **同花顺 PC 端** 均可使用 RHTHS（只要你能用该软件**正常登录并交易**，无需区分是否「专业版」或是否已向同花顺申请 **任务 API**）：

| 客户端类型 | 说明 |
|------------|------|
| **PC 免费版** | 个人常用免费客户端 |
| **金融大师** | 完整行情与交易环境 |
| **远航版** | 远航系列 PC 客户端 |
| **券商定制版** | 各券商冠名、定制的同花顺 PC 版 |
| **独立下单（xiadan）** | 同花顺独立下单程序，与主客户端配合或单独使用 |

> **统一说明：以上版本均不需要另行开通「任务 API / 官方量化 API」。** RHTHS 在本机已运行的同花顺/下单进程内对接 `ths_api`，不依赖额外云端交易授权。

### 默认安装后可用（三步）

1. 安装 **RHTHS** 安装包（含 `rhths-gui.exe`、`rhths.exe`、`rhths-mcp.exe`）  
2. 在 GUI 中执行 **「安装 / 更新 Hook」**（或运行安装包自带部署脚本）  
3. **启动同花顺（或独立下单）并登录** → 运行 `rhths health` 显示正常 → 即可配置 **OpenClaw / Hermes Agent** 或运行 **CLI**

无需等待 API 审核、无需改券商通道、无需更换交易软件品牌。

---

## 谁可以直接用？无需另开「任务 API」

> **凡使用上表任一同花顺 PC 客户端、且已能手动登录下单的用户，安装 RHTHS 后即可做 AI 交易与量化——不必向同花顺申请任务 API。**

| 传统做法 | RHTHS 做法 |
|----------|------------|
| 申请任务 API / 量化 API、等待审核 | **默认安装 + Hook**，沿用现有 PC 客户端与账户 |
| 为 AI 单独开发交易接口 | **OpenClaw / Hermes Agent** 配 MCP 即可对话查仓、下单 |
| 云端 API 与 PC 持仓不一致 | 走**本机同花顺 `ths_api`**，与软件里看到的一致 |

**不需要：** 任务 API 开通、第三方云交易接口、迁移到其它交易系统。

建议先用 **模拟（dry-run）** 验证 AI 与策略，再按需实盘（风险自负）。

---

## 软件能做什么

| 能力 | 说明 |
|------|------|
| **AI 交易（MCP）** | **OpenClaw、Hermes Agent** 等通过对话查资金/持仓/委托、问财选股、模拟或实盘买卖 |
| **量化脚本（CLI）** | `rhths.exe` 供 PowerShell、计划任务、Python 等本机自动化 |
| **交易查询** | 资金账号、资金、持仓、当日/历史委托 |
| **下单与撤单** | 买入、卖出、撤单；默认模拟，实盘需 `confirm_live` |
| **行情与选股** | 实时行情、K 线、问财、指标（如 MACD） |
| **图形管理** | `rhths-gui.exe`：Hook、交易面板、激活、升级 |
| **产品授权** | RHTHS 标准版/高级版（≠ 同花顺任务 API）；标准版实盘有每日次数保护 |

---

## 两种使用方式

| 优先级 | 方式 | 适用场景 |
|--------|------|----------|
| ⭐ **AI 交易** | **[MCP](./MCP使用说明.md)** `rhths-mcp.exe` | **OpenClaw、Hermes Agent**（本机 stdio）；笔记本 AI 连家里交易机（HTTP） |
| 量化 / 运维 | **[CLI](./CLI使用说明.md)** `rhths.exe` | 本机脚本、批处理、计划任务 |

---

## 核心组件

| 组件 | 说明 |
|------|------|
| **rhths-mcp.exe** | **AI 交易入口**（MCP → 网关 19312） |
| **RhThsHost.dll** | 同花顺进程内网关 `127.0.0.1:19312` |
| **rhths.exe** | CLI 量化 / 调试 |
| **rhths-gui.exe** | 安装 Hook、激活、日常管理 |
| **hook/** | 注入同花顺的部署文件 |

---

## 环境要求

- **Windows 10/11**
- **同花顺 PC 端**之一：免费版 / 金融大师 / 远航版 / 券商定制版 / 独立下单（xiadan），能登录交易即可
- **RHTHS** 已安装且 Hook 已部署（安装包默认流程或 `rhths-gui.exe`）——**无需任务 API**
- **AI 交易（可选）**：**OpenClaw / Hermes Agent / Cursor** + [MCP使用说明.md](./MCP使用说明.md)

---

## 快速开始

1. 启动同花顺并登录  
2. `rhths.exe health` → `ok: true`  
3. **AI 交易**：配置 OpenClaw 或 Hermes Agent → [MCP使用说明.md](./MCP使用说明.md)  
4. **量化脚本** → [CLI使用说明.md](./CLI使用说明.md)  
5. 首次安装 → [快速开始.md](./快速开始.md)

---

## 文档目录

| 文档 | 内容 |
|------|------|
| [MCP使用说明.md](./MCP使用说明.md) | **OpenClaw / Hermes Agent** 本机与局域网配置 |
| [快速开始.md](./快速开始.md) | 安装检查 |
| [CLI使用说明.md](./CLI使用说明.md) | 命令行参考 |
| [mcp.server.rhths-trade.json](./mcp.server.rhths-trade.json) | MCP 配置模板 |
| [mcp.tools.json](./mcp.tools.json) | 工具名与参数摘要 |

---

## 架构示意

```text
同花顺 (ths_api) ──► RhThsHost.dll :19312 ──► rhths-mcp.exe ──► OpenClaw / Hermes / Cursor（AI 交易）
                                            └──► rhths.exe      （CLI 量化）
```

---

## 许可与联系

- 遵守同花顺与券商规则；实盘风险自负；建议先模拟。  
- RHTHS 标准版/高级版在 GUI 激活（与「同花顺任务 API」无关）。  
- 技术支持：李先生 · 18670334431（微信同号）

---

> 面向**最终用户**与 **AI 助手**。开发构建见主仓库 `README.md`、`docs/`。
