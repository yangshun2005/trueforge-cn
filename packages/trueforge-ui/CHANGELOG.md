# 更新日志

## 0.3.0-rc.0

### 次要更改

- 3539da2: 添加 `brand.mode`（`icon-title` | `icon-only` | `logo`），以便宿主首先选择外观样式；`name` 始终作为标记的标签，`resolveBrandChrome` 将模式映射到布局外观。
- 16feb29: 添加 `customActionRenderers`，以便宿主可以在客户端工具上使用自己的 UI 暂停编辑器，并通过 `onSubmit(content)` 恢复。
- 0297727: 添加上下文管理压缩触发器，具有模型感知的默认值，并迁移持久化的旧版令牌阈值。

### 补丁更改

- b11cfc3: 接入远程聊天会话删除功能，并改进历史记录删除操作样式。
- 3ca4e2e: 修复按钮组件的图标和内边距，删除 trueforge-ui 包中冲突的本地 prettier.json。
- a655537: 更新已发布的依赖范围（AI SDK、Hono、MCP SDK、Redis、assistant-ui 及相关包）。
- aa4be44: 在助手消息中，在新标签页中打开 Markdown 链接。
- d7015fd: 在工具审批时显示 MCP 服务器工具名称。
- 更新依赖 [648273b]
  - @truefoundry/trueforge-sdk@0.1.4-rc.0

## 未发布

### 次要更改

- **`brand.mode`** — 明确的外观样式。设置 `mode`，然后传递其所需的字段。
  `name` 始终作为标记的标签（`alt` / `aria-label`）：
  - **`'icon-title'`** — `name` + 可选 `icon`（标题显示在展开的外观中）
  - **`'icon-only'`** — `name` + `icon`（保留 alt，无标题文本）
  - **`'logo'`** — `name` + `icon` + `logo`（文字商标显示在展开的外观中；`name` 仅作为 alt）
  - **默认** — 省略 `brand` 以使用 TrueForge 文字商标 / 方形标记
- **`resolveBrandChrome()`** — 将 `brand.mode` 映射到
  `{ expandedVariant, collapsedVariant, showTitle }`。`SidebarLayout` 使用它；自定义
  布局也应使用。

### 更改

- **`BrandConfig`** — 根据 `mode`（`BrandMode`）进行区分。从
  配置中移除 `showTitle`；可见标题遵循模式。

## 0.2.4

### 补丁更改

- f1c5c6b: 当 UI 服务器未暴露目录时，回退到待处理的 MCP 授权 URL。

## 0.2.3

### 补丁更改

- abd9e38: 代理库行：在技能之前显示连接器，使用编辑器的灯泡图标表示技能，并在悬停时通过工具提示显示连接器/技能名称。
- cc49d4a: 重命名目录、沙盒文件下载和 MCP 工具路径；Fern upsert 变为 create_or_update。会话和轮次的默认值和最大值为 25；会话和轮次事件列表的默认值和最大值为 100。
- d7a640f: 统一 AgentSpec、设置、目录和聊天选择器中的 OpenAPI 类型名称：Catalog/Configured/Available 资源视图、AgentSpec 嵌套的 Model/Skill/InitialUserMessage、Put*Request → Update*Request、MCP 缩写大小写、GetMeResponse，以及嵌套 AgentSpec/capabilities 模式的显式名称。
- 7e2e02c: 每当设置关闭时刷新编辑器目录，包括导航到聊天或命名代理时。
- 更新依赖 [cc49d4a]
  - @truefoundry/trueforge-sdk@0.1.3

## 0.2.3-rc.0

### 补丁更改

- abd9e38: 代理库行：在技能之前显示连接器，使用编辑器的灯泡图标表示技能，并在悬停时通过工具提示显示连接器/技能名称。
- cc49d4a: 重命名目录、沙盒文件下载和 MCP 工具路径；Fern upsert 变为 create_or_update。会话和轮次的默认值和最大值为 25；会话和轮次事件列表的默认值和最大值为 100。
- d7a640f: 统一 AgentSpec、设置、目录和聊天选择器中的 OpenAPI 类型名称：Catalog/Configured/Available 资源视图、AgentSpec 嵌套的 Model/Skill/InitialUserMessage、Put*Request → Update*Request、MCP 缩写大小写、GetMeResponse，以及嵌套 AgentSpec/capabilities 模式的显式名称。
- 7e2e02c: 每当设置关闭时刷新编辑器目录，包括导航到聊天或命名代理时。
- 更新依赖 [cc49d4a]
  - @truefoundry/trueforge-sdk@0.1.3-rc.0

