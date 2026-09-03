# TEMP：自包含服务端类型（复制版，无导入）

> **仅草案。** 未接入包中。
> 以下类型是从 `truefoundry-gateway-sdk` / 运行时结构中**粘贴**而来，便于你在无需导入的情况下阅读契约。
> 实际代码仍从网关重新导出（`src/server/types.ts`），除非你决定升级。

源固定版本（约）：`truefoundry-gateway-sdk@0.4.0-rc.1`

---

## 目录（SDK 所有——已本地化）

```ts
export interface ModelEntry {
  name: string;
  provider: string;
  apiModel: string;
  modelId: string;
}

export interface AgentSkill {
  id: string;
  name: string;
  fqn?: string;
  description?: string;
}

export interface ConnectorState {
  id: string;
  name: string;
  description?: string;
}

export interface AgentLibraryEntry {
  name: string;
  description?: string;
  model?: string;
  skillsCount?: number;
  mcpCount?: number;
  author?: string;
  updatedAt?: string;
}

export type SearchAgentsParams = {
  query?: string;
  limit?: number;
  offset?: number;
};

export type DraftAgentSpecPatch = {
  model: {
    name: string;
    params?: { maxTokens?: number; reasoningEffort?: string };
  };
  skills: { fqn: string; preload: false }[];
  mcpServers: {
    type: 'truefoundry-mcp-registry';
    name: string;
    enableTools: ['@all'];
  }[];
};
```

---

## 网关 / 运行时结构（复制版）

### Subject

```ts
export interface Subject {
  subjectId: string;
  subjectType: string;
  subjectSlug: string;
}
```

### 分页

```ts
export interface TokenPagination {
  nextPageToken?: string;
  previousPageToken?: string;
  limit: number;
}

/** 宿主必须满足的结构化页面形状（网关 `Page` 是一个类）。 */
export interface PageResult<T, R = unknown> {
  data: T[];
  response: R;
  hasNextPage(): boolean;
  getNextPage(): Promise<PageResult<T, R>>;
}

export type ListSessionsOrder = 'asc' | 'desc';

export type PageParams = {
  limit?: number;
  order?: ListSessionsOrder;
  pageToken?: string;
  startTimestamp?: string;
  endTimestamp?: string;
};

export type PreviousTurnIdInput = 'auto' | 'none' | string;
```

### AgentSpec（含嵌套）

```ts
export interface ModelParamsCacheControl {
  type: string;
}

export interface ModelParams {
  maxTokens?: number;
  temperature?: number;
  topP?: number;
  topK?: number;
  parallelToolCalls?: boolean;
  reasoningEffort?: string;
  cacheControl?: ModelParamsCacheControl;
}

export interface Model {
  name: string;
  params?: ModelParams;
}

export interface SkillMount {
  /** 智能体技能版本的完全限定名称。 */
  fqn: string;
  /** 若为 true，SKILL.md 将被注入智能体上下文。 */
  preload?: boolean;
}

export type ToolsSelectorTag = '@all' | '@read-only';
export type ToolsSelectorItem = ToolsSelectorTag | string;

export type RequireApprovalToolsSelectorTag = '@all' | '@write' | '@destructive';
export type RequireApprovalToolSelectorItem = RequireApprovalToolsSelectorTag | string;

export interface BaseMcpServer {
  name: string;
  enableTools?: ToolsSelectorItem[];
  disableTools?: ToolsSelectorItem[];
  preloadTools?: ToolsSelectorItem[];
  requireApprovalForTools?: RequireApprovalToolSelectorItem[];
  preload?: boolean;
}

export interface RegisteredMcpServer extends BaseMcpServer {
  type: 'truefoundry-mcp-registry';
}

export interface InlineMcpServer extends BaseMcpServer {
  type: 'inline';
  url: string;
}

export type McpServer = RegisteredMcpServer | InlineMcpServer;

export interface AgentSpecUserMessage {
  type: 'user.message';
  content: string;
}

export type ResponseFormat =
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

export interface RuntimeConfig {
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

export interface CompactionConfig {
  enabled?: boolean;
  trigger?: {
    type: 'input_tokens';
    value: number;
  };
}

/** 智能体定义（网关 `TruefoundryGatewayApi.AgentSpec`） */
export interface AgentSpec {
  model: Model;
  instructions?: string;
  messages?: AgentSpecUserMessage[];
  mcpServers?: McpServer[];
  responseFormat?: ResponseFormat;
  skills?: SkillMount[];
  config?: RuntimeConfig;
}

/** 部分补丁（来自 `@truefoundry/assistant-ui-runtime`） */
export type AgentSpecUpdate = {
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

### Session / DraftSession

```ts
export interface Session {
  type: 'session';
  id: string;
  agentName: string;
  title?: string;
  createdBySubject: Subject;
  createdAt: string;
  updatedAt: string;
}

