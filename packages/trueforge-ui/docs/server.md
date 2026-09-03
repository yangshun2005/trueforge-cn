# 计划：服务器抽象（仅 UI SDK）

面向宿主端的端口，使 TrueFoundry 网关**或**自带的 API 能够接入同一个契约：

- **聊天** — 会话、轮次、草稿 `AgentSpec` 同步
- **构建器** — 目录（模型 / 技能 / MCP，只读）+ `saveAgent`（将草稿提升为命名代理）

## 约束

所有内容都位于 **`@truefoundry/trueforge-ui`** 中。

**不修改** `@truefoundry/assistant-ui-runtime`。

运行时继续接受 `client: AgentSessionClient`（以及用于草稿的 `privateClient`）。本包拥有面向宿主的 `Server` 门面（façade）和适配器。

## 影响（明确说明）

| 关注点                                                        | 本计划中的实际情况                                                                                                                                                |
| ------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 自带构建器（`getModels` / `getSkills` / `getMcp` / `saveAgent`） | 真实 — 在本 SDK 中实现并使用                                                                                                                                      |
| 自带聊天流式传输                                              | 受限 — 宿主仍必须向运行时提供兼容网关的客户端，**或**将其 API 包装在运行时已期望的对象后面                                                                        |
| `AgentChatServer`                                             | 本 SDK 中稳定的**宿主** API；TFY 路径提取/传递网关客户端到未更改的运行时中                                                                                         |

## 已确定的决策

| 决策                    | 选择                                                                                                            |
| ----------------------- | --------------------------------------------------------------------------------------------------------------- |
| 形态                    | TypeScript `interface`（组合），而非继承                                                                        |
| 拆分                    | `AgentChatServer` + `AgentBuilderServer`                                                                        |
| 包                      | 仅此 SDK                                                                                                        |
| 模式（v1）              | 命名 + 草稿（单代理）                                                                                           |
| 聊天请求 / 响应类型     | 网关 SDK 类型（`AgentSession`、`AgentSpec`、`Turn` 等）                                                         |
| 目录类型                | SDK 最小化的 `ModelEntry` / `AgentSkill` / `ConnectorState` / `AgentLibraryEntry`；宿主通过泛型扩展              |
| `saveAgent`             | 将草稿 `AgentSpec` 提升为命名代理；返回值目前仍由宿主定义                                                       |
| 目录列表                | v1 中只读                                                                                                       |
| 运行时                  | 未更改；仍为 `client` / `privateClient` / `agent`                                                               |
| 聊天泛型                | v1 中**无** — 会话/轮次类型是网关固定的再导出                                                                   |

## 接口（目标）

聊天类型从 `truefoundry-gateway-sdk`（代理 + API）导入。`AgentSpec` 是网关 SDK（`TruefoundryGatewayApi.AgentSpec`），由 `@truefoundry/assistant-ui-runtime` 再导出。构建器目录类型是 **SDK 最小化**的（本 UI 读取的字段）；宿主通过 `T extends Base` 保留额外类型。

