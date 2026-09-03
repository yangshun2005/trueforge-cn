# 自定义 UI

每个精选功能原子都通过 `useSlot` 解析。可通过 `SlotsProvider` 进行覆盖；嵌套的 provider 会回退到父级/默认值。基础组件（`Button`、`IconButton`、`Dialog` 等）是直接导入的——不是插槽——因此请通过主题令牌/CSS 设置其样式，或 Fork 宿主布局。

```tsx
import { SlotsProvider, Thread } from '@truefoundry/trueforge-ui';

function MyWelcome({ heading }: { heading?: string }) {
  return <p className="px-4 py-6 text-center text-lg">{heading ?? 'What are we building today?'}</p>;
}

<SlotsProvider overrides={{ WelcomeScreen: MyWelcome }}>
  <Thread />
</SlotsProvider>;
```

## 主题对象

优先使用 `theme={{ preset, mode, tokens, brand, icons, className, classNames }}`，而不是修改第三方 CSS：

```tsx
<TrueForgeUI
  layout="sidebar"
  theme={{
    preset: 'claude',
    mode: 'dark',
    tokens: { primary: '#e11d48' },
    brand: { mode: 'logo', name: 'Acme', icon: '/brand/icon.svg', logo: '/brand/wordmark.svg' },
    icons: { send: MySendSvg },
    classNames: {
      markdown: 'prose max-w-none',
      inlineCode: 'font-semibold',
      syntaxHighlighter: { root: 'rounded-lg', lineNumber: 'opacity-60' },
      openui: { root: 'my-openui', scope: 'p-2' },
      monaco: { root: 'h-64', editor: 'rounded-lg', monacoTheme: 'vs-dark' },
    },
  }}
/>
```

预设：`trueforge`（默认）、`claude`、`chatgpt`、`gemini`——风格致敬，而非产品复制。宿主 CSS 仍然适用于 `.aui-root` / `.aui-markdown` / `.aui-syntax-highlighter` / `.aui-openui` / `.aui-monaco`。

## 插槽覆盖范围

公开的覆盖面（基础组件保持主题/CSS——不是插槽）：

- **布局/输入区：** `Accordion`、`AccordionSummary`、`AccordionDetails`、`ComposerShell`、`ComposerLeftSection`、`ComposerRightSection`、`ComposerSendButton`、`DraftComposerLeftSection`、`DraftComposerRightSection`、`DraftCompositeSelector`、`DraftModelSelector`、`ThreadRootShell`、`ThreadViewportShell`、`ThreadComposerAreaShell`、`MessageGroup`、`ScrollToBottomButton`、`MessageListSkeleton`、`WelcomeScreen`
- **消息/内容：** `AssistantMessageBubble`、`UserMessageBubble`、`UserMessageEdit`、`UserMessageActionBar`、`MessageActionBar`、`MessageErrorBanner`、`MessageIndicator`、`MessageTimestamp`、`Markdown`、`SyntaxHighlighter`、`OpenUiFenceBlock`、`SandboxArtifactDownload`、`ChatFileDownload`、`MonacoEditorCore`、`CodeEditor`
- **会话列表：** `ThreadListShell`、`ThreadListNewButton`、`ThreadListRow`、`ThreadListRowSkeleton`、`ThreadListEmptyState`、`HistoryLoader`、`AgentsLibrary`、`AgentsLibraryButton`、`SessionsBrowserButton`、`SaveAgentButton`、`SelectAgentEmptyState`、`ClearChatButton`
- **Agent 详情/会话：** `AgentDetailsPage`、`AgentDetailsHeader`、`AgentDetailsTabs`、`AgentDetailsUnavailable`、`AgentOverview`、`AgentOverviewCard`、`AgentSessions`、`AgentSessionsFilters`、`SessionsPage`、`AgentMetrics`、`AgentMetricsView`、`AgentMetricsTimeRangeFilter`、`AgentMetricCard`、`AgentMetricChart`、`AgentSessionListRow`、`AgentSessionDetailHeader`、`AgentSessionMetricsStrip`、`AgentSessionTimelineContainer`、`AgentSessionEventTimeline`、`AgentSessionEventTimelineChart`、`AgentSessionTurnHeader`、`AgentCodeSnippets`、`AgentCodeBlock`
- **附件/通知：** `AttachmentCard`、`AttachmentPreviewDialog`、`AttachmentPickerButton`、`Toast`、`ToastStack`
- **工具/提示词：** `ToolCallCard`、`ToolCallContentBlock`、`ToolApprovalBar`、`ToolGroupCard`、`SubAgentCard`、`SandboxToolCallCard`、`AgentStepsCard`、`ReasoningCard`、`AskUserPrompt`、`McpAuthPrompt`、`ResumeUnavailable`

