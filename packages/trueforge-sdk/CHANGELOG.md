## [0.1.4-rc.1] - 2026-09-03

## [0.1.4-rc.0] - 2026-08-27

## 0.1.4-rc.1

### Patch Changes

- ecdadb2: Regenerate SDK from updated OpenAPI spec.
- ecdadb2: Regenerate SDK from updated OpenAPI spec.
- ecdadb2: Add `internal.agents.getCodeSnippets` API under the new SDK `internal` namespace.
- ecdadb2: Add tenant-unique optional session `external_id`, `Sessions.getOrCreateByExternalId`, and an idempotent `POST /internal/sessions/get-or-create-by-external-id` endpoint and SDK method.
- ecdadb2: Add caller-owned session `metadata` (`Record<string, string>` with size limits) on create, update, and read. Persist as a new `session.metadata` jsonb column; leave session `custom` unchanged.
- ecdadb2: Add TrueFoundry-managed MCP list/get (SFY registry, gateway proxy URL, create/update 424).

## [0.1.3] - 2026-08-19

## 0.1.4-rc.0

### 补丁变更

- 648273b: 根据更新的 OpenAPI 规范重新生成 SDK。

## [0.1.3-rc.0] - 2026-08-19

## 0.1.3

### 补丁变更

- cc49d4a: 根据更新的 OpenAPI 规范重新生成 SDK。

## [0.1.2] - 2026-08-18

## 0.1.3-rc.0

### 补丁变更

- cc49d4a: 根据更新的 OpenAPI 规范重新生成 SDK。

## [0.1.1] - 2026-08-17

## 0.1.2

### 补丁变更

- 3113aa4: 根据更新的 OpenAPI 规范重新生成 SDK。
- 45dc6cd: 将 MCP 授权的 `redirect_url` 替换为同源的 `return_to` 路径，以防止 OAuth 后的开放重定向。

## [0.1.0] - 2026-08-16

## 0.1.1

### 补丁变更

- 5100c59: 根据更新的 OpenAPI 规范重新生成 SDK。

## [0.1.0-rc.1] - 2026-08-14

## 0.1.0

### 次要变更

- b56c003: 所有公共包的初始 0.1.0-rc.1 预发布版本。

### 补丁变更

- e9bf976: 将设置 MCP、技能、模型提供者和沙箱的创建/放置请求体包装为 `{ manifest }`。列表/获取项目将存储的文档（`name` 加上 `manifest`，以及派生字段）嵌套。创建操作返回 201。聊天列表和目录保持扁平结构。适配器目录遵循新的 SDK 格式。

## [0.1.0-rc.0] - 2026-08-13

## 0.1.0-rc.1

### 补丁变更

- e9bf976: 将设置 MCP、技能、模型提供者和沙箱的创建/放置请求体包装为 `{ manifest }`。列表/获取项目将存储的文档（`name` 加上 `manifest`，以及派生字段）嵌套。创建操作返回 201。聊天列表和目录保持扁平结构。适配器目录遵循新的 SDK 格式。

# @truefoundry/trueforge-sdk

## 0.1.0-rc.0

### 次要变更

- b56c003: 所有公共包的初始 0.1.0-rc.1 预发布版本。