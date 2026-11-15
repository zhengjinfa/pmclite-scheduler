# pmclite-scheduler

MCP server for production scheduling in small and mid-size factories. It connects an AI assistant (Claude Desktop, Cursor, Cline, or any MCP-capable client) to the scheduling engine inside **MRP-PMC**, a Windows desktop app for production planning and material control.

> Local-first and stdio-only. The server runs on your own machine next to the desktop app and reads its local encrypted database. Nothing is uploaded to a cloud service.

## Requirements

- Windows 10/11 (x64)
- MRP-PMC desktop app — download from https://www.pmclite.com
- Any MCP client that supports stdio servers

## Install

1. Install MRP-PMC from https://www.pmclite.com and open it once, so it creates its local database.

### Claude Code: one command

Replace `<your-user>` with your Windows user name, then run it in a terminal:

```bash
claude mcp add --env NO_PROXY=127.0.0.1,localhost pmclite -- "C:\Users\<your-user>\AppData\Local\MRP-PMC\MRP-PMC.exe" mcp-server
```

`claude mcp list` should now list `pmclite`. Every flag (`--env`, `--transport`, `--scope`) has to come **before** the server name; whatever follows `--` is the command the client launches.

### Any other MCP client: JSON config

2. Merge this entry into your existing `mcpServers` (do not replace the whole file):

```json
{
  "mcpServers": {
    "pmclite-scheduler": {
      "command": "C:/Users/<you>/AppData/Local/MRP-PMC/MRP-PMC.exe",
      "args": ["mcp-server"],
      "env": {
        "PMCLITE_DATA_DIR": "C:/Users/<you>/AppData/Roaming/com.dell.MRP-PMC"
      }
    }
  }
}
```

3. Restart the client and ask something like `今天有哪些工单要开工？`.

The executable path depends on how the app was installed; the default per-user install is `%LOCALAPPDATA%\MRP-PMC\MRP-PMC.exe`. `PMCLITE_DATA_DIR` points at the folder holding the app's database.

## Tools

| Tool | What it does |
| --- | --- |
| `get_dashboard` | Today's board, KPI and labor view (`scope=today\|kpi\|labor\|all`, `days`) |
| `query_work_orders` | Query work orders by status, line or date (`include=["kit"]` adds kit status) |
| `query_master_data` | Materials, BOM and routing master data |
| `estimate_hours` | Estimate work hours for a work order, product or batch |
| `check_readiness` | Kit readiness check at a given sequence position (`at_sequence`) |
| `preview_schedule` | Dry-run a schedule without writing it back |
| `apply_schedule` | Commit a previewed schedule to the database |
| `reschedule_from_actuals` | Re-plan from reported actuals |
| `report_progress` | Report completed quantity for a work order |
| `report_incident` | Log a downtime or material incident |
| `query_records` | Read back reported progress and incidents (`type=report\|incident`) |
| `evaluate_order` | Promise-date check for a new order |
| `assess_insertion` | What a rush order costs the existing plan (`max_shift_days`, default 3) |
| `get_import_schema` | Field schema for bulk import |
| `import_data` | Import master data or work orders |

## Example prompts

- `今天的生产情况怎么样？`
- `WO-001 现在齐套吗？缺什么料？`
- `这个急单插进去，会让哪些工单延期几天？`
- `按实际报工重排一下剩下的工单`

## Data and privacy

- The vault is a local SQLCipher database; the server goes through the app's own data layer.
- No network calls to vendors or third parties. AI traffic is only between your MCP client and the local process.
- Reads are safe while the app is open. Write operations (apply schedule, report progress, import) are committed by the app, which should be idle or closed.

## Limitations

- Windows only: the server drives a desktop application, so it cannot run in a Linux container or a hosted sandbox.
- stdio transport only — there is no public HTTP endpoint to connect remotely.
- Multi-user and cloud sync are not part of this server.

## Links

- Product site: https://www.pmclite.com
- Skill bundle for AI agents: https://github.com/zhengjinfa/pmclite-skills
- Vendor: 广州轻排信息科技有限公司 (Guangzhou Qingpai Information Technology Co., Ltd.)

## 中文说明

轻排（MRP-PMC）生产排产的 MCP 服务端，用 stdio 协议把 AI 助手接到本地排产引擎上，覆盖工单查询、工时估算、齐套检查、排程预演与落库、按实绩重排、报工、异常上报、交期评审、插单测算和数据导入共 15 个工具。

- 仅支持 Windows 10/11：服务端驱动的是桌面程序，无法在 Linux 容器或云端沙箱中运行。
- 数据不出本机：本地加密库，经程序自己的数据层访问。
- 读操作在程序运行时可用；写操作（落库排程、报工、导入）建议先让程序空闲或关闭。

安装：先到 https://www.pmclite.com 安装 MRP-PMC 并打开一次。Claude Code 用一行命令（注意选项写在服务器名之前，`--` 之后才是启动命令）：

```bash
claude mcp add --env NO_PROXY=127.0.0.1,localhost pmclite -- "C:\Users\<用户名>\AppData\Local\MRP-PMC\MRP-PMC.exe" mcp-server
```

装完 `claude mcp list` 里应出现 `pmclite`。其他客户端把上面 `mcpServers` 那段合并进已有配置即可。