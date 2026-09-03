# @truefoundry/trueforge-core

## 0.2.0-rc.0

### 次要更改

- 0297727: 添加上下文管理压缩触发器，采用模型感知的默认值，并迁移已持久化的旧版令牌阈值。

### 补丁更改

- 940c4e5: 优先使用 MCP Python SDK 2.0 的 snake_case 工具注解字段，并为旧版 SDK 提供 camelCase 回退，不改变破坏性工具检测行为。
- a655537: 更新已发布的依赖范围（AI SDK、Hono、MCP SDK、Redis、assistant-ui 及相关包）。

## 0.1.4

### 补丁更改

- 42eee39: 启用独立的纯内存本地沙箱回退（无设置行），持久化花式 `v1:type:raw` 沙箱 ID，移除租户前缀所有权检查，保持 TFY 沙箱写入相对于当前工作目录（无 `/opt` / `/usr/local`），让每个沙箱提供方管理自己的 PATH（Sandbox 中无硬编码的 Daytona 尾部），并且仅授予 SRT 中 Code Mode 套接字父目录权限（而非宿主机 `/tmp`）。
- d7a640f: 统一 AgentSpec、设置、目录和聊天选择器中的 OpenAPI 类型名称：Catalog/Configured/Available 资源视图、AgentSpec 嵌套的 Model/Skill/InitialUserMessage、Put*Request → Update*Request、MCP 缩写大小写、GetMeResponse，以及嵌套 AgentSpec/capabilities schema 的显式名称。
- 7ae5376: 将 SANDBOX_IMAGE_URI 更新为 CI 推送的镜像。
- 889caca: 持久化 `model.message`，省略空内容并延迟 `call_tool` 包装器 `tool_info`，与实时流保持一致。
- 2ca7fb2: 从公共 AgentSpec 中移除未使用的 `config.sandbox.network_policy`（git 认证注入）。
- 43d780e: 从 sandboxImage.json 加载 SANDBOX_IMAGE_URI，以便 CI 可以重写固定版本。

## 0.1.4-rc.0

### 补丁更改

- 42eee39: 启用独立的纯内存本地沙箱回退（无设置行），持久化花式 `v1:type:raw` 沙箱 ID，移除租户前缀所有权检查，保持 TFY 沙箱写入相对于当前工作目录（无 `/opt` / `/usr/local`），让每个沙箱提供方管理自己的 PATH（Sandbox 中无硬编码的 Daytona 尾部），并且仅授予 SRT 中 Code Mode 套接字父目录权限（而非宿主机 `/tmp`）。
- d7a640f: 统一 AgentSpec、设置、目录和聊天选择器中的 OpenAPI 类型名称：Catalog/Configured/Available 资源视图、AgentSpec 嵌套的 Model/Skill/InitialUserMessage、Put*Request → Update*Request、MCP 缩写大小写、GetMeResponse，以及嵌套 AgentSpec/capabilities schema 的显式名称。
- 7ae5376: 将 SANDBOX_IMAGE_URI 更新为 CI 推送的镜像。
- 889caca: 持久化 `model.message`，省略空内容并延迟 `call_tool` 包装器 `tool_info`，与实时流保持一致。
- 2ca7fb2: 从公共 AgentSpec 中移除未使用的 `config.sandbox.network_policy`（git 认证注入）。
- 43d780e: 从 sandboxImage.json 加载 SANDBOX_IMAGE_URI，以便 CI 可以重写固定版本。

## 0.1.3

### 补丁更改

- 08700d1: 在沙箱镜像中固定 `mcp==1.29.0`，以便 Code Mode 可以导入 `mcp.types`。
- c546350: 在 `SandboxOptions` 上传递 `tenantName`，而不是通过 exec 环境注入 `TFY_TENANT_NAME`。

## 0.1.2

### 补丁更改

- 363a522: 在构建线程之前，从主 toolSets 在 SessionHandle 中仅连接一次共享沙箱 Code Mode。
- 5b981ab: 即使拥有执行器已消失（重启）或 Redis 无法确认中止，也取消会话。在存储中冻结正在运行的轮次，以便可以开始新的轮次。`freezeAndGetTurn` 现在接受取消原因（抢占保持 `cancelled-for-next-turn`；显式取消保持 `client-cancelled`）。Redis 超时和传输故障仍会冻结，并警告取消不干净。

## 0.1.1

### 补丁更改

- 69237db: 在沙箱提供方配置时等待 Daytona 快照注册，以便认证失败返回 422 而不是虚假的待处理状态，并保持 GET 状态刷新持久化。
- 7783fc0: 指示 ask_user_question 在上下文明显偏向某个选项时，将第一个选项标记为（推荐）。

## 0.1.0

### 次要更改

- b56c003: 所有公共包的初始 0.1.0-rc.1 预发布版本。

## 0.1.0-rc.0

### 次要更改

- b56c003: 所有公共包的初始 0.1.0-rc.1 预发布版本。