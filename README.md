# DevOrbit MCP Server

> 14 tools, dual protocol (2025-06-18 / 2025-11-25), case-scoped audit, identity proxy

## 服务配置

```json
{
  "mcpServers": {
    "devorbit": {
      "type": "streamable_http",
      "url": "https://studio-rain777-devorbit.api-inference.modelscope.net/mcp"
    }
  }
}
```

> 直连地址：`http://8.154.25.108:4174/mcp`（本地环境，无需鉴权）
> 魔搭托管地址：`https://studio-rain777-devorbit.api-inference.modelscope.net/mcp`（需 ModelScope Token 认证）
> Worker MCP 调用经身份代理注入 caller/caseId/traceId

## 工具列表

| 工具 | 说明 |
|------|------|
| `issue.fetch_signals` | 从 Issue 系统拉取故障信号（intake-worker） |
| `observability.fetch_signals` | 从可观测平台拉取指标/日志/链路信号 |
| `repository.read_file` | 读取代码仓库文件内容 |
| `repository.create_workspace` | 创建隔离工作区 |
| `repository.write_file` | 写入代码文件到工作区 |
| `ci.run_tests` | 运行测试门禁 |
| `knowledge.search_cases` | 搜索历史经验知识库 |
| `knowledge.write_case` | 写入知识卡到经验库 |
| `release.canary` | 灰度发布门禁检查 |

## 协议

- MCP Streamable HTTP，双版本：`2025-06-18` / `2025-11-25`
- 会话绑定 caseId/traceId（initialize 时锁定）
- 审计：每条 tools/call 记录 caller/tool/status/caseId/traceId
- 身份代理：Worker Bearer → x-devorbit-agent/x-case-id/x-trace-id 注入

## 真实执行证据

R4 自主探针 passed：7/7 Worker 各自调用自有 MCP，69 条同 Case/Trace 审计。

## 文件

- `schemas/` — MCP 工具 OpenAPI 定义
- `mcp/` — HTTP transport + tool server 实现
- `tool-policy.json` — Agent×Tool allowlist + case scope

## 许可证

Apache-2.0
