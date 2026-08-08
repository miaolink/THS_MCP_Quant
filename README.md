# RHTHS

> 文中 **THS** 均指相关 PC 行情交易客户端（免费版 / 金融大师 / 远航版 / 券商定制版 / 独立下单 xiadan 等）。

**THS PC 用户的 AI 交易与量化网关** — 在用户本机已登录的 THS 策略 / 下单 Python 环境中调用 **`ths_api`**，经本机网关提供给 **OpenClaw**、**Hermes Agent**、Cursor 等做 AI 交易，并支持 **CLI** 量化脚本。适配常见 PC 客户端（免费版 / 金融大师 / 远航版 / 券商定制版 / 独立下单等）。

> **说明：** RHTHS 为第三方辅助工具，**非 THS 官方产品**，未声称获得 THS 特别授权。能否使用取决于你本机 THS 账户与策略 / 交易环境是否已可用；请遵守 THS 用户协议、券商规则及当地法律法规。

在线文档：[https://www.miaolink.cn/rhths/index.php](https://www.miaolink.cn/rhths/index.php) · MCP 配置详见 [MCP使用说明.md](./MCP使用说明.md)

## 说人话

若你本机 THS 已能正常登录并交易（含模拟），且策略 / 交易相关 Python 环境可用，安装 RHTHS 并完成扩展部署后，即可用 MCP / CLI 查仓与下单。具体权限以 THS 与券商实际开通情况为准。

## 技术路线：进程内 `ths_api`（非读屏点选）

RHTHS 主要路径**不是**「找窗口 → 模拟点击 → OCR 读屏」，而是在 THS / 独立下单进程内通过 Python **`import ths_api`**（与策略脚本同类用法）完成查询与下单，再由本机网关转发给 MCP / CLI。

| | 常见界面自动化 | RHTHS（进程内 `ths_api`） |
|---|----------------|---------------------------|
| 实现方式 | 模拟鼠标键盘、控件坐标、图像识别 | Python 调用策略环境中的 `ths_api` |
| 速度 | 慢（界面操作、等待渲染） | 较快（无界面往返） |
| 稳定性 | 分辨率 / 皮肤 / 弹窗变化易失效 | 不依赖窗口布局 |
| 数据一致性 | 读屏 / 抓控件易偏差 | 资金、持仓、委托与客户端内数据一致 |

> **说明：** 查询与模拟下单在网关层不设第三方「按次 API 配额」。实盘须遵守券商与 THS 规则；RHTHS**标准版**对实盘买卖有每日次数保护（高级版无此限制）。

```text
常见界面自动化：AI/脚本 ──► 模拟点击 THS 窗口
RHTHS：AI/脚本 ──► 网关 19312 ──► 进程内 ths_api
```

---

## 联系作者（优先微信）

安装 RHTHS、扩展部署、MCP / 局域网连不上、标准版 / 高级版激活与购买等问题，**请优先联系作者**，微信通常比自行排查更快。

| 项目 | 内容 |
|------|------|
| **联系人** | 李先生 |
| **手机** | [18670334431](tel:18670334431) |
| **微信** | 同上号码（**推荐**：添加时请备注「RHTHS」+ 简要问题） |

<img src="./weixin.jpg" alt="李先生微信二维码（长按或扫码添加）" width="260" />

> 说明：交易与资金风险由用户自行承担；技术咨询仅限 RHTHS 安装、配置与产品授权，不提供代客理财或投资建议。

## AI 交易（首选：OpenClaw · Hermes Agent）

在 **OpenClaw** 或 **Hermes Agent** 里配置 RHTHS 的 MCP 服务 **`rhths-trade`** 后，用自然语言即可驱动 THS 账户（默认**模拟下单**，实盘须显式确认）：

| 你可以对 AI 说 | MCP 背后能力 |
|----------------|--------------|
| 「帮我看下资金和持仓」 | `rh_trade_account`、`rh_trade_positions` |
| 「今天有哪些委托」 | `rh_trade_orders_today` |
| 「模拟买入 100 股某某，最新价」 | `rh_trade_buy`（`dry_run`） |
| 「问财选市盈率小于 20 的票」 | `rh_market_wencai`、`rh_market_select_stocklist` |
| 「恢复策略运行 / 条件单 resume」 | `rh_condition_resume`（需已有 `signal_id`） |
| 「启动下单程序 xiadan」 | `rh_system_start_xiadan` |
| 「切换到实盘模式」（需谨慎） | `rh_trade_mode_set` + `confirm_live` |

完整话术与**全部 MCP 工具**示例见 **[AI交易示例.md](./AI交易示例.md)**。

**推荐客户端（配置方式相同，合并 `rhths-trade` 即可）：**

1. **OpenClaw / Claw** — 自主 Agent，本机 stdio 或局域网 HTTP 连交易机 
2. **Hermes Agent** — MCP Agent，本机命令或远程 URL 
3. **Cursor**、Claude Desktop、Windsurf、Cline、Continue、Cherry Studio 等

**机读配置模板：** [mcp.server.rhths-trade.json](./mcp.server.rhths-trade.json) · 工具列表：[mcp.tools.json](./mcp.tools.json) · **AI 自动安装：** [AI自动安装MCP与SKill示例.md](./AI自动安装MCP与SKill示例.md)

```text
你 ──对话──► OpenClaw / Hermes Agent ──MCP──► rhths-mcp.exe ──► THS (ths_api)
 查仓 · 行情 · 下单（可先模拟）
```

**RHTHS** 在本机 THS 进程内对接 `ths_api`，对外暴露统一网关；**AI 交易走 MCP**，**量化脚本走 CLI**，二者共用同一套交易语义。

---

## 适配哪些 THS ？

以下 **THS PC 端** 在「能登录并交易、且本机策略 / 交易 Python 环境可用」的前提下，可与 RHTHS 配合使用：

| 客户端类型 | 说明 |
|------------|------|
| **PC 免费版** | 个人常用免费客户端 |
| **金融大师** | 完整行情与交易环境 |
| **远航版** | 远航系列 PC 客户端 |
| **券商定制版** | 各券商冠名、定制的 THS PC 版 |
| **独立下单（xiadan）** | THS 独立下单程序，与主客户端配合或单独使用 |

> RHTHS 在你**本机已登录**的 THS / 下单进程内使用环境自带的 `ths_api`，**不是**另申请一套云端交易 API Key。 THS 或券商若另有程序化、策略权限要求，以官方规定为准。

### 安装后三步

1. 安装 **RHTHS** 安装包（含 `rhths-gui.exe`、`rhths.exe`、`rhths-mcp.exe`） 
2. 在 GUI 中执行 **「安装 / 更新扩展」**（或运行安装包自带部署脚本） 
3. **启动 THS（或独立下单）并登录** → 运行 `rhths health` 显示正常 → 即可配置 **OpenClaw / Hermes Agent** 或运行 **CLI**

---

## 谁可以直接用？

> 使用上表任一 THS PC 客户端、且已能手动登录下单、本机策略 / 交易环境可用的用户：安装 RHTHS 并完成扩展部署后，可用 MCP / CLI 做查询与下单。

| 常见做法 | RHTHS 做法 |
|----------|------------|
| 界面模拟点击 | 进程内 Python 调用 `ths_api` |
| 另申请云端量化 / 交易 API | 沿用本机已登录的 PC 客户端与账户 |
| 为 AI 单独开发交易接口 | **OpenClaw / Hermes Agent** 配 MCP 即可对话查仓、下单 |
| 云端持仓与 PC 不一致 | 走本机 THS 内 `ths_api`，与软件里看到的一致 |

**不提供：** 第三方云交易接口托管、迁移到其它交易系统品牌、**MySQL 或其它数据库同步**（属其它非标集成）。

> **与旧 thsQuant 的关系：** RHTHS**不连接** `:19090` / `:18989`。若你曾用 thsQuant，请改用本机 **`http://127.0.0.1:19312`** 与 `rhths` / `rhths-mcp`；两套程序可并存但互不依赖。

建议先用 **模拟（dry-run）** 验证 AI 与策略，再按需实盘（风险自负）。

---

## 软件能做什么

| 能力 | 说明 |
|------|------|
| **`ths_api` 网关** | 进程内 Python 调用；查询 / 模拟无第三方按次配额 |
| **AI 交易（MCP）** | **OpenClaw、Hermes Agent** 等通过对话查资金 / 持仓 / 委托、问财选股、模拟或实盘买卖 |
| **量化脚本（CLI）** | `rhths.exe` 供 PowerShell、计划任务、Python 等本机自动化 |
| **交易查询** | 资金账号、资金、持仓、当日 / 历史委托 |
| **下单与撤单** | 买入、卖出、撤单；默认模拟，实盘需 `confirm_live` |
| **行情与选股** | 实时行情、K 线、分时、盘口、板块、问财、5 日因子、指标 |
| **条件单 / 策略** | `ths.run` / `resume` 等（进程内信号引擎，经网关 `:19312`） |
| **进程管理** | 启动 / 停止 `hexin.exe`、`xiadan.exe`（CLI / MCP / GUI） |
| **自动交易审核** | 本地 JSON 配置与审核队列（可选，非 MySQL） |
| **图形管理** | `rhths-gui.exe`：扩展部署、交易面板、激活、升级 |
| **产品授权** | RHTHS 标准版 / 高级版（产品注册码）；标准版实盘有每日次数保护 |

---

## 两种使用方式

| 优先级 | 方式 | 适用场景 |
|--------|------|----------|
| **AI 交易** | **[MCP](./MCP使用说明.md)** `rhths-mcp.exe` | **OpenClaw、Hermes Agent**（本机 stdio）；笔记本 AI 连家里交易机（HTTP） |
| 量化 / 运维 | **[CLI](./CLI使用说明.md)** `rhths.exe` | 本机脚本、批处理、计划任务 |

---

## 核心组件

| 组件 | 说明 |
|------|------|
| **rhths-mcp.exe** | **AI 交易入口**（MCP → 网关 19312） |
| **RhThsHost.dll** | THS 进程内网关（默认本机 `127.0.0.1:19312`） |
| **rhths.exe** | CLI 量化 / 调试 |
| **rhths-gui.exe** | 扩展部署、激活、日常管理 |
| **扩展模块** | 部署到 THS 策略环境的网关模块 |

---

## 环境要求

- **Windows 10/11**
- **THS PC 端**之一：免费版 / 金融大师 / 远航版 / 券商定制版 / 独立下单（xiadan），能登录交易，且策略 / 交易 Python 环境可用
- **RHTHS** 已安装且扩展已部署
- **AI 交易（可选）**：**OpenClaw / Hermes Agent / Cursor** + [MCP使用说明.md](./MCP使用说明.md)

### 独立下单（xiadan.exe）必设项（唯一配置入口）

RHTHS 通过 `ths_api` 程序化下单时，**必须在 `xiadan.exe` 里完成下图设置**（其它客户端无此替代路径）。打开方式：独立下单窗口顶部菜单 **「设置」** → 选项卡 **「快速交易」**，将图中标注的 **4 项全部设为「否」**：

| 设置项 | 必须值 |
|--------|--------|
| 撤单前是否需要确认 | **否** |
| 买入时是否需要确认 | **否** |
| 卖出时是否需要确认 | **否** |
| 委托成功后是否弹出提示对话框 | **否** |

![xiadan 快速交易：买入/卖出/撤单确认与委托成功弹窗须设为「否」](./ScreenShot_2026-05-24_165949_700.png)

> **说明：** 若上述任一项为「是」，程序化委托可能被确认框或弹窗阻塞，导致 MCP/CLI 下单超时或失败。修改后点击 **「确定」** 保存，并**完全退出后重新启动 `xiadan.exe`**（部署扩展 后亦须重启）。

---

## 快速开始

1. 启动 THS 并登录 
2. `rhths.exe health` → `ok: true` 
3. **AI 交易**：配置 OpenClaw 或 Hermes Agent → [MCP使用说明.md](./MCP使用说明.md) · 话术 [AI交易示例.md](./AI交易示例.md) 
4. **量化脚本** → [CLI使用说明.md](./CLI使用说明.md) 
5. 首次安装 → [快速开始.md](./快速开始.md)

---

## 文档目录

| 文档 | 内容 |
|------|------|
| [软件功能图.md](./软件功能图.md) | **控制台界面截图**（概览 / 交易 / 激活 / 设置 / MCP 等功能说明） |
| [AI交易示例.md](./AI交易示例.md) | **OpenClaw / Hermes** 自然语言话术与全工具示例 |
| [SKILL.md](./SKILL.md) | **Agent 融合指南**（MCP + CLI，供 OpenClaw / Hermes 加载） |
| [MCP使用说明.md](./MCP使用说明.md) | **OpenClaw / Hermes Agent** 本机与局域网配置 |
| [AI自动安装MCP与SKill示例.md](./AI自动安装MCP与SKill示例.md) | **一句话让 AI 自动安装 MCP + Skill**（本机 / 局域网） |
| [快速开始.md](./快速开始.md) | 安装检查 |
| [CLI使用说明.md](./CLI使用说明.md) | 命令行参考 |
| [mcp.server.rhths-trade.json](./mcp.server.rhths-trade.json) | MCP 配置模板 |
| [mcp.tools.json](./mcp.tools.json) | 工具名与参数摘要 |

---

## 架构示意

```text
 THS (ths_api) ──► RhThsHost / 网关 :19312 ──► rhths-mcp.exe ──► OpenClaw / Hermes / Cursor（AI 交易）
 └──► rhths.exe （CLI 量化）
```

---

## 使用说明与合规提示

请用户知悉并合规使用：

| 项目 | 说明 |
|------|------|
| **产品性质** | 第三方辅助工具，非 THS 官方出品；不修改 THS 安装包、不提供盗版或破解补丁 |
| **技术路径** | 在用户正常启动并登录的 THS / 独立下单进程内，通过策略环境 Python 调用 `ths_api`，再以本机 HTTP 提供给 CLI / MCP |
| **扩展部署** | GUI「安装 / 更新扩展」将 RHTHS 模块部署到本机 THS 策略环境，可卸载；不替代 THS 软件授权 |
| **用户责任** | 须遵守**THS 用户协议**、**券商交易规则**及所在地法律法规；实盘风险自负，建议先模拟验证 |
| **禁止用途** | 代客理财、非法荐股、操纵市场、侵入他人账户、规避监管或未授权远程控制他人交易机等 |

> 若 THS 或券商对自动化、程序化交易有额外规定，请以官方最新条款为准。

---

## 许可与免责

- **产品许可**：RHTHS 标准版 / 高级版在 GUI 激活。 
- **合规**：禁止将本产品用于盗版、未授权访问他人账户等违法行为。 
- **免责声明**：资金与交易风险由用户自行承担，详见本站页面底部 [免责声明](https://www.miaolink.cn/rhths/index.php)。 
- **技术支持**：李先生 · [18670334431](tel:18670334431)（微信同号，二维码见文首 [联系作者](#联系作者优先微信)）

---

> 面向**最终用户**与 **AI 助手**。开发构建见主仓库 `README.md`、`docs/`。
