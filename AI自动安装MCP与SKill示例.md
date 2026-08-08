# AI 自动安装 MCP 与 Skill 示例

> **你只需要两件事：** ① 复制下面「一句话」发给 AI；② 告诉 AI **MCP 服务 IP**。 
> **安装步骤由 AI 自己学：** [GitHub — miaolink/THS_MCP_Quant](https://github.com/miaolink/THS_MCP_Quant)（与本地 `web/usermd` 文档一致）。

---

## MCP 服务 IP 怎么填（只记这一条）

| 你填的 IP | 含义 | AI 应采用的连接方式 |
|-----------|------|---------------------|
| **`127.0.0.1`** | AI 与交易在**同一台**装 THS 的电脑 | 本机 **stdio**（`rhths-mcp.exe` + `args: ["stdio"]`） |
| **`192.168.x.x`**（如 `192.168.100.168`） | AI 在别的电脑，交易机在家/办公室局域网 | 远程 **HTTP**：`http://该IP:19310/mcp`（交易机须已开 `rhths-gui` 或 `rhths-mcp.exe http`） |

> 交易机上的 THS 网关始终是 **`http://127.0.0.1:19312`**（仅交易进程内，与「MCP 服务 IP」不是同一个端口）。 
> MCP 服务名固定：**`rhths-trade`**。

**交易机前提（一次性）：**THS 已登录、RHTHS 扩展已部署、`rhths health` 正常；xiadan 快速交易 4 项须为「否」见 [README.md](./README.md#独立下单xiadanexe必设项唯一配置入口)。

---

## 一句话安装（复制即用）

把 **`【MCP服务IP】`** 换成 `127.0.0.1` 或你的交易机局域网 IP（如 `192.168.100.168`）；**`【客户端】`** 换成 OpenClaw / Hermes / Cursor 等。

```text
请打开并学习 GitHub 仓库 https://github.com/miaolink/THS_MCP_Quant ，按其中的 MCP使用说明.md、mcp.tools.json、mcp.server.rhths-trade.json、SKILL.md 自动帮我完成 rhths-trade 的 MCP 安装和 Agent Skill 安装。

我的 MCP 服务 IP：【MCP服务IP】
我使用的 AI 客户端：【客户端】

要求：你自己读仓库文档决定用 stdio 还是 HTTP；服务名必须是 rhths-trade；默认 RHTHS_ALLOW_LIVE=0（仅模拟）；装完后调用 rh_trade_health 验证，并用中文告诉我是否成功。若 IP 是 127.0.0.1 请在本机搜索 rhths-mcp.exe 并写入配置；若是局域网 IP 请先测 http://该IP:19310/health。
```

### 更短版（只要 IP + 客户端）

```text
按 https://github.com/miaolink/THS_MCP_Quant 文档给我装 rhths-trade MCP 和 Skill，MCP 服务 IP=【MCP服务IP】，客户端=【客户端】，装完 rh_trade_health 自检。
```

---

## 两个填好的例子（直接发）

**本机（THS 与 AI 同一台电脑）：**

```text
按 https://github.com/miaolink/THS_MCP_Quant 文档给我装 rhths-trade MCP 和 Skill，MCP 服务 IP=127.0.0.1，客户端=Hermes Agent，装完 rh_trade_health 自检。
```

**局域网（交易机 IP 示例）：**

```text
按 https://github.com/miaolink/THS_MCP_Quant 文档给我装 rhths-trade MCP 和 Skill，MCP 服务 IP=192.168.100.168，客户端=OpenClaw，装完 rh_trade_health 自检。
```

---

## 装好后验证（可选第二句）

```text
请用 rhths-trade 调用 rh_trade_health、rh_trade_account、rh_trade_positions，用中文总结；失败则按仓库里 MCP使用说明.md 排障。
```

---

## 给 AI 的仓库链接（一般不用你手动发）

AI 可自行打开或拉取以下文件（与 [THS_MCP_Quant](https://github.com/miaolink/THS_MCP_Quant) 根目录一致）：

| 文件 | 链接 |
|------|------|
| 总览 | [README.md](https://github.com/miaolink/THS_MCP_Quant/blob/main/README.md) |
| MCP 安装 | [MCP使用说明.md](https://github.com/miaolink/THS_MCP_Quant/blob/main/MCP%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.md) |
| 工具表 | [mcp.tools.json](https://github.com/miaolink/THS_MCP_Quant/blob/main/mcp.tools.json) |
| 配置模板 | [mcp.server.rhths-trade.json](https://github.com/miaolink/THS_MCP_Quant/blob/main/mcp.server.rhths-trade.json) |
| Skill | [SKILL.md](https://github.com/miaolink/THS_MCP_Quant/blob/main/SKILL.md) |
| 话术示例 | [AI交易示例.md](https://github.com/miaolink/THS_MCP_Quant/blob/main/AI%E4%BA%A4%E6%98%93%E7%A4%BA%E4%BE%8B.md) |

---

## 自检清单（AI 做完你对一下）

- [ ] 配置里已有 **`rhths-trade`**
- [ ] IP=`127.0.0.1` → 本机 **stdio**；IP=局域网地址 → **`url`** 指向 `:19310`
- [ ] 已安装 **SKILL.md**（OpenClaw / Hermes 技能目录）
- [ ] **`rh_trade_health`** 返回正常
- [ ] 未擅自开启实盘

---

## 仍连不上时（再发一句）

```text
MCP 服务 IP 是【你的IP】，请按 https://github.com/miaolink/THS_MCP_Quant/blob/main/MCP%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.md 第七节帮我排障并改好配置。
```

**本地完整文档（与 GitHub 同步）：** [MCP使用说明.md](./MCP使用说明.md) · [快速开始.md](./快速开始.md) · 在线 https://www.miaolink.cn/rhths/index.php
