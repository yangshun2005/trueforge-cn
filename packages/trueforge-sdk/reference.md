# 参考文档
## Agents
<details><summary><code>client.agents.<a href="/src/api/resources/agents/client/Client.ts">list</a>() -> TrueForge.ListAgentsResponse</code></summary>
<dl>
<dd>

#### 📝 描述

<dl>
<dd>

<dl>
<dd>

该租户所有已配置的 Agent。
</dd>
</dl>
</dd>
</dl>

#### 🔌 用法

<dl>
<dd>

<dl>
<dd>

```typescript
await client.agents.list();

```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ 参数

<dl>
<dd>

<dl>
<dd>

**requestOptions:** `AgentsClient.RequestOptions` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.agents.<a href="/src/api/resources/agents/client/Client.ts">create</a>({ ...params }) -> TrueForge.GetAgentResponse</code></summary>
<dl>
<dd>

#### 📝 描述

<dl>
<dd>

<dl>
<dd>

创建一个 Agent 并分配一个不可变的 ID。如果 `name` 已被占用则失败。`name` 之后无法更改。
</dd>
</dl>
</dd>
</dl>

#### 🔌 用法

<dl>
<dd>

<dl>
<dd>

```typescript
await client.agents.create({
    manifest: {
        model: {
            name: "name"
        }
    },
    name: "name"
});

```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ 参数

<dl>
<dd>

<dl>
<dd>

**request:** `TrueForge.CreateAgentRequest` 
    
</dd>
</dl>

<dl>
<dd>

**requestOptions:** `AgentsClient.RequestOptions` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.agents.<a href="/src/api/resources/agents/client/Client.ts">get</a>(agent_id) -> TrueForge.GetAgentResponse</code></summary>
<dl>
<dd>

#### 📝 描述

<dl>
<dd>

<dl>
<dd>

通过不可变 ID 获取已配置的 Agent。
</dd>
</dl>
</dd>
</dl>

#### 🔌 用法

<dl>
<dd>

<dl>
<dd>

```typescript
await client.agents.get("agent_id");

```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ 参数

<dl>
<dd>

<dl>
<dd>

**agent_id:** `string` — 不可变的 Agent 标识符。
    
</dd>
</dl>

<dl>
<dd>

**requestOptions:** `AgentsClient.RequestOptions` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.agents.<a href="/src/api/resources/agents/client/Client.ts">update</a>(agent_id, { ...params }) -> TrueForge.GetAgentResponse</code></summary>
<dl>
<dd>

#### 📝 描述

<dl>
<dd>

<dl>
<dd>

替换以不可变 `agent_id` 为键的现有 Agent 的 manifest。
</dd>
</dl>
</dd>
</dl>

#### 🔌 用法

<dl>
<dd>

<dl>
<dd>

```typescript
await client.agents.update("agent_id", {
    manifest: {
        model: {
            name: "name"
        }
    }
});

```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ 参数

<dl>
<dd>

<dl>
<dd>

**agent_id:** `string` — 不可变的 Agent 标识符。
    
</dd>
</dl>

<dl>
<dd>

**request:** `TrueForge.UpdateAgentRequest` 
    
</dd>
</dl>

<dl>
<dd>

**requestOptions:** `AgentsClient.RequestOptions` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.agents.<a href="/src/api/resources/agents/client/Client.ts">delete</a>(agent_id) -> TrueForge.DeleteAgentResponse</code></summary>
<dl>
<dd>

#### 📝 描述

<dl>
<dd>

<dl>
<dd>

通过不可变 ID 删除已配置的 Agent。如果已删除则幂等。
</dd>
</dl>
</dd>
</dl>

#### 🔌 用法

<dl>
<dd>

<dl>
<dd>

```typescript
await client.agents.delete("agent_id");

```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ 参数

<dl>
<dd>

<dl>
<dd>

**agent_id:** `string` — 不可变的 Agent 标识符。
    
</dd>
</dl>

<dl>
<dd>

**requestOptions:** `AgentsClient.RequestOptions` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

## Auth
<details><summary><code>client.auth.<a href="/src/api/resources/auth/client/Client.ts">me</a>() -> TrueForge.GetMeResponse</code></summary>
<dl>
<dd>

#### 📝 描述

<dl>
<dd>

<dl>
<dd>

返回已认证的调用者身份（`type`、`tenant_id`、`subject`、`roles`），包装为 `{ data }`。当启用浏览器 OIDC 时，`type` 为 `oidc-connected`，否则为 `default`。启用认证时，此操作需要有效的 `id_token` cookie 或 `Authorization: Bearer` 令牌（否则返回 401）。禁用认证时，返回独立的默认身份。
</dd>
</dl>
</dd>
</dl>

#### 🔌 用法

