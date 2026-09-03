# @truefoundry/trueforge

## 0.2.0-rc.0

### 次要更改

- 0297727: 添加上下文管理压缩触发器，具备模型感知默认值，并迁移持久化的旧版令牌阈值。

### 补丁更改

- 3539da2: 添加 `brand.mode`（`icon-title` | `icon-only` | `logo`），使宿主优先选择界面外观；`name` 始终标记徽标，`resolveBrandChrome` 将模式映射到布局外观。
- 940c4e5: 优先使用 MCP Python SDK 2.0 的 snake_case 工具注解字段，并为旧版 SDK 提供 camelCase 回退，不改变破坏性工具检测行为。
- a655537: 更新已发布的依赖范围（AI SDK、Hono、MCP SDK、Redis、assistant-ui 及相关包）。
- 5ccac3d: 更新 /healthz 以返回 JSON 状态和包版本。
- fba6129: 要求 Node.js 22.14+（`better-sqlite3` v13 为 Node-API 10 构建，在 22.13 及以下版本会触发 SIGSEGV）。
- 更新依赖 [940c4e5]
- 更新依赖 [a655537]
- 更新依赖 [0297727]
  - @truefoundry/trueforge-core@0.2.0-rc.0

## 0.1.4

### 补丁更改

- 42eee39: 启用独立的纯内存本地沙箱回退（无设置行），持久化花式 `v1:type:raw` 沙箱 ID，移除租户前缀所有权检查，保持 TFY 沙箱写入相对于当前工作目录（无 `/opt` / `/usr/local`），让每个沙箱提供者管理 PATH（Sandbox 中无硬编码的 Daytona 尾部），并仅授予 SRT 中 Code Mode 套接字父目录权限（而非宿主 `/tmp`）。
- cc49d4a: 重命名目录、沙箱文件下载和 MCP 工具路径；Fern upsert 变为 create_or_update。会话和轮次默认及最大值为 25；会话和轮次事件列表默认及最大值为 100。
- d7a640f: 统一 AgentSpec、设置、目录和聊天选择器中的 OpenAPI 类型名称：Catalog/Configured/Available 资源视图，AgentSpec 嵌套的 Model/Skill/InitialUserMessage，Put*Request → Update*Request，MCP 缩写大小写，GetMeResponse，以及嵌套的 AgentSpec/capabilities schema 的显式名称。
- 6251d2a: 从 SDK 中省略 POST /api/v1/auth/logout；UI 直接 POST 到清除 Cookie 的路径。
- 2c3278e: 将重放的 OIDC 回调（成功登录后浏览器返回）视为已登录，而非 `/?error=login_failed`，并在会话仍有效时忽略该过期查询。
- 5e03c3d: 将 Mintlify API Reference 分组缩减为 Auth、Capabilities、Models、MCP Servers、Skills、Sandboxes、Agents 和 Agent Sessions。
- 更新依赖 [42eee39]
- 更新依赖 [d7a640f]
- 更新依赖 [7ae5376]
- 更新依赖 [889caca]
- 更新依赖 [2ca7fb2]
- 更新依赖 [43d780e]
  - @truefoundry/trueforge-core@0.1.4

## 0.1.4-rc.0

### 补丁更改

- 42eee39: 启用独立的纯内存本地沙箱回退（无设置行），持久化花式 `v1:type:raw` 沙箱 ID，移除租户前缀所有权检查，保持 TFY 沙箱写入相对于当前工作目录（无 `/opt` / `/usr/local`），让每个沙箱提供者管理 PATH（Sandbox 中无硬编码的 Daytona 尾部），并仅授予 SRT 中 Code Mode 套接字父目录权限（而非宿主 `/tmp`）。
- cc49d4a: 重命名目录、沙箱文件下载和 MCP 工具路径；Fern upsert 变为 create_or_update。会话和轮次默认及最大值为 25；会话和轮次事件列表默认及最大值为 100。
- d7a640f: 统一 AgentSpec、设置、目录和聊天选择器中的 OpenAPI 类型名称：Catalog/Configured/Available 资源视图，AgentSpec 嵌套的 Model/Skill/InitialUserMessage，Put*Request → Update*Request，MCP 缩写大小写，GetMeResponse，以及嵌套的 AgentSpec/capabilities schema 的显式名称。
- 6251d2a: 从 SDK 中省略 POST /api/v1/auth/logout；UI 直接 POST 到清除 Cookie 的路径。
- 2c3278e: 将重放的 OIDC 回调（成功登录后浏览器返回）视为已登录，而非 `/?error=login_failed`，并在会话仍有效时忽略该过期查询。
- 5e03c3d: 将 Mintlify API Reference 分组缩减为 Auth、Capabilities、Models、MCP Servers、Skills、Sandboxes、Agents 和 Agent Sessions。
- 更新依赖 [42eee39]
- 更新依赖 [d7a640f]
- 更新依赖 [7ae5376]
- 更新依赖 [889caca]
- 更新依赖 [2ca7fb2]
- 更新依赖 [43d780e]
  - @truefoundry/trueforge-core@0.1.4-rc.0