```ts
import type { AgentSession, AgentDraftSession, Turn, TurnStreamData } from 'truefoundry-gateway-sdk/agents';
import type {
  AgentSpec,
  TurnInputItem,
  PreviousTurnIdInput,
  TurnState,
  SessionEventItem,
  ListSessionsResponse,
  ListDraftSessionsResponse,
  ListOwnedSessionsResponse,
  ListTurnsResponse,
  ListSessionEventsResponse,
  ListSessionsOrder,
} from 'truefoundry-gateway-sdk';
import type { Page, BinaryResponse } from 'truefoundry-gateway-sdk/core'; // 或等效的核心导出

type PageParams = {
  limit?: number;
  order?: ListSessionsOrder;
  pageToken?: string;
  startTimestamp?: string;
  endTimestamp?: string;
};

/** 聊天 / 会话端口 — 网关固定（非泛型）。 */
interface AgentChatServer {
  // 命名
  createSession(req: { agentName: string; tfyMetadata?: string }): Promise<AgentSession>;
  listSessions(req: { agentName: string } & PageParams): Promise<Page<AgentSession, ListSessionsResponse>>;
  getSession(req: { sessionId: string }): Promise<AgentSession>;

  // 草稿
  createDraftSession(req: {
    agentSpec: AgentSpec;
    agentName?: string;
    tfyMetadata?: string;
  }): Promise<AgentDraftSession>;
  getDraftSession(req: { draftSessionId: string }): Promise<AgentDraftSession>;
  listDraftSessions(
    req?: { agentName?: string } & PageParams,
  ): Promise<Page<AgentDraftSession, ListDraftSessionsResponse>>;
  listOwnedSessions(
    req?: { agentName?: string } & PageParams,
  ): Promise<Page<AgentSession | AgentDraftSession, ListOwnedSessionsResponse>>;
  /** 端口辅助方法：getDraftSession + update。运行时草稿同步仍可能通过 privateClient 进行。 */
  updateDraftSession(req: { draftSessionId: string; agentSpec?: AgentSpec }): Promise<AgentDraftSession>;

  // 轮次（重载与 PreparedTurn.execute 匹配）
  createTurn(req: {
    sessionId: string;
    input?: TurnInputItem[];
    previousTurnId?: PreviousTurnIdInput;
    stream: false;
    pollIntervalMs?: number;
  }): Promise<TurnState>;
  createTurn(req: {
    sessionId: string;
    input?: TurnInputItem[];
    previousTurnId?: PreviousTurnIdInput;
    stream?: true;
  }): AsyncIterable<TurnStreamData>;

  cancelSession(req: { sessionId: string }): Promise<void>;
  /** 宿主/BFF — 可选，直到删除 UI 发布。 */
  deleteSession?(req: { sessionId: string }): Promise<void>;
  listTurns(req: { sessionId: string; limit?: number; pageToken?: string }): Promise<Page<Turn, ListTurnsResponse>>;
  getTurn(req: { sessionId: string; turnId: string }): Promise<Turn>;
  listEvents(req: {
    sessionId: string;
    pageToken?: string;
    lastTurnId?: string;
    limit?: number;
  }): Promise<Page<SessionEventItem, ListSessionEventsResponse>>;
  subscribeToTurn(req: {
    sessionId: string;
    turnId: string;
    afterSequenceNumber?: number;
  }): AsyncIterable<TurnStreamData>;

  downloadSandboxFile?(sandboxId: string, req: { path: string }): Promise<BinaryResponse>;
}

/**
 * 构建器目录 + 持久化。
 * 不在 truefoundry-gateway-sdk 中 — 由宿主/BFF 提供。
 * 类似于 MUI Table：基础类型声明 SDK 所需内容；T extends Base 保留宿主的额外类型。
 */
interface AgentBuilderServer<
  TModel extends ModelEntry = ModelEntry,
  TSkill extends AgentSkill = AgentSkill,
  TMcp extends ConnectorState = ConnectorState,
  TAgent extends AgentLibraryEntry = AgentLibraryEntry,
  TSave = unknown,
> {
  getModels(): Promise<TModel[]>;
  getSkills(): Promise<TSkill[]>;
  getMcp(): Promise<TMcp[]>;
  /** 搜索 / 列出命名代理，用于代理库。 */
  searchAgents(req?: { query?: string; limit?: number; offset?: number }): Promise<TAgent[]>;
  /** 将草稿 AgentSpec 提升为命名代理 */
  saveAgent(req: { agentName: string; agentSpec: AgentSpec; draftSessionId?: string }): Promise<TSave>;
  /** 宿主/BFF — 可选，直到删除代理 UI 发布。 */
  deleteAgent?(req: { agentName: string }): Promise<void>;
}

type AgentUIServer<
  TModel extends ModelEntry = ModelEntry,
  TSkill extends AgentSkill = AgentSkill,
  TMcp extends ConnectorState = ConnectorState,
  TAgent extends AgentLibraryEntry = AgentLibraryEntry,
  TSave = unknown,
> = AgentChatServer & AgentBuilderServer<TModel, TSkill, TMcp, TAgent, TSave>;
```

默认值使裸 `AgentUIServer` 等同于 SDK 最小化的目录形态。
**注意：** 网关会话辅助类是类实例（带方法的 `AgentSession` / `AgentDraftSession`）。TFY 适配器直接返回这些实例。自带实现必须返回满足相同类型的值（或进行包装，直到端口仅暴露纯 DTO — v1 不要求）。

纯 DTO 也作为 `Session` / `DraftSession`（`TruefoundryGatewayApi`）再导出。

### 扩展目录类型

```ts
type MyModel = ModelEntry & { latencyMs: number };

const server: AgentUIServer<MyModel> = createTrueFoundryServer({
  // …
  getModels: async () => [
    {
      id: '…',
      name: '…',
      provider: { name: '…' },
      properties: {},
      apiModel: '…',
      modelId: '…',
      latencyMs: 12,
    },
  ],
});
// server.getModels() → Promise<MyModel[]>
```

