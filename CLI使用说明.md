# CLI 使用说明（rhths.exe）

`rhths.exe` 是**本机**命令行客户端：通过安装目录旁的网关（默认 `http://127.0.0.1:19312`）访问**本机**同花顺。  

**不支持**在未安装同花顺的其它电脑上直接运行 CLI 去连远程交易（请改用 [MCP 局域网模式](./MCP使用说明.md#三局域网--远程安装mcp-http)）。

> **无需任何外部 API 授权**；**非 WinGUI 键鼠模拟** — CLI 经网关 **`ths_api` 标准调用**，极速稳定；**不含逆向破解、不破解同花顺**。合规见 [README.md](./README.md#技术与合规说明合法使用)。MCP 与 CLI 对照见 [SKILL.md](./SKILL.md)。



---



## 一、适用场景



| 适合 | 不适合 |

|------|--------|

| 本机 PowerShell / 批处理 / 计划任务 | 在 Mac、Linux 或未装同花顺的 PC 上跑 `rhths.exe` |

| 本机调试、对账、定时拉持仓 | 让云服务器直接执行交易（无同花顺则无网关） |

| 与 MCP 共用同一台「交易机」 | 替代 MCP 做跨机器 AI 对话（请用 MCP HTTP） |



---



## 二、准备（本机）



1. **同花顺**已在本机启动并登录  

2. `rhths.exe` 在 PATH 中，或使用完整路径，例如：



```powershell

$rhths = "D:\allinpol\RHTHS\dist\rhths.exe"

```



3. 检查连通：



```powershell

& $rhths health

& $rhths version

```



---



## 三、全局选项



所有子命令均支持：



| 选项 | 说明 |

|------|------|

| `--json` | 输出 JSON（默认开启） |

| `--pretty` | 格式化 JSON，便于阅读 |

| `--gateway-url URL` | 临时指定网关地址（一般保持 `127.0.0.1:19312`） |

| `--pipe` | 使用命名管道（高级，一般用 HTTP） |



环境变量（可在 PowerShell 中设置）：



| 变量 | 默认 | 说明 |

|------|------|------|

| `RHTHS_GATEWAY_URL` | `http://127.0.0.1:19312` | 网关地址（须指向**本机** RhThsHost，**不是** 19090） |

| `RHTHS_ALLOW_LIVE` | `0` | `1` 才允许实盘 |



---



## 四、系统命令



```powershell

rhths health          # 网关与同花顺 ths_api 状态

rhths version         # RHTHS 版本号

rhths catalog         # 网关路由列表

rhths license         # 授权/版本（标准版、高级版）信息

```



示例（可读输出）：



```powershell

rhths health --pretty

rhths positions --pretty

```



---



## 五、交易查询



```powershell

# 资金账号列表（多账号时）

rhths users



# 资金（可选 --user 账号 --rzrq 融资融券）

rhths account

rhths account --pretty



# 持仓

rhths positions



# 当日委托

rhths orders today

rhths orders today --active-only    # 仅未完成



# 历史委托（默认近 15 日、不含今天）

rhths orders history

rhths orders history --start-date 20250101 --end-date 20250115 --code 600000



# 预热缓存（批量查询前）

rhths warmup

```



---



## 六、行情与指标

```powershell
# 行情
rhths market quote 603919
rhths market price 603919 300033
rhths market kline 603919 --period 1440 --length 30
rhths market kline-15s 603919
rhths market kline-probe 603919
rhths market timeshare 603919
rhths market depth 603919
rhths market wencai "市盈率小于20"
rhths market wencai "市盈率小于20" --detail
rhths market screen "沪深A股;市盈率<30"
rhths market stock5d schema
rhths market stock5d factors 603919
rhths market moneyflow-probe 603919
rhths market blocks
rhths market block-stocks --block 行业板块名

# 订阅（需配合 wait-update）
rhths market reg-quote 603919 300033
rhths market reg-kline 603919 --period 1440 --length 30
rhths market wait-update --timeout 0.5
rhths market unreg-kline 603919

# 指标
rhths indicator list
rhths indicator calc 603919 MACD
```

子命令帮助：`rhths market --help`、`rhths indicator --help`

---

## 六点五、条件单与策略（进程内 ths_api）

```powershell
rhths condition schema
rhths condition add --codelist 000001 --condition "quote[code]['price'] > 0" --action "pass"
rhths condition resume --signal-id 20260524_00000001
rhths condition pause --signal-id 20260524_00000001
rhths condition delete --signal-id 20260524_00000001

# Pine / UI 信号单（JSON 体）
rhths condition run-json --body-json '{"type":"信号单","name":"测试",...}'

# 统一动作分发（与 thsQuant py/call 语义类似，走 :19312）
rhths py call --action condition.resume --params-json "{\"signal_id\":\"...\"}"
rhths py call --action market.stock5d --params-json "{\"code\":\"300033\"}"
```

---

## 六点六、同花顺进程（system）

```powershell
rhths system status
rhths system start-hexin
rhths system stop-hexin
rhths system start-xiadan
rhths system stop-xiadan
```

路径来自 `%AppData%\RHTHS\settings.json`（GUI 可配置）。

---

## 六点七、资金日快照与全部撤单

```powershell
rhths account-daily --start-date 20250101 --end-date 20250131
rhths cancel-all --dry-run
```

日快照保存在 `PythonLog\rhths\data\trade_account_daily.jsonl`（调用 `rhths account` 时默认也会追加当日记录）。

---

## 六点八、自动交易审核（本地 JSON，非 MySQL）

```powershell
rhths autotrading config-get
rhths autotrading config-set --mode review
rhths autotrading review-list --status pending
rhths autotrading review-approve --review-id xxx
rhths autotrading audit-logs --limit 100
rhths autotrading exchange-orders
```

数据目录：`PythonLog\rhths\data\`（与网关日志同级）。

---

## 七、下单与撤单



**默认均为模拟（dry-run）**，不会真实成交。



### 模拟卖出 / 买入



```powershell

rhths sell 603919 --qty 100 --dry-run

rhths buy 603919 --qty 100 --price zxjg --dry-run

rhths cancel --scope all --dry-run
rhths cancel-all --dry-run

```



### 原始命令（高级）



```powershell

rhths invoke "sell 603919 dsj3 100 -notip" --dry-run

```



### 实盘（需显式开启，风险自负）



```powershell

$env:RHTHS_ALLOW_LIVE = "1"

rhths sell 603919 --qty 100 --live --confirm

rhths buy 600000 --qty 100 --price zxjg --live --confirm

```



| 参数 | 说明 |

|------|------|

| `--dry-run` | 模拟（默认 true） |

| `--live` | 实盘（须配合环境变量与 `--confirm`） |

| `--confirm` | 确认实盘 |

| `--qty` | 数量 |

| `--price` | 价格模式；买常用 `zxjg`，卖常用 `dsj3` |

| `--user` | 资金账号（可选） |

| `--rzrq` | 融资融券 |



**标准版**：实盘买入+卖出合计每天最多 **10 次**；超限会报错。  

**高级版**：在 `rhths-gui` 激活注册码后无次数限制。



---



## 八、输出与脚本配合



默认 JSON，便于 PowerShell 解析：



```powershell

rhths account --json | ConvertFrom-Json

rhths positions --pretty > pos.txt

```



在批处理 / 计划任务中，先 `rhths health`，成功后再执行业务命令。



---



## 九、与 MCP 的区别



| | CLI | MCP |

|---|-----|-----|

| 谁调用 | 你、本机脚本 | Cursor、OpenClaw、Hermes Agent 等 AI |

| 程序 | `rhths.exe` | `rhths-mcp.exe` |

| 运行位置 | **仅交易机本机** | 本机（stdio）或任意电脑连交易机（HTTP） |

| 典型用途 | 自动化、调试、定时 | 对话式查资金、持仓、下单 |



二者最终都访问**同一台交易机**上的网关 `19312`；CLI 必须在该机上执行，MCP 可在其它机器通过 HTTP 间接访问。



---



## 十、常见问题



| 现象 | 处理 |

|------|------|

| 连接拒绝 | 启动同花顺；GUI 部署 Hook 后重启 xiadan |

| `LIVE_BLOCKED` | 设置 `RHTHS_ALLOW_LIVE=1` 并加 `--live --confirm` |

| 升级 Hook 后命令失效 | 完全退出 xiadan 再启动 |

| JSON 乱码 | 使用 `--pretty` 或 `ConvertFrom-Json` |

| 想在另一台电脑用命令行 | 不能远程跑 CLI；请用 [MCP HTTP](./MCP使用说明.md#三局域网--远程安装mcp-http) |



安装与 Hook 部署见产品安装包说明；MCP 配置见 [MCP使用说明.md](./MCP使用说明.md)。

