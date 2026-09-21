# DevOrbit MCP Server

> V1.1.0 · Apache-2.0 · 14 tools, dual protocol, case-scoped audit, identity proxy

## 简介

DevOrbit MCP Server 是 DevOrbit 多 Agent 研发闭环平台的核心工具层。七个职能 Worker（intake/impact/rca/patch/verify/release/learning）各自通过独有 Bearer 令牌连接 MCP Server，调用授权范围内的工具完成信号采集、代码读写、测试门禁、知识检索和灰度发布等操作。

每条 `tools/call` 请求经身份代理注入 `x-devorbit-agent`（Worker 身份）、`x-case-id`（案例标识）、`x-trace-id`（追踪标识），实现同 Case/Trace 的审计归属。会话在 `initialize` 时锁定 caseId/traceId，后续调用保持一致性。

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

> 魔搭托管地址需 ModelScope Token 认证。

## 工具列表（14 个）

| 工具 | 调用者 | 说明 |
|------|--------|------|
| `issue.fetch_signals` | intake-worker | 从 Issue 系统拉取故障信号（用户反馈、工单、变更记录） |
| `observability.fetch_signals` | intake/rca-worker | 从可观测平台拉取指标、日志、链路追踪信号（surface + deep 两层） |
| `repository.read_file` | impact/rca/patch/verify/release-worker | 读取代码仓库文件内容，支持路径和版本指定 |
| `repository.create_workspace` | patch-worker | 创建隔离工作区，确保补丁在独立分支生成 |
| `repository.write_file` | patch-worker | 写入代码文件到隔离工作区 |
| `ci.run_tests` | patch/verify-worker | 运行测试门禁，返回 Red/Green 状态和失败详情 |
| `knowledge.search_cases` | rca-worker | 搜索历史经验知识库，按服务/环境/版本过滤 |
| `knowledge.write_case` | learning-worker | 写入知识卡到经验库（含根因、补丁、验证结果、失败原因） |
| `release.canary` | release-worker | 灰度发布门禁检查（SLO、错误率、p95 延迟） |

## 协议

- **MCP Streamable HTTP**，双版本：`2025-06-18` / `2025-11-25`
- **会话绑定**：caseId/traceId 在 `initialize` 时锁定，后续调用保持一致
- **审计**：每条 `tools/call` 记录 caller/tool/status/caseId/traceId，支持按 Worker/工具/时间过滤
- **身份代理**：Worker Bearer → `x-devorbit-agent`/`x-case-id`/`x-trace-id` 头注入
- **会话 TTL**：12 小时（消除短 TTL 导致的 Worker 生命周期死亡）

## 真实执行证据

官方 AgentTeams 自主探针 `CASE-AUTO-INVENTORY-20260918-R4` 已通过（status=passed）：
- 7/7 Worker 各自调用自有 MCP 工具
- 69 条同 Case/Trace 的 MCP 审计条目
- 422 条 Worker 消息 + 169 条 Leader 消息
- Leader 完成分诊→DAG 派发→逐任务验收→交付终态

## 文件结构

| 路径 | 说明 |
|------|------|
| `mcp/http-transport.js` | MCP Streamable HTTP 传输层（会话管理、协议协商） |
| `mcp/protocol.js` | MCP 协议实现（initialize/tools/list/tools/call） |
| `mcp/tool-server.js` | 工具服务器（工具注册与分发） |
| `mcp/tools.js` | 14 个工具定义（inputSchema、handler、policy） |
| `mcp/client.js` | MCP 客户端（Worker 侧连接管理） |
| `schemas/http-adapter.openapi.json` | MCP 工具 OpenAPI 定义 |
| `schemas/tool-contract.schema.json` | 工具契约 Schema |
| `tool-policy.json` | Agent×Tool allowlist + case scope（哪个 Worker 能调哪个工具） |

## 验证

- `npm test`：118/118 PASS（含身份代理、工具策略、审计归属测试）
- `npm run release-audit`：1607/1607 PASS
- AgentTeams 自主探针：status=passed（7/7 Worker、69 条审计）

## 许可证

Apache-2.0

## 相关链接

- [DevOrbit 主仓库](https://github.com/RainDong19981998/DevOrbit)
- [DevOrbit 创空间](https://modelscope.cn/studios/rain777/DevOrbit)
- [DevOrbit Skills](https://modelscope.cn/skills/rain777/devorbit-skills)