## 目录 API 与 AgentSpec 类型

目录基础类型是 **SDK 最小化**的（本 UI 读取的字段）。平台/cpApi 载荷可能更丰富 — 将其映射到这些形态中，或通过泛型保留额外字段。

### 目录方法

```ts
getModels(): Promise<ModelEntry[]>;
getSkills(): Promise<AgentSkill[]>;
getMcp(): Promise<ConnectorState[]>;
searchAgents(req?: { query?: string; limit?: number; offset?: number }): Promise<AgentLibraryEntry[]>;
```

在平台应用中，RTK Query 端点命名为 `getEnabledModels`、`getAgentSkills`、`getMcpServers` 和 `getAgents`（`GET /api/svc/v1/agents?namePrefix=…`）。

#### `AgentLibraryEntry`

```ts
interface AgentLibraryEntry {
  name: string;
  description?: string;
  model?: string;
  skillsCount?: number;
  mcpCount?: number;
  author?: string;
  updatedAt?: string;
}
```

#### `ModelEntry`

通过 `apiModel` 写入 `AgentSpec.model.name`（回退：`name`）。

```ts
interface ModelEntry {
  name: string;
  provider: string;
  apiModel: string;
  modelId: string;
}
```

#### `AgentSkill`

通过 `fqn` 挂载到 `AgentSpec.skills` 上（回退：`name`）。

```ts
interface AgentSkill {
  id: string;
  name: string;
  fqn?: string;
  description?: string;
}
```

#### `ConnectorState`

写入 `AgentSpec.mcpServers[].name`。

```ts
interface ConnectorState {
  id: string;
  name: string;
  description?: string;
}
```

#### 会话 / 草稿会话（网关 DTO）

```ts
interface Session {
  type: 'session';
  id: string;
  agentName: string;
  title?: string;
  createdBySubject: Subject;
  createdAt: string;
  updatedAt: string;
}

interface DraftSession {
  type: 'session/draft';
  id: string;
  agentSpec: AgentSpec;
  agentName?: string;
  title?: string;
  createdBySubject: Subject;
  createdAt: string;
  updatedAt: string;
}
```

线程列表使用 `id`、`title?`、`updatedAt`。草稿标题回退到 `agentSpec.model.name`。`AgentSession` / `AgentDraftSession` 在此基础上添加方法。

### `AgentSpec`（完整）

事实来源：`truefoundry-gateway-sdk` → `TruefoundryGatewayApi.AgentSpec`  
由 `@truefoundry/assistant-ui-runtime` 再导出

```ts
interface AgentSpec {
  model: Model;
  instructions?: string;
  messages?: AgentSpecUserMessage[];
  mcpServers?: McpServer[];
  responseFormat?: ResponseFormat;
  skills?: SkillMount[];
  config?: RuntimeConfig;
}

interface CompactionConfig {
  enabled?: boolean;
  trigger?: {
    type: 'input_tokens';
    value: number;
  };
}

interface Model {
  name: string;
  params?: ModelParams;
}

interface ModelParams {
  maxTokens?: number;
  temperature?: number;
  topP?: number;
  topK?: number;
  parallelToolCalls?: boolean;
  reasoningEffort?: string;
  cacheControl?: { type: string };
}

interface SkillMount {
  /** 代理技能版本的完全限定名称。 */
  fqn: string;
  /** 如果为 true，则将 SKILL.md 注入代理上下文中。 */
  preload?: boolean;
}

type ToolsSelectorTag = '@all' | '@read-only';
type ToolsSelectorItem = ToolsSelectorTag | string;

type RequireApprovalToolsSelectorTag = '@all' | '@write' | '@destructive';
type RequireApprovalToolSelectorItem = RequireApprovalToolsSelectorTag | string;

interface BaseMcpServer {
  name: string;
  enableTools?: ToolsSelectorItem[];
  disableTools?: ToolsSelectorItem[];
  preloadTools?: ToolsSelectorItem[];
  requireApprovalForTools?: RequireApprovalToolSelectorItem[];
  preload?: boolean;
}

interface RegisteredMcpServer extends BaseMcpServer {
  type: 'truefoundry-mcp-registry';
}

interface InlineMcpServer extends BaseMcpServer {
  type: 'inline';
  url: string;
}

type McpServer = RegisteredMcpServer | InlineMcpServer;

interface AgentSpecUserMessage {
  type: 'user.message';
  content: string;
}

type ResponseFormat =
  | { type: 'text' }
  | { type: 'json_object' }
  | {
      type: 'json_schema';
      jsonSchema: {
        name: string;
        description?: string;
        schema?: Record<string, unknown>;
        strict?: boolean;
      };
    };

interface RuntimeConfig {
  iterationLimit?: number;
  sandbox?: {
    enabled: boolean;
    fileDownloads?: boolean;
  };
  dynamicSubAgents?: { enabled?: boolean };
  contextManagement?: {
    compaction?: CompactionConfig;
    largeToolResponse?: { enabled?: boolean };
  };
  generativeUi?: { enabled?: boolean };
  askUserQuestions?: { enabled?: boolean };
}
```

