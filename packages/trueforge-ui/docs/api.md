# 公共 API

`@truefoundry/trueforge-ui` 的精选导出。设计系统原语（`CenteredModal`、`Switch` 等）归属于此包。像 `Button` / `IconButton` 这样的底层控件通过主题令牌 / CSS 进行样式设置（而非 `SlotsProvider` 插槽）；它们的 **属性类型**（`ButtonProps`、`IconButtonProps` 等）被导出，供围绕这些契约构建的主机使用。组件 **值** 仅在其属于公共接口的一部分时才会在下方列出。

## 快速开始

| 导出                               | 说明                                                                                |
| -------------------------------- | ------------------------------------------------------------------------------------ |
| `TrueForgeUI`                    | 插槽 + 运行时 + 内置布局                                                    |
| `TrueForgeUIProps`、`ChatLayout` | Props / 布局联合                                                                 |
| `TrueForgeServerConfig`          | `server` 属性：`type: "truefoundry"` \| `type: "trueforge"` \| 就绪的 `AgentUIServer` |

### 内置服务器

| `server` 配置                                                           | SDK 的功能                                                                       |
| -------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| `{ type: 'truefoundry', apiKey, controlPlaneURL, gatewayPlaneURL? }` | 调用运行时 `createTrueFoundryAgentUIServer`                                          |
| `{ type: 'trueforge', baseUrl?, token?, fetch?, catalog? }`          | 动态导入 `plugins/trueforge-agent-server-adapter` → 完整的 Harness `AgentUIServer` |
| 就绪的 `AgentUIServer`                                                | 透传（主机组合或 `createTrueFoundryServer`）                                |

TrueForge 主机需要 `@truefoundry/trueforge-sdk`。Cookie 应用传递 `fetch`；嵌入通常传递 `token`。

```tsx
// Harness / TrueForge
<TrueForgeUI
  server={{ type: 'trueforge', baseUrl: '/', token: process.env.TRUEFORGE_TOKEN }}
  layout="sidebar"
/>

// 或同源 Cookie：
<TrueForgeUI server={{ type: 'trueforge', fetch: authAwareFetch }} layout="sidebar" />
```

不使用 `<TrueForgeUI />` 的工厂：

```ts
import { createTrueForgeAgentUIServer } from '@truefoundry/trueforge-ui/plugins/trueforge-agent-server-adapter';

const server = createTrueForgeAgentUIServer({ baseUrl: '/', token });
```

Harness 工厂包含可选的 `AgentMetricsServer` 端口，供 agent 详情 Metrics 标签页使用。自定义服务器可以提供带有 `getCharts`、`getMeters` 和 `getChartData` 的 `metrics`；通过 `useAgentMetricsServer` / `useOptionalAgentMetricsServer` 访问它。默认的可视化界面分布在 `AgentMetrics`、`AgentMetricsView`、`AgentMetricsTimeRangeFilter`、`AgentMetricCard` 和 `AgentMetricChart` 插槽中。

## 组合

| 导出                                                        | 说明                           |
| ------------------------------------------------------------- | ------------------------------- |
| `TrueFoundryChatProvider`                                     | 命名 agent 运行时 + 通知提示    |
| `TrueFoundryChatProviderProps`                                | `client` XOR `apiKey`+`baseUrl` |
| `Thread`                                                      | 完整线程 + 编辑器          |
| `ThreadContainer`、`ComposerContainer`、`ThreadListContainer` | 构建块                 |
| `ToasterProvider`、`useToaster`、`useToasterOptional`         | 成功和错误通知提示        |
| 其他 `*Container` 导出                                    | 高级消息 / 工具接线  |

## 插槽 / 主题

| 导出                                                               | 说明                       |
| -------------------------------------------------------------------- | --------------------------- |
| `SlotsProvider`、`useSlot`、`defaultSlots`                           | 覆盖注册表           |
| `SlotOverrides`、`AtomSlots`、`ThemeMode`                            | 类型                       |
| `useTheme`、`useThemeMode`                                           | 主题模式                  |
| 功能原子（`WelcomeScreen`、`ComposerShell`、气泡、通知提示等） | 默认值 + 覆盖目标 |

## Chrome / 运行时

| 导出                                                                                | 说明                                         |
| ------------------------------------------------------------------------------------- | --------------------------------------------- |
| `useAui`、`useAuiState`、`AssistantState`                                             | 也可从 `@truefoundry/trueforge-ui/assistant-ui` 获取 |
| `AgentSessionClient`                                                                  | 网关客户端                                |
| `useTrueFoundryAgentRuntime`、`trueFoundryAttachmentAdapter`、其他 `useTrueFoundry*` | 运行时钩子                                 |

## 覆盖类型

| 导出                                         | 说明                                                            |
| ---------------------------------------------- | ---------------------------------------------------------------- |
| `ButtonProps`、`ButtonSize`、`IconButtonProps` | 包内原语的属性契约（令牌/CSS，而非插槽） |
| 原子 `*Props` 类型                            | 每个原子的覆盖形状                                         |

## 样式

| 导出路径                            | 说明                                                 |
| -------------------------------------- | ----------------------------------------------------- |
| `@truefoundry/trueforge-ui/styles.css` | 令牌 + 工具 + OpenUI（可选；自动注入）                     |