export interface DraftSession {
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

### Turn 输入 / 状态

```ts
export type UserMessageContent =
  | string
  | Array<
      | { type: 'text'; text: string } // TextContent（简化版）
      | { type: 'file'; /* FileContent — 参见网关 */ [key: string]: unknown }
    >;

export interface UserMessage {
  type: 'user.message';
  content: UserMessageContent;
}

export interface ApprovalAllow {
  status: 'allow';
}

export interface ApprovalDeny {
  status: 'deny';
  reason?: string;
}

export type ApprovalDecision = ApprovalAllow | ApprovalDeny;

export interface UserToolApprovalEvent {
  type: 'user.tool_approval';
  threadId: string;
  toolCallId: string;
  approval: ApprovalDecision;
}

export interface UserToolResponseEvent {
  type: 'user.tool_response';
  threadId: string;
  toolCallId: string;
  content: string;
}

export type TurnInputItem = UserMessage | UserToolApprovalEvent | UserToolResponseEvent;

export interface TurnStateRunning {
  status: 'running';
}

export interface TurnStateDone {
  status: 'done';
  output?: {
    type: 'model.message';
    id: string;
    threadId: string;
    createdAt: string;
    content?: unknown;
    name?: string;
    refusal?: string;
    reasoningContent?: string;
    toolCalls?: unknown[];
    finishReason?: string;
    usage?: unknown;
  };
  requiredActions: unknown[]; // 网关中为 ActionRequiredEvent[]
  completedAt: string;
}

export type TurnStateCancelledReason =
  'server-execution-timeout' | 'client-cancelled' | 'cancelled-for-next-turn' | 'abandoned';

export interface TurnStateCancelled {
  status: 'cancelled';
  reason: TurnStateCancelledReason;
  completedAt: string;
}

export interface TurnStateError {
  status: 'error';
  message: string;
  completedAt: string;
}

export type TurnState = TurnStateRunning | TurnStateDone | TurnStateCancelled | TurnStateError;

export interface Turn {
  id: string;
  sessionId: string;
  previousTurnId?: string;
  input?: TurnInputItem[];
  state: TurnState;
  createdBySubject: Subject;
  createdAt: string;
}
```

### 流 / 会话事件

```ts
/**
 * 网关 `TurnStreamingEvent` 联合类型（仅名称——完整事件载荷在网关中）：
 * ModelMessageEvent | ModelMessageDeltaEvent | ToolResponseEvent |
 * ThreadCreatedEvent | ThreadDoneEvent | McpAuthRequiredEvent | McpInitializeEvent |
 * SandboxCreatedEvent | ToolApprovalRequiredEvent | ToolResponseRequiredEvent |
 * TurnCreatedEvent | TurnDoneEvent
 */
export type TurnStreamingEvent = { type: string; [key: string]: unknown };

export interface TurnStreamData {
  /** 用于通过 subscribeToTurn 恢复的 SSE 事件 ID。 */
  sequenceNumber: number;
  event: TurnStreamingEvent;
}

/**
 * 网关 `SessionEvent` 联合类型（流式子集，不含增量）：
 * TurnCreatedEvent | TurnDoneEvent | ModelMessageEvent | ToolResponseEvent |
 * ThreadCreatedEvent | ThreadDoneEvent | McpAuthRequiredEvent | McpInitializeEvent |
 * SandboxCreatedEvent | ToolApprovalRequiredEvent | ToolResponseRequiredEvent
 */
export type SessionEvent = { type: string; [key: string]: unknown };

export interface SessionEventItem {
  turnId: string;
  event: SessionEvent;
}
```

### 列表响应

```ts
export interface ListSessionsResponse {
  data: Session[];
  pagination: TokenPagination;
}

export interface ListDraftSessionsResponse {
  data: DraftSession[];
  pagination: TokenPagination;
}

export interface ListOwnedSessionsResponse {
  data: Array<Session | DraftSession>;
  pagination: TokenPagination;
}

export interface ListTurnsResponse {
  data: Turn[];
  pagination: TokenPagination;
}

export interface ListSessionEventsResponse {
  data: SessionEventItem[];
  pagination: TokenPagination;
}
```

### 行为化会话表面（类 → 结构化粘贴）

网关以类的形式提供 `AgentSession` / `AgentDraftSession`。对于面向 BYO 的
副本，将其视为 DTO + 方法：

```ts
export interface PreparedTurn {
  execute(opts: { stream: false; pollIntervalMs?: number }): Promise<TurnState>;
  execute(opts?: { stream?: true }): AsyncIterable<TurnStreamData>;
}

export interface AgentSession extends Session {
  prepareTurn(opts?: { input?: TurnInputItem[]; previousTurnId?: PreviousTurnIdInput }): PreparedTurn;
  listTurns(opts?: { pageToken?: string; limit?: number }): Promise<PageResult<Turn, ListTurnsResponse>>;
  getTurn(req: { turnId: string }): Promise<Turn>;
  cancel(): Promise<void>;
  listEvents(opts?: {
    pageToken?: string;
    lastTurnId?: string;
    limit?: number;
  }): Promise<PageResult<SessionEventItem, ListSessionEventsResponse>>;
}

export interface AgentDraftSession extends DraftSession {
  update(req?: { agentSpec?: AgentSpec }): Promise<void>;
  prepareTurn(opts?: { input?: TurnInputItem[]; previousTurnId?: PreviousTurnIdInput }): PreparedTurn;
  listTurns(opts?: { pageToken?: string; limit?: number }): Promise<PageResult<Turn, ListTurnsResponse>>;
  getTurn(req: { turnId: string }): Promise<Turn>;
  cancel(): Promise<void>;
  listEvents(opts?: {
    pageToken?: string;
    lastTurnId?: string;
    limit?: number;
  }): Promise<PageResult<SessionEventItem, ListSessionEventsResponse>>;
}
```

---

## 服务端端口（使用上述副本）

```ts
export interface AgentChatServer {
  createSession(req: { agentName: string; tfyMetadata?: string }): Promise<AgentSession>;
  // listSessions(
  //   req: { agentName: string } & PageParams,
  // ): Promise<PageResult<AgentSession, ListSessionsResponse>>;
  getSession(req: { sessionId: string }): Promise<AgentSession>;

  createDraftSession(req: {
    agentSpec: AgentSpec;
    agentName?: string;
    tfyMetadata?: string;
  }): Promise<AgentDraftSession>;
  getDraftSession(req: { draftSessionId: string }): Promise<AgentDraftSession>;
  // listDraftSessions(
  //   req?: { agentName?: string } & PageParams,
  // ): Promise<PageResult<AgentDraftSession, ListDraftSessionsResponse>>;
  listSessions( // 用于所有会话列表
    req?: { agentName?: string } & PageParams, // agentName 过滤器
  ): Promise<PageResult<AgentSession | AgentDraftSession, ListOwnedSessionsResponse>>;
  updateDraftSession(req: { draftSessionId: string; agentSpec?: AgentSpec }): Promise<AgentDraftSession>;

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
  deleteSession?(req: { sessionId: string }): Promise<void>;
  listTurns(req: {
    sessionId: string;
    limit?: number;
    pageToken?: string;
  }): Promise<PageResult<Turn, ListTurnsResponse>>;
  getTurn(req: { sessionId: string; turnId: string }): Promise<Turn>;
  listEvents(req: {
    sessionId: string;
    pageToken?: string;
    lastTurnId?: string;
    limit?: number;
  }): Promise<PageResult<SessionEventItem, ListSessionEventsResponse>>;
  subscribeToTurn?(req: {
    sessionId: string;
    turnId: string;
    afterSequenceNumber?: number;
  }): AsyncIterable<TurnStreamData>;

  downloadSandboxFile?(sandboxId: string, req: { path: string }): Promise<unknown>; // 网关中为 BinaryResponse
}

export interface AgentBuilderServer<
  TModel extends ModelEntry = ModelEntry,
  TSkill extends AgentSkill = AgentSkill,
  TMcp extends ConnectorState = ConnectorState,
  TAgent extends AgentLibraryEntry = AgentLibraryEntry,
  TSave = unknown,
> {
  getModels(): Promise<TModel[]>;
  getSkills(): Promise<TSkill[]>;
  getMcp(): Promise<TMcp[]>;
  searchAgents(req?: SearchAgentsParams): Promise<TAgent[]>;
  saveAgent(req: { agentName: string; agentSpec: AgentSpec; draftSessionId?: string }): Promise<TSave>;
  deleteAgent?(req: { agentName: string }): Promise<void>;
}

export type AgentUIServer<
  TModel extends ModelEntry = ModelEntry,
  TSkill extends AgentSkill = AgentSkill,
  TMcp extends ConnectorState = ConnectorState,
  TAgent extends AgentLibraryEntry = AgentLibraryEntry,
  TSave = unknown,
> = AgentChatServer & AgentBuilderServer<TModel, TSkill, TMcp, TAgent, TSave>;
```

---

## 此粘贴中的备注 / 缺口

- `TurnStreamingEvent` / `SessionEvent` / 部分 `TurnStateDone` 叶子节点为**摘要形式**（`unknown` / `{ type: string }`）——完整事件载荷是大型网关联合类型；按需展开。
- 网关中的 `Page` 是一个**类**；上述 `PageResult` 是结构化替代品。
- 实际的 `src/server/types.ts` 仍**导入/重新导出**网关——此文件仅为文档 / 升级草案。