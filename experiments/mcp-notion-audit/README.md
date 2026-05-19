# MCP Audit：Notion 官方 MCP Server 的 `query-data-source` 工具

> 实验目的：用 Handbook MCP 章节给的"四维度"（**name / description / inputSchema / 副作用**）拆解一个真实 MCP tool 的 schema，理解 OpenAPI → MCP 的映射方式与权限设计。
>
> 不写代码，只 audit。来源全部为公开 GitHub 文件。

## 标的物

- 官方仓库：[makenotion/notion-mcp-server](https://github.com/makenotion/notion-mcp-server)（local MCP server，22 tools，v2.0）
- 远程版（Notion 主推）：https://developers.notion.com/docs/mcp
- 拆解的 tool：`query-data-source`（v2.0 新增，取代 v1 的 `post-database-query`）

## 工程意外发现

**这个 server 不是手写每个 tool，而是 OpenAPI → MCP 自动转换。**

源码路径 `src/openapi-mcp-server/mcp/proxy.ts` 里有个 `MCPProxy` 类，构造函数接受一个 OpenAPI v3 spec（`scripts/notion-openapi.json`，66KB），然后调用 `OpenAPIToMCPConverter` 把每个 operation 转成 MCP `Tool` 对象。

启动时：

```
OpenAPI spec  →  ListToolsRequest 返回 22 个 Tool  →  Client 把 Tool 喂给模型
                  ↓
CallToolRequest("query-data-source", args) → MCPProxy 查 lookup 表 → HttpClient POST 到 Notion API
```

这意味着：**Notion 不需要为 MCP 写一份新代码，把现有 REST API 的 OpenAPI 自动派生即可。** 缺点是 schema 质量等于 OpenAPI 质量；优点是 22 个工具 + 未来扩展几乎零成本。

## 四维度拆解：`query-data-source`

### 1. Name

```
query-data-source
```

干净的 kebab-case，从 OpenAPI `operationId` 直接来。v2.0 改名提示给 LLM："你查的不是 database，是 data source（v2 引入的抽象）"。

### 2. Description

来自 OpenAPI summary + description：
- summary: `Query a data source`
- description: `Query a data source (database) using filters and sorts`

**Handbook 评分**：⚠️ 偏弱。
- ✅ 说了"用 filter 和 sort"
- ❌ 没说**什么时候用它**（vs `retrieve-a-data-source` 拿 schema）
- ❌ 没说**返回什么**（paged result? max page size?）
- ❌ 没说**有没有副作用**（这个是只读，但 description 没明示）

对照 Handbook："Tool schema 写得模糊，模型就会用错误参数填空。" —— 这里有真实风险：模型可能把 `query` 错用成 `retrieve` 拿 schema。

### 3. Input Schema

**Path 参数**（必填）：

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `data_source_id` | string | ✅ | data source 标识符（v2 替代 database_id） |

**Query 参数**：

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `filter_properties` | string[] | ❌ | 限制返回的 page property value IDs |
| `Notion-Version` | header（共享 ref） | — | API 版本 |

**Body 参数**：

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `filter` | object | ❌ | 过滤条件（递归 schema，支持 and/or） |
| `sorts` | array | ❌ | 排序规则 |
| `page_size` | number | ❌ | 单页大小（默认 100） |
| `start_cursor` | string | ❌ | 分页游标 |
| `archived` | boolean | ❌ | 是否包含已归档 |
| `in_trash` | boolean | ❌ | 是否包含回收站 |

**Handbook 评分**：✅ 大部分清晰，但有两个坑：

1. **`filter` 是递归 schema** —— Notion filter 支持 `{and: [filter, filter]}` 这种嵌套结构。OpenAPI 里用 `$ref` 自引用。LLM 经常**双重序列化**这种嵌套对象（看 proxy.ts 里有专门的 `deserializeParams` 函数，注释指向 [issue #176](https://github.com/makenotion/notion-mcp-server/issues/176)）—— 真实工程问题。
2. **没有 `required` 字段** —— body 整体没声明哪个必填，于是空 body 也算合法（"全量返回"）。这对模型来说是好事（minimum viable call），但也容易触发**意外的全量读取**。

### 4. 副作用 / 权限

OpenAPI 该 path 的 `security: []` —— **看起来无 auth**。但显然 Notion 是要 auth 的，怎么解释？

看 `src/openapi-mcp-server/auth/` 和 `client/http-client.ts`：**auth 在 HttpClient 层注入**，token 来自启动时配的 `OPENAPI_MCP_HEADERS` 环境变量（典型值 `{"Authorization": "Bearer ntn_xxx", "Notion-Version": "2022-06-28"}`）。

这是 Handbook **Permission** 章节的关键警示落地：

> "MCP 让工具连接更方便，但方便不等于安全。"

**实际权限边界**：

| 维度 | 现状 |
|---|---|
| 只读 vs 写入 | 本工具只读；但同 server 暴露的 `post-page` / `update-a-data-source` 等是写入 |
| 当前会话 vs 长期 | **长期** —— internal integration token 一旦给出，server 重启也持续生效 |
| 是否用户确认 | 协议层无确认；靠 Client（如 Claude Code）的 `--permission-mode` 兜底 |
| 敏感信息访问 | Notion integration 配置时可以限制 page 范围，但 MCP server 看不见这层 |
| 副作用 | 只读 |
| 撤销 | 在 https://www.notion.so/profile/integrations 撤 token |
| 审计 | server 端无日志（除非自加） |

**Handbook 的"权限至少要区分"清单 vs 这个 server 实际**：

| Handbook 要求 | Notion local MCP 实际 |
|---|---|
| 只读 vs 写入 | ⚠️ 同一个 server 混着 22 个，没有 read-only 模式开关 |
| 会话 vs 长期 | ❌ 都是长期 |
| 是否需要用户确认 | ❌ 协议层无；靠 client |
| 是否敏感访问 | ⚠️ Notion integration 层可限范围，但 MCP 层透明 |
| 是否副作用 | ❌ 工具元数据里看不出 |
| 是否可撤销 | ✅ Notion integration UI |
| 是否可审计 | ❌ 默认无 |

README 自己也承认（首页警告）：

> "We limit the scope of Notion API's exposed (for example, you will not be able to delete databases via MCP), there is a non-zero risk to workspace data by exposing it to LLMs."

## 把 audit 翻译成"我下次设计 MCP server 时的 checklist"

1. **operationId 不等于好的 tool name** —— description 要补"什么时候用 / 什么时候不用"
2. **副作用维度必须放进 description**，不能只放在 HTTP method 上（GET/POST/PATCH 模型不一定 follow）
3. **递归 schema 要写 deserializeParams** —— 否则 Claude Code / Cursor 会双重序列化
4. **权限不能假装由协议解决** —— MCP 只管"工具长什么样"，权限要在 server 启动配置 / Client UI / 上游 integration 三层叠加
5. **只读和写入应该分 server**（或加 capability flag） —— 否则 Permission 章节里"只读 vs 写入"维度失效
6. **每次调用写审计日志**（来源、参数、结果） —— Notion MCP 没做，自己做时必加

## 引用

- 仓库根：https://github.com/makenotion/notion-mcp-server
- OpenAPI spec（66KB）：https://github.com/makenotion/notion-mcp-server/blob/main/scripts/notion-openapi.json
- MCPProxy 实现：https://github.com/makenotion/notion-mcp-server/blob/main/src/openapi-mcp-server/mcp/proxy.ts
- 双重序列化 issue：https://github.com/makenotion/notion-mcp-server/issues/176
- Remote MCP 文档：https://developers.notion.com/docs/mcp
- Handbook MCP 章节：https://aiweb3.school/zh/handbook/ai/mcp/