## 0.1.3

### 补丁更改

- 3113aa4: 将 MCP 服务器 SDK 方法从 `deleteAuthorize` 重命名为 `deleteAuthorization`。
- 45dc6cd: 将 MCP 授权 `redirect_url` 替换为同源的 `return_to` 路径，以防止 OAuth 后的开放重定向。
- c546350: 在 `SandboxOptions` 上传递 `tenantName`，而非通过 exec 环境注入 `TFY_TENANT_NAME`。
- 更新依赖 [08700d1]
- 更新依赖 [c546350]
  - @truefoundry/trueforge-core@0.1.3

## 0.1.2

### 补丁更改

- 9485811: 当 MCP 服务器 URL 更改时清除 DCR OAuth 令牌和待处理的授权，因为 URL 是令牌受众。
- 5b981ab: 即使拥有执行器已消失（重启）或 Redis 无法确认中止，也取消会话。在存储中冻结正在运行的轮次，以便新轮次可以开始。`freezeAndGetTurn` 现在接受取消原因（barge-in 保持 `cancelled-for-next-turn`；显式取消保持 `client-cancelled`）。Redis 超时和传输故障仍会冻结，并警告取消不干净。
- 更新依赖 [363a522]
- 更新依赖 [5b981ab]
  - @truefoundry/trueforge-core@0.1.2

## 0.1.1

### 补丁更改

- 69237db: 在沙箱提供者配置时等待 Daytona 快照注册，以便认证失败返回 422 而非虚假的待处理状态，并保持 GET 状态刷新的持久化。
- f056973: 通过配置驱动的 Hono bodyLimit（MAX_REQUEST_BODY_BYTES）拒绝过大的 HTTP 请求体并返回 413。
- 9a4d1a7: 为 shell 位置（`/`、`/agents/:agentName`、`/sessions/:sessionId`、`/settings`）添加可选的 `withRouter` URL 同步，通过 `routes` 自定义路径，并将 `react-router-dom` 作为可选对等依赖。从 TrueForge 服务器为客户端深层链接提供应用 shell。
- 更新依赖 [69237db]
- 更新依赖 [7783fc0]
  - @truefoundry/trueforge-core@0.1.1

## 0.1.0

### 次要更改

- b56c003: 所有公共包的初始 0.1.0-rc.1 预发布。

### 补丁更改

- e9bf976: 将设置 MCP、技能、模型提供者和沙箱的创建/放置请求体包装为 `{ manifest }`。列表/获取项嵌套存储的文档（`name` 加 `manifest`，以及派生字段）。创建返回 201。聊天列表和目录保持扁平。适配器目录遵循新的 SDK 形态。
- 更新依赖 [b56c003]
  - @truefoundry/trueforge-core@0.1.0

## 0.1.0-rc.1

### 补丁更改

- e9bf976: 将设置 MCP、技能、模型提供者和沙箱的创建/放置请求体包装为 `{ manifest }`。列表/获取项嵌套存储的文档（`name` 加 `manifest`，以及派生字段）。创建返回 201。聊天列表和目录保持扁平。适配器目录遵循新的 SDK 形态。

## 0.1.0-rc.0

### 次要更改

- b56c003: 所有公共包的初始 0.1.0-rc.1 预发布。

### 补丁更改

- 更新依赖 [b56c003]
  - @truefoundry/trueforge-core@0.1.0-rc.0