## 0.2.2

### 补丁更改

- 3113aa4: 将 MCP 服务器 SDK 方法从 `deleteAuthorize` 重命名为 `deleteAuthorization`。
- 45dc6cd: 将 MCP 授权的 `redirect_url` 替换为同源的 `return_to` 路径，以防止 OAuth 后的开放重定向。
- 更新依赖 [3113aa4]
- 更新依赖 [45dc6cd]
  - @truefoundry/trueforge-sdk@0.1.2

## 0.2.1

### 补丁更改

- c5223ad: 将 `react-dom` 和 `react-router-dom` 作为包依赖而非对等依赖发布，以便消费者无需单独安装它们。
- 92ee970: 解码面向用户的 API 错误消息中的常见转义序列（`\n`、`\t`、`\r`、`\uXXXX` 等），并使用 `whitespace-pre-wrap` 渲染它们，以便多行 Zod 验证输出正确显示。
- 53104b1: 保持活动状态和记忆的草稿沙盒设置与服务器能力同步。

## 0.2.0

### 次要更改

- 9a4d1a7: 为 shell 位置（`/`、`/agents/:agentName`、`/sessions/:sessionId`、`/settings`）添加可选的 `withRouter` URL 同步，可通过 `routes` 自定义路径，并将 `react-router-dom` 作为可选对等依赖。从 TrueForge 服务器为客户端深层链接提供应用 shell。

### 补丁更改

- 更新依赖 [5100c59]
  - @truefoundry/trueforge-sdk@0.1.1

## 0.1.0

### 次要更改

- 2f08a99: 通过 localStorage 在新建聊天和重新加载时记住纯草稿编辑器选择（模型、技能、MCP 连接器、配置）。编辑流程和不可变的库代理保持不变。`TrueForgeUI` 的 `layout` 是可选的，默认为 `"sidebar"`。
- b56c003: 所有公共包的初始 0.1.0-rc.1 预发布版本。

### 补丁更改

- e9bf976: 将设置 MCP、技能、模型提供者和沙盒的创建/提交主体包装为 `{ manifest }`。列表/获取项目嵌套存储的文档（`name` 加 `manifest`，以及派生字段）。创建返回 201。聊天列表和目录保持扁平。适配器目录遵循新的 SDK 结构。
- a9e8187: 改进嵌套工具审批处理，并升级 `@truefoundry/assistant-ui-runtime`。
- 0730213: 保存代理对话框：为每个连接器添加**预加载**切换（写入 `mcp_servers[].preload`），将能力渲染为并排卡片，并精简模态框 — 内联模型/连接器/技能标签（模型显示其提供者徽标）、“连接器”标签、无副标题、内容自适应高度和更紧凑的间距。工具提示现在传送到最近的 `<dialog>` 中，以便在模态内容上方渲染。
- 更新依赖 [e9bf976]
- 更新依赖 [b56c003]
  - @truefoundry/trueforge-sdk@0.1.0

## 0.1.0-rc.2

### 次要更改

- 2f08a99: 通过 localStorage 在新建聊天和重新加载时记住纯草稿编辑器选择（模型、技能、MCP 连接器、配置）。编辑流程和不可变的库代理保持不变。`TrueForgeUI` 的 `layout` 是可选的，默认为 `"sidebar"`。

### 补丁更改

- a9e8187: 改进嵌套工具审批处理，并升级 `@truefoundry/assistant-ui-runtime`。
- 0730213: 保存代理对话框：为每个连接器添加**预加载**切换（写入 `mcp_servers[].preload`），将能力渲染为并排卡片，并精简模态框 — 内联模型/连接器/技能标签（模型显示其提供者徽标）、“连接器”标签、无副标题、内容自适应高度和更紧凑的间距。工具提示现在传送到最近的 `<dialog>` 中，以便在模态内容上方渲染。