## URL 路由（`withRouter`）

选择启用浏览器 URL 同步以支持外壳导航。需要在宿主中安装 `react-router-dom`（v6 或 v7）；它是可选的对等依赖，除非设置了 `withRouter`，否则不会被打入包中，因此 Dock/Widget 嵌入以及拥有自己路由器的宿主应保持关闭（默认状态）。

```tsx
<TrueForgeUI server={server} layout="sidebar" withRouter />
```

镜像到 URL 的位置：

- `/` — 新聊天/库首页（取决于模式）
- `/agents/:agentName` — 库 Agent 的不可变“试用”
- `/sessions` — 所有用户的会话浏览器（命名 Agent 和草稿）
- `/sessions/:sessionId` — 特定聊天会话
- `/settings` — 设置浮层（关闭后导航到其下方的聊天位置）
- `/library` — Agents
- `/library/:agentId` — Agent 详情。`?tab=overview|sessions|code|metrics` 选择标签页（默认为 Overview）；当服务器提供可选的 `metrics` 端口时，Metrics 可用

自定义路径（仅在设置 `withRouter` 时生效）。将任何条目设置为 `false` 以保持该位置仅为浮层且无 URL：

```tsx
<TrueForgeUI
  server={server}
  withRouter
  routes={{
    basename: '/app',
    paths: { session: '/chats/:sessionId', libraryAgent: '/library/:agentId', settings: false },
  }}
/>
```

自定义的 `agent` / `session` / `libraryAgent` 模板必须保留其 `:param` 段，否则该位置可以写入 URL 但无法读回。

外壳状态始终是事实来源；路由器仅镜像它。不支持将 `withRouter` 与 `initialSessionId` 结合使用——URL 优先。

行为说明：

- 哈希和宿主拥有的查询键在导航过程中会被保留。会话键（`sessionId`、`agentId`、`tab`、`view`、`s_tw`、`s_sts`、`s_ets`）在目标位置不拥有它们时会被移除，防止过期的过滤器或选择泄漏到不相关的路由中。
- 复制的库会话链接是当前页面上的 `?agentId=&sessionId=`（在设置 `withRouter` 时加上 `/library/:agentId`）。打开它会进入该 Agent 的会话标签页。点击库中的 Agent 会写入 `?tab=overview`，这样遗留的聊天 `sessionId` 不会打开会话页面。当 `withRouter` 关闭时，相同的查询也有效。
- 所有用户的会话页面在启用 `withRouter` 时为 `/sessions`，关闭时为 `?view=sessions`。Agent 和时间过滤器位于查询中（`agentId`、相对时间窗口的 `s_tw`，或绝对时间范围的 `s_sts`/`s_ets`）。打开会话会将 `s_sts`/`s_ets` 固定在 `created_at` 附近（±5 分钟），这样刷新时仍能在第 1 页找到该行，而无需滚动列表。
- `/sessions/:sessionId` 链接通过 `getSession` 解析，因此聊天会以自身的 Agent 绑定和可变性打开，而不是作为新草稿。
- 无法识别的路径（以及格式错误的转义）会规范化为根位置。
- 指向宿主无法承载的位置的 URL（例如没有 Agent 库时的 `/agents/x`，或设置功能被禁用时的 `/settings`）会保留在地址栏中，直到下次导航将其纠正。

提供 SDK 的宿主必须为未知路径发送应用外壳（SPA 回退），否则深层链接会在 React 启动前返回 404。

## 自定义布局

将 React 组件作为 `layout` 传入以拥有自己的界面框架；组合导出的 `Thread` / `ThreadListContainer` / 品牌辅助函数：

```tsx
function CenteredLayout({ className }: { className?: string }) {
  return (
    <div className={`mx-auto flex h-full max-w-3xl flex-col ${className ?? ""}`}>
      <ThreadListContainer />
      <Thread />
    </div>
  );
}

<TrueForgeUI layout={CenteredLayout} server={server} agentMode={…} />;
```

## 重新设置气泡样式

```tsx
import { SlotsProvider, Thread, type AssistantMessageBubbleProps } from '@truefoundry/trueforge-ui';

function AssistantBubbleV2({ children, error, actionBar }: AssistantMessageBubbleProps) {
  return (
    <div className="mr-auto max-w-[75%] rounded-2xl bg-neutral-100 px-4 py-2">
      {children}
      {error}
      {actionBar}
    </div>
  );
}

<SlotsProvider overrides={{ AssistantMessageBubble: AssistantBubbleV2 }}>
  <Thread />
</SlotsProvider>;
```

完整的硬切割设计和迁移说明请参阅 [theming.md](./theming.md)。