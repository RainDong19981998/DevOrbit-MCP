# DevOrbit MCP Server

> GOAI 2026 Agent Infra 决赛项目

## 工具清单（14 个）

| 分组 | 工具 | 调用者 |
|---|---|---|
| Issue | issue.fetch_signals | intake-worker |
| Observability | observability.fetch_signals | intake/rca-worker |
| Repository | repository.read_file | impact/rca/patch/verify/release-worker |
| Repository | repository.create_workspace | patch-worker |
| Repository | repository.write_file | patch-worker |
| CI | ci.run_tests | patch/verify-worker |
| Knowledge | knowledge.search_cases | rca-worker |
| Knowledge | knowledge.write_case | learning-worker |
| Release | release.canary | release-worker |

## 协议

- MCP Streamable HTTP，双版本：`2025-06-18` / `2025-11-25`
- 会话绑定 caseId/traceId（initialize 时锁定）
- 审计：每条 tools/call 记录 caller/tool/status/caseId/traceId
- 身份代理：Worker Bearer → x-devorbit-agent/x-case-id/x-trace-id 注入

## 真实执行证据

R4 自主探针 passed：7/7 Worker 各自调用自有 MCP，69 条同 Case/Trace 审计。

## 文件

- `schemas/` — MCP 工具 OpenAPI 定义
- `src/mcp/` — HTTP transport + tool server 实现
- `config/tool-policy.json` — Agent×Tool allowlist + case scope