<dl>
<dd>

<dl>
<dd>

```typescript
await client.auth.me();

```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ 参数

<dl>
<dd>

<dl>
<dd>

**requestOptions:** `AuthClient.RequestOptions` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

## Server
<details><summary><code>client.server.<a href="/src/api/resources/server/client/Client.ts">getCapabilities</a>() -> TrueForge.GetCapabilitiesResponse</code></summary>
<dl>
<dd>

#### 📝 描述

<dl>
<dd>

<dl>
<dd>

报告该租户可用的可选运行时能力。
</dd>
</dl>
</dd>
</dl>

#### 🔌 用法

<dl>
<dd>

<dl>
<dd>

```typescript
await client.server.getCapabilities();

```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ 参数

<dl>
<dd>

<dl>
<dd>

**requestOptions:** `ServerClient.RequestOptions` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

## MCP 服务器
<details><summary><code>client.mcpServers.<a href="/src/api/resources/mcpServers/client/Client.ts">list</a>() -> TrueForge.ListAvailableMcpServersResponse</code></summary>
<dl>
<dd>

#### 📝 描述

<dl>
<dd>

<dl>
<dd>

以精简的名称/URL 列表形式返回 MCP 服务器，供 composer 使用。不包含 auth 或 auth_status。
</dd>
</dl>
</dd>
</dl>

#### 🔌 用法

<dl>
<dd>

<dl>
<dd>

```typescript
await client.mcpServers.list();

```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ 参数

<dl>
<dd>

<dl>
<dd>

**requestOptions:** `McpServersClient.RequestOptions` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.mcpServers.<a href="/src/api/resources/mcpServers/client/Client.ts">authorize</a>(name, { ...params }) -> TrueForge.McpAuthStatus</code></summary>
<dl>
<dd>

#### 📝 描述

<dl>
<dd>

<dl>
<dd>

返回 MCP 服务器当前的认证状态。对于 OAuth（`auth.type` 为 dcr），当存在可用令牌时返回 authenticated；否则返回 auth_required 并附带授权 URL。可选的 return_to 参数指定 OAuth 回调将浏览器重定向到的位置；未提供时，回调返回 JSON。
</dd>
</dl>
</dd>
</dl>

#### 🔌 用法

<dl>
<dd>

<dl>
<dd>

```typescript
await client.mcpServers.authorize("name");

```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ 参数

<dl>
<dd>

<dl>
<dd>

**name:** `string` — MCP 服务器名称。
    
</dd>
</dl>

<dl>
<dd>

**request:** `TrueForge.AuthorizeMcpServersRequest` 
    
</dd>
</dl>

<dl>
<dd>

**requestOptions:** `McpServersClient.RequestOptions` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.mcpServers.<a href="/src/api/resources/mcpServers/client/Client.ts">deleteAuthorization</a>(name) -> TrueForge.GetMcpServerResponse</code></summary>
<dl>
<dd>

#### 📝 描述

<dl>
<dd>

<dl>
<dd>

在适用时断开 MCP 服务器的 OAuth 连接，并返回更新后的服务器及其 auth_status。当服务器不使用存储的 OAuth 令牌时，此操作不执行任何操作。
</dd>
</dl>
</dd>
</dl>

#### 🔌 用法

<dl>
<dd>

<dl>
<dd>

```typescript
await client.mcpServers.deleteAuthorization("name");

```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ 参数

<dl>
<dd>

<dl>
<dd>

**name:** `string` — MCP 服务器名称。
    
</dd>
</dl>

<dl>
<dd>

**requestOptions:** `McpServersClient.RequestOptions` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.mcpServers.<a href="/src/api/resources/mcpServers/client/Client.ts">listTools</a>(name) -> TrueForge.ListMcpServerToolsResponse</code></summary>
<dl>
<dd>

#### 📝 描述

<dl>
<dd>

<dl>
<dd>

给定 MCP 服务器暴露的所有工具（不分页），由 MCP `tools/list` 调用返回。
</dd>
</dl>
</dd>
</dl>

#### 🔌 用法

<dl>
<dd>

<dl>
<dd>

```typescript
await client.mcpServers.listTools("name");

```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ 参数

<dl>
<dd>

<dl>
<dd>

**name:** `string` — MCP 服务器名称。
    
</dd>
</dl>

<dl>
<dd>

**requestOptions:** `McpServersClient.RequestOptions` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

## 模型
<details><summary><code>client.models.<a href="/src/api/resources/models/client/Client.ts">list</a>() -> TrueForge.ListAvailableModelsResponse</code></summary>
<dl>
<dd>

#### 📝 描述

<dl>
<dd>

<dl>
<dd>

以精简的 FQN 列表形式返回已配置的模型，供 composer 使用。
</dd>
</dl>
</dd>
</dl>

