# RHTHS

**同花顺 PC 用户的 AI 交易与量化网关** — **非传统 WinGUI 键鼠模拟**，采用同花顺进程内**原生 `ths_api`**，**高效、极速、稳定**；适配免费版 / 金融大师 / 远航版 / 券商定制版 / 独立下单等，**无需任何外部 API 授权**，安装即用。专为 **OpenClaw**、**Hermes Agent** 及 Cursor 做 AI 交易，并支持 **CLI** 量化脚本。

在线文档：[https://www.miaolink.cn/rhths/index.php](https://www.miaolink.cn/rhths/index.php) · MCP 配置详见 [MCP使用说明.md](./MCP使用说明.md)

---

## 技术路线：原生 `ths_api`，不是 WinGUI 自动化

RHTHS **不走**市面上常见的「找窗口 → 模拟点击 → OCR 读屏」等 **WinGUI / 外挂式** 方案，而是在同花顺进程内通过 Hook 直接调用官方策略环境自带的 **`ths_api`**，再由本机网关转发给 MCP / CLI。

| | 传统 WinGUI 自动化 | RHTHS（原生 `ths_api`） |
|---|---------------------|-------------------------|
| 实现方式 | 模拟鼠标键盘、控件坐标、图像识别 | **进程内 API 调用**，与策略脚本同源 |
| 速度 | 慢（界面操作、等待渲染） | **极速**（毫秒级，无界面往返） |
| 稳定性 | 分辨率/皮肤/弹窗一变就易失效 | **稳定**（不依赖窗口布局） |
| 能力边界 | 常受界面步骤与脚本条数限制 | **与 PC 同花顺策略环境一致**，查询/模拟无额外接口配额；无「按次购买 API 调用量」 |
| 数据一致性 | 读屏/抓控件易偏差 | 资金、持仓、委托与**客户端内数据一致** |

> **说明：**「无限制」指**不依赖 WinGUI 自动化套路、不设第三方 API 调用配额**；查询与模拟下单在网关层不限次数。实盘仍须遵守券商与同花顺规则；RHTHS **标准版**对实盘买卖有每日次数保护（高级版无此限制），与「是否 WinGUI」无关。

```text
❌ 传统：AI/脚本 ──► 模拟点击同花顺窗口（慢、脆、难维护）
✅ RHTHS：AI/脚本 ──► 网关 19312 ──► 同花顺进程内 ths_api（快、稳、原生）
```

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

以下 **同花顺 PC 端** 均可使用 RHTHS（只要你能用该软件**正常登录并交易**即可，**不必**先申请任何外部 API 授权）：

| 客户端类型 | 说明 |
|------------|------|
| **PC 免费版** | 个人常用免费客户端 |
| **金融大师** | 完整行情与交易环境 |
| **远航版** | 远航系列 PC 客户端 |
| **券商定制版** | 各券商冠名、定制的同花顺 PC 版 |
| **独立下单（xiadan）** | 同花顺独立下单程序，与主客户端配合或单独使用 |

> **统一说明：不需要开通任何外部 API 授权**（含同花顺官方量化接口、券商独立交易 API、第三方云交易 API 等）。RHTHS 在你**本机已登录**的同花顺/下单进程内使用内置 `ths_api`，不是再去申请一套 API Key。

### 默认安装后可用（三步）

1. 安装 **RHTHS** 安装包（含 `rhths-gui.exe`、`rhths.exe`、`rhths-mcp.exe`）  
2. 在 GUI 中执行 **「安装 / 更新 Hook」**（或运行安装包自带部署脚本）  
3. **启动同花顺（或独立下单）并登录** → 运行 `rhths health` 显示正常 → 即可配置 **OpenClaw / Hermes Agent** 或运行 **CLI**

无需等待外部 API 审核、无需改券商通道、无需更换交易软件品牌。

---

## 谁可以直接用？无需任何外部 API 授权

> **凡使用上表任一同花顺 PC 客户端、且已能手动登录下单的用户：安装 RHTHS 并完成 Hook 后即可做 AI 交易与量化——无需向同花顺、券商或第三方申请/开通任何 API 授权。**

| 传统做法 | RHTHS 做法 |
|----------|------------|
| WinGUI 模拟点击、易碎且慢 | **原生 `ths_api`**，高效极速稳定 |
| 向同花顺/券商申请量化或交易 API、等待审核 | **默认安装 + Hook**，沿用现有 PC 客户端与账户 |
| 为 AI 单独开发交易接口 | **OpenClaw / Hermes Agent** 配 MCP 即可对话查仓、下单 |
| 云端 API 与 PC 持仓不一致 | 走**本机同花顺内置 `ths_api`**，与软件里看到的一致 |

**不需要：** 任何外部 API 开通、第三方云交易接口、迁移到其它交易系统。

> 说明：文档中的 `ths_api` 指同花顺客户端进程内的 Python 交易接口，**不是**要你额外申请的「官方开放 API」。

建议先用 **模拟（dry-run）** 验证 AI 与策略，再按需实盘（风险自负）。

---

## 软件能做什么

| 能力 | 说明 |
|------|------|
| **原生 `ths_api` 网关** | 非 WinGUI；进程内调用，极速稳定，查询/模拟无第三方 API 配额限制 |
| **AI 交易（MCP）** | **OpenClaw、Hermes Agent** 等通过对话查资金/持仓/委托、问财选股、模拟或实盘买卖 |
| **量化脚本（CLI）** | `rhths.exe` 供 PowerShell、计划任务、Python 等本机自动化 |
| **交易查询** | 资金账号、资金、持仓、当日/历史委托 |
| **下单与撤单** | 买入、卖出、撤单；默认模拟，实盘需 `confirm_live` |
| **行情与选股** | 实时行情、K 线、问财、指标（如 MACD） |
| **图形管理** | `rhths-gui.exe`：Hook、交易面板、激活、升级 |
| **产品授权** | RHTHS 标准版/高级版（产品注册码，≠ 外部 API 授权）；标准版实盘有每日次数保护 |

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
- **RHTHS** 已安装且 Hook 已部署——**无需任何外部 API 授权**
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
| [SKILL.md](./SKILL.md) | **Agent 融合指南**（MCP + CLI，供 Cursor / OpenClaw / Hermes 加载） |
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

## 技术与合规说明（合法使用）

RHTHS 在技术与合规上坚持以下原则，请用户知悉并合规使用：

| 项目 | 说明 |
|------|------|
| **不含逆向破解** | **不对**同花顺客户端进行逆向工程、脱壳、篡改、外挂注入式破解，**不涉及**对「任务」模块或交易核心的非法破解 |
| **标准调用** | 在**已安装且由用户正常启动、登录**的同花顺/独立下单进程内，通过官方策略环境提供的 **`ths_api` 标准接口** 完成查询与下单，调用方式与 PC 端策略脚本同类，属于**标准 API 调用**而非模拟破解协议 |
| **不破解软件** | **不破解、不绕过**同花顺软件授权与安全机制；不修改同花顺安装包、不提供盗版或破解补丁 |
| **Hook 用途** | 本地 Hook 仅用于在同花顺进程内加载 **RhThsHost** 网关模块，将 `ths_api` 能力以本机 HTTP 方式暴露给 CLI/MCP，**目的为接入与扩展，而非破坏软件** |
| **用户责任** | 用户须遵守**同花顺用户协议**、**券商交易规则**及所在地法律法规；实盘风险自负，建议先模拟验证 |

> 若同花顺或券商对自动化、程序化交易有额外规定，请以官方最新条款为准；RHTHS 不提供任何规避监管或违规交易的用途。

---

## 许可与联系

- **产品许可**：RHTHS 标准版/高级版在 GUI 激活（与「外部 API 授权」无关）。  
- **合规**：请在上表前提下使用；禁止将本产品用于破解、盗版、未授权访问他人账户等违法行为。  
- **免责声明**：资金与交易风险由用户自行承担，详见本站页面底部 [免责声明](https://www.miaolink.cn/rhths/index.php)（PHP 展示）。  
- **技术支持**：李先生 · 18670334431（微信同号）

---

> 面向**最终用户**与 **AI 助手**。开发构建见主仓库 `README.md`、`docs/`。