## 0.1.0-rc.1

### 补丁更改

- e9bf976: 将设置 MCP、技能、模型提供者和沙盒的创建/提交主体包装为 `{ manifest }`。列表/获取项目嵌套存储的文档（`name` 加 `manifest`，以及派生字段）。创建返回 201。聊天列表和目录保持扁平。适配器目录遵循新的 SDK 结构。
- 更新依赖 [e9bf976]
  - @truefoundry/trueforge-sdk@0.1.0-rc.1

## 0.1.0-rc.0

### 次要更改

- b56c003: 所有公共包的初始 0.1.0-rc.1 预发布版本。

### 补丁更改

- 更新依赖 [b56c003]
  - @truefoundry/trueforge-sdk@0.1.0-rc.0

本项目所有值得注意的更改都将记录在此文件中。

格式基于 [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)，
本项目遵循 [语义化版本控制](https://semver.org/spec/v2.0.0.html)。

## [未发布]

### 更改

- **`@truefoundry/assistant-ui-runtime` ≥ 0.1.18** — 依赖于整合的
  运行时版本，包括：
  - `getModels` → `properties.reasoningEfforts` 用于支持思考的模型
  - extras 祖先遍历，以便嵌套/只读 AUI 客户端解析
    `useTrueFoundryRespondToToolApproval`（子代理允许/拒绝）
  - 并行启用的模型 + 提供者元数据 CP 获取
  - `ModelSelectorEntry.name` / `id` = `model_fqn`（`account/model-id`）
  - 网关挂载清理（`normalizeMcpMount` / `normalizeSkillMount`，
    注册表 `enableTools` 默认 `["@all"]`）

### 修复

- **子代理允许/拒绝** — `create_sub_agent` 内的嵌套工具审批现在
  使用 _嵌套_ 工具的审批 ID 调用 `useTrueFoundryRespondToToolApproval`。
  旧的桥接基于外部子代理部分（通常没有审批），因此点击静默无效。
  需要 `@truefoundry/assistant-ui-runtime` ≥ 0.1.19，以便 extras 通过只读嵌套 AUI 客户端解析。
- **沙盒提供者契约** — 从 `SandboxProviderConfig` 中移除 `snapshotName`，
  并将 `listSandboxProviders` 包装为 `{ data, snapshotSyncStatus }`（Harness
  `status` / `status_reason`）。重新导出 `SandboxProviderListEntry` /
  `SandboxSnapshotSyncStatus`。

### 新增

- **草稿编辑器偏好** — 纯草稿模型、技能、MCP 连接器和
  配置选择会被记住（内存 + `localStorage`），并在重新加载后作为下一次
  新建聊天的种子。目录加载后，过期的目录条目会被清理。
  编辑流程（设置了 `agentId`）和不可变的库代理保持不变。
- **可选的 `layout`** — `<TrueForgeUI />` 的 `layout` 属性默认为 `"sidebar"`。
- **`type: "trueforge"` 内置服务器** — `<TrueForgeUI server={{ type: 'trueforge', baseUrl?, token?, fetch? }} />`
  通过 `@truefoundry/trueforge-ui/plugins/trueforge-agent-server-adapter`
  （`createTrueForgeAgentUIServer`）解析完整的 Harness `AgentUIServer`，
  包括聊天、构建器和默认设置目录。依赖于 `@truefoundry/trueforge-sdk`
  （monorepo 内为 workspace:*；打包中为外部依赖）。
- **亮/暗品牌徽标** — `theme.brand.logo` 接受 `BrandLogoConfig`
  （`{ src?, light?, dark?, href? }`）。源文件根据解析的主题
  模式选择，回退到另一种模式，然后回退到 `src`，因此单个配置的模式
  涵盖两者，仅 `{ light }` 永远不会渲染缺失的图像。`href` 将
  徽标包装为同标签页链接。纯字符串（`logo: '/logo.svg'`）是
  与模式无关的源的简写。没有可用源的配置回退到默认
  标记，而不是空的 `<img>`。
- **`BrandLogo` 插槽** — 现在位于插槽表中，并通过 `useSlot` 由
  每个布局解析，因此 `overrides={{ BrandLogo: MyMark }}` 可到达侧边栏头部、
  小部件浮动按钮和欢迎屏幕。
- **`useBrandName()`** — 配置的品牌名称或默认名称，因此布局不再
  内联回退字符串。

### 更改

- **主题预设重命名** — 默认预设 ID `truefoundry` → `trueforge`
  （`ThemePreset`、`PRESETS`、`data-preset`、文档）。
- **样式自动注入** — `ThemeProvider` 在运行时注入 SDK 样式表。
  宿主不再需要 `@import '@truefoundry/trueforge-ui/styles.css'`（保留
  导出用于 SSR）。语义令牌和暗色模式限定在 `.aui-theme-root`
  范围内（不对宿主页面进行 `html.dark` / `:root` 重新主题化）。
- **依赖** — `@truefoundry/assistant-ui-runtime` 固定为 `0.1.13`。
  `ModelSelectorEntry` 现在需要 `id`、嵌套的 `provider: ProviderEntry` 和
  `properties: ModelProperties`（替换扁平的 `provider` / `providerLogo` /
  `reasoningEfforts`）。重新导出 `ProviderEntry` 和 `ModelProperties`。

### 修复

- **历史记录行可变性** — 优先使用会话中的 `custom.isMutable`（运行时
  标记），而不是从 `agentName` 推断。孤立的命名引用（已删除的代理，无
  `agentName`）作为不可变打开。`openHistorySession` 允许 `isMutable: false`
  而无需代理名称。
- **编辑绑定的草稿历史切换** — 当 shell 仍带有编辑 `agentName`/`agentId`
  且行是不同的会话时，不要通过 `switchToThread` 重用可变 shell。
  通过 `openHistorySession` 重新挂载，以便更新代理外观不会覆盖错误的库代理。

### 破坏性更改

- **服务器端口类型由 `@truefoundry/assistant-ui-runtime` 拥有** — 此包不再
  在本地定义 `AgentChatServer` / `AgentBuilderServer` / 目录 DTO。
  面向宿主的类型通过 `src/server/types.ts` 从 `@truefoundry/assistant-ui-runtime/server`
  重新导出。优先从 `@truefoundry/trueforge-ui` 导入类型
  （宿主不应需要运行时包来获取类型）。需要运行时 `0.1.13`。
- **移除公共分页辅助函数** — `PageResult`、`TokenPagination`、
  `ListSessionsResponse`、`ListTurnsResponse`、`ListSessionEventsResponse` 和
  不透明的 `SessionEvent` 从此包中移除。聊天列表 API 使用运行时
  `ListResult<T>`（`{ data, nextPageToken? }`）。
- **`AgentUIServer` / 挂载遵循运行时契约** — 组合端口匹配运行时
  `AgentUIServerPort`；`SkillMount` / `McpServerMount` 保持不透明的 `object`
  （宿主通过泛型扩展）。在每个构建器宿主上实现 `getCapabilities()`。
- **单一品牌组件** — 移除 `BrandIcon` 导出；`BrandLogo` 是唯一的产品
  标记，仅渲染标记本身。还需要名称作为文本的调用者将其与
  `useBrandName()` 配对（如侧边栏头部所做），而不是依赖
  硬编码一种排列的第二个组件。将 `<BrandIcon />` 替换为
  `<BrandLogo />`，将 `overrides={{ BrandIcon }}` 替换为 `overrides={{ BrandLogo }}`。
- **`brand.name` 为必填** — `theme.brand` 保持可选，但设置它现在
  需要 `name`，因为它作为徽标的标签，没有名称的徽标没有可访问的
  名称。`logo` 保持可选，因此 `brand: { name: 'Acme' }` 仍将宿主文本
  与默认标记配对。`useBrand()` 返回 `Partial<BrandConfig>`，因为 `brand`
  可能完全不存在。
- **`theme.brand` 仅限图像** — 移除 `brand.icon`；只有一个徽标源