#### 🔌 用法

<dl>
<dd>

<dl>
<dd>

```typescript
await client.models.list();

```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ 参数

<dl>
<dd>

<dl>
<dd>

**requestOptions:** `ModelsClient.RequestOptions` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

## 调度
<details><summary><code>client.schedules.<a href="/src/api/resources/schedules/client/Client.ts">list</a>({ ...params }) -> core.Page&lt;TrueForge.Schedule, TrueForge.ListSchedulesResponse&gt;</code></summary>
<dl>
<dd>

#### 📝 描述

<dl>
<dd>

<dl>
<dd>

列出该租户的调度，最新的在前。可选地按 `agent_names` 过滤。
</dd>
</dl>
</dd>
</dl>

#### 🔌 用法

<dl>
<dd>

<dl>
<dd>

```typescript
await client.schedules.list();

```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ 参数

<dl>
<dd>

<dl>
<dd>

**request:** `TrueForge.ListSchedulesRequest` 
    
</dd>
</dl>

<dl>
<dd>

**requestOptions:** `SchedulesClient.RequestOptions` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.schedules.<a href="/src/api/resources/schedules/client/Client.ts">create</a>({ ...params }) -> TrueForge.GetScheduleResponse</code></summary>
<dl>
<dd>

#### 📝 描述

<dl>
<dd>

<dl>
<dd>

为现有 Agent（按名称）创建调度，并在激活时添加其第一个待处理运行。
</dd>
</dl>
</dd>
</dl>

#### 🔌 用法

<dl>
<dd>

<dl>
<dd>

```typescript
await client.schedules.create({
    agentName: "agent_name",
    manifest: {
        cron: "cron",
        task: "task"
    },
    name: "name"
});

```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ 参数

<dl>
<dd>

<dl>
<dd>

**request:** `TrueForge.CreateScheduleRequest` 
    
</dd>
</dl>

<dl>
<dd>

**requestOptions:** `SchedulesClient.RequestOptions` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.schedules.<a href="/src/api/resources/schedules/client/Client.ts">createRun</a>({ ...params }) -> TrueForge.CreateScheduleRunResponse</code></summary>
<dl>
<dd>

#### 📝 描述

<dl>
<dd>

<dl>
<dd>

使用调度任务立即开始一次调度运行。不会替换或推进 cron 待处理运行。
</dd>
</dl>
</dd>
</dl>

#### 🔌 用法

<dl>
<dd>

<dl>
<dd>

```typescript
await client.schedules.createRun({
    scheduleId: "schedule_id"
});

```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ 参数

<dl>
<dd>

<dl>
<dd>

**request:** `TrueForge.CreateScheduleRunRequest` 
    
</dd>
</dl>

<dl>
<dd>

**requestOptions:** `SchedulesClient.RequestOptions` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.schedules.<a href="/src/api/resources/schedules/client/Client.ts">get</a>(schedule_id) -> TrueForge.GetScheduleResponse</code></summary>
<dl>
<dd>

#### 📝 描述

<dl>
<dd>

<dl>
<dd>

按 ID 获取调度。
</dd>
</dl>
</dd>
</dl>

#### 🔌 用法

<dl>
<dd>

<dl>
<dd>

```typescript
await client.schedules.get("schedule_id");

```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ 参数

<dl>
<dd>

<dl>
<dd>

**schedule_id:** `string` — 不可变的调度标识符。
    
</dd>
</dl>

<dl>
<dd>

**requestOptions:** `SchedulesClient.RequestOptions` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.schedules.<a href="/src/api/resources/schedules/client/Client.ts">update</a>(schedule_id, { ...params }) -> TrueForge.GetScheduleResponse</code></summary>
<dl>
<dd>

#### 📝 描述

<dl>
<dd>

<dl>
<dd>

替换名称和 manifest；当状态/cron/时区发生变化时，替换或删除待处理运行。
</dd>
</dl>
</dd>
</dl>

#### 🔌 用法

<dl>
<dd>

<dl>
<dd>

```typescript
await client.schedules.update("schedule_id", {
    manifest: {
        cron: "cron",
        task: "task"
    },
    name: "name"
});

```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ 参数

<dl>
<dd>

<dl>
<dd>

**schedule_id:** `string` — 不可变的调度标识符。
    
</dd>
</dl>

<dl>
<dd>

**request:** `TrueForge.UpdateScheduleRequest` 
    
</dd>
</dl>

<dl>
<dd>

**requestOptions:** `SchedulesClient.RequestOptions` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.schedules.<a href="/src/api/resources/schedules/client/Client.ts">delete</a>(schedule_id) -> TrueForge.DeleteScheduleResponse</code></summary>
<dl>
<dd>

#### 📝