### `AgentSpecUpdate`（部分补丁）

```ts
type AgentSpecUpdate = {
  instructions?: string;
  model?: Partial<Model> & {
    params?: Partial<ModelParams>;
  };
  mcpServers?: McpServer[];
  skills?: SkillMount[];
  messages?: AgentSpecUserMessage[];
  responseFormat?: ResponseFormat;
  config?: RuntimeConfig;
};
```

### 本应用写入的草稿子集

典型的构建器 → 草稿补丁形态（非完整 `AgentSpec`）：

```ts
{
  model: {
    name: string;
    params?: { maxTokens?: number; reasoningEffort?: string };
  };
  skills: { fqn: string; preload: false }[];
  mcpServers: {
    type: "truefoundry-mcp-registry";
    name: string;
    enableTools: ["@all"];
  }[];
}
```

## 网关映射（`AgentChatServer`）

| 端口方法                              | 网关来源                                        |
| ------------------------------------- | ----------------------------------------------- |
| `createSession`                       | `AgentSessionClient.createSession`              |
| `listSessions`                        | `AgentSessionClient.listSessions`               |
| `getSession`                          | `AgentSessionClient.getSession`                 |
| `createDraftSession`                  | `PrivateAgentSessionClient.createDraftSession`  |
| `getDraftSession`                     | `PrivateAgentSessionClient.getDraftSession`     |
| `listDraftSessions`                   | `PrivateAgentSessionClient.listDraftSessions`   |
| `listOwnedSessions`                   | `PrivateAgentSessionClient.listOwnedSessions`   |
| `updateDraftSession`                  | `getDraftSession` + `AgentDraftSession.update`  |
| `createTurn`                          | 会话 `prepareTurn` → `execute` / stream         |
| `cancelSession`                       | 会话 `cancel`                                   |
| `listTurns` / `getTurn` / `listEvents`| 会话轮次辅助方法                                |
| `subscribeToTurn`                     | 轮次 `stream` / subscribe                       |
| `downloadSandboxFile`                 | `PrivateAgentSessionClient.downloadSandboxFile` |

**不在网关中：** `getModels`、`getSkills`、`getMcp`、`searchAgents`、`saveAgent`、`deleteAgent?`、`deleteSession?`。

## 调用流程（最终状态）

```
宿主
  └─ createTrueFoundryServer / 自定义 AgentUIServer
TrueForgeUI({ server, agentConfig, … })
  ├─ 构建器 UI ──► server.getModels / getSkills / getMcp / saveAgent
  └─ TrueFoundryChatProvider
        └─ useTrueFoundryAgentRuntime({
             client,          // 来自 TFY 适配器内部
             privateClient?,  // 草稿
             agent: named | draft
           })   // 运行时包未修改
```

## 阶段

### 阶段 0 — 规范冻结

1. 将本文档作为接口和映射的事实来源。
2. 确认组合类型：`AgentUIServer = AgentChatServer & AgentBuilderServer`。

**完成标准：** 接口已达成一致；无需代码。

### 阶段 1 — 类型 + TFY 组合辅助函数

1. 在 `src/server/` 下添加 `AgentChatServer`、`AgentBuilderServer`、`AgentUIServer`。
2. 添加 `createTrueFoundryServer({ apiKey, baseUrl, saveAgent, getModels?, getSkills?, getMcp? })`：
   - 构造 `AgentSessionClient` + `PrivateAgentSessionClient`
   - 将聊天方法实现为薄包装
   - 通过宿主回调（`ModelEntry[]` / `AgentSkill[]` / `ConnectorState[]`）实现构建器方法
   - 为运行时暴露 SDK 内部句柄，例如
     `getGatewayClients(): { client, privateClient }`