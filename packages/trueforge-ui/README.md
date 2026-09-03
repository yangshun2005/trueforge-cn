# @truefoundry/trueforge-ui

[CI](https://github.com/truefoundry/trueforge/actions/workflows/ci.yml)
[npm](https://www.npmjs.com/package/@truefoundry/trueforge-ui)
[许可证](./LICENSE)

一个可主题化、可组合的 React SDK，用于构建生产级 AI 智能体聊天应用。
通过灵活的 UI 层构建您自己的智能体聊天平台，可与 **TrueFoundry**（控制平面 + 网关）、**TrueForge**（Harness）或任何自定义后端配合使用。该 SDK 完全开源，让您完全掌控架构、组件和集成。
由 [assistant-ui](https://www.assistant-ui.com/) 驱动，遵循熟悉的 **shadcn/ui** 和 **Tailwind CSS 变量**主题约定，让您轻松定制体验的每个方面。

**特性**

- **完全可主题化** — 使用受 **TrueFoundry**、**ChatGPT**、**Claude** 和 **Gemini** 启发的内置主题，或创建您自己的品牌标识。
- **设计上可组合** — 灵活替换布局、组件和交互模式，以适应您的产品。
- **后端无关** — 通过 `TrueForgeServerConfig` 连接到 **TrueFoundry**、**TrueForge** 或任何自定义 API。
- **开源** — 无供应商锁定，可扩展、定制和贡献。
- **基于 assistant-ui 构建** — 利用现代 React 基础，无缝集成 shadcn/ui 和 Tailwind CSS。
- **生产就绪** — 专注于您的智能体，SDK 负责处理聊天体验。

自带您的**品牌、颜色、布局、组件和服务器**——Agent SDK 负责连接其余部分。

---

## 目录

- [安装](#安装)
- [快速开始](#快速开始)
- [`<TrueForgeUI />` 属性](#trueforgeui--属性)
- [主题](#主题)
- [内容类名](#内容类名)
- [品牌 / 标志](#品牌--标志)
- [智能体模式](#智能体模式)
- [布局](#布局)
- [自定义布局](#自定义布局)
- [覆盖组件](#覆盖组件)
- [服务器](#服务器)
- [导出](#导出)
- [文档](#文档)
- [许可证](#许可证)

---

## 安装

```bash
npm install @truefoundry/trueforge-ui
# 或
pnpm add @truefoundry/trueforge-ui
# 或
yarn add @truefoundry/trueforge-ui
```

`react` / `react-dom` 是必需的 peer 依赖。**宿主应用必须设置好
[Tailwind CSS](https://tailwindcss.com/)**（推荐 v4），以便 preflight
和应用界面能与 SDK 协同工作。在宿主样式表中：

```css
@import 'tailwindcss';
```

SDK 样式（令牌、工具类、OpenUI）会在 `TrueForgeUI` /
`ThemeProvider` 挂载时自动加载——对于纯客户端应用，您**无需**导入 SDK 样式表。

可选（SSR / 显式加载顺序）：

```css
@import 'tailwindcss';
@import '@truefoundry/trueforge-ui/styles.css';
```

版本范围请参阅 [docs/compatibility.md](./docs/compatibility.md)。

---

## 快速开始

**TrueFoundry** (`type: "truefoundry"`) — 控制平面 + 网关

零配置路径：SDK 根据您的 API 密钥和控制平面 URL（可选显式网关 URL）构建智能体 UI 服务器。

```tsx
import { TrueForgeUI } from '@truefoundry/trueforge-ui';

export default function App() {
  return (
    <div style={{ height: '100dvh' }}>
      <TrueForgeUI
        server={{
          type: 'truefoundry',
          apiKey: process.env.TFY_API_KEY!,
          controlPlaneURL: process.env.TFY_CONTROL_PLANE_URL!,
          // gatewayPlaneURL: process.env.TFY_GATEWAY_URL, // 可选
        }}
        layout="sidebar"
      />
    </div>
  );
}
```

**TrueForge** (`type: "trueforge"`) — Harness

TrueForge / Harness API 的零配置路径。SDK 动态加载
`@truefoundry/trueforge-ui/plugins/trueforge-agent-server-adapter`，构建
HTTP 客户端，并将聊天 + 构建器 + 默认设置目录组合成
`AgentUIServer`。

安装适配器使用的 SDK peer（此 monorepo 中的工作区包）：

```bash
pnpm add @truefoundry/trueforge-sdk
```

**Bearer 令牌（嵌入 / 远程 API）：**

```tsx
import { TrueForgeUI } from '@truefoundry/trueforge-ui';

export default function App() {
  return (
    <div style={{ height: '100dvh' }}>
      <TrueForgeUI
        server={{
          type: 'trueforge',
          baseUrl: process.env.TRUEFORGE_BASE_URL, // 可选；默认 '/'
          token: process.env.TRUEFORGE_TOKEN!,
        }}
        layout="sidebar"
      />
    </div>
  );
}
```

**Cookie / 同源主机** — 省略 `token` 并注入 `fetch`（例如，遵循 OIDC cookie 的认证感知包装器）：

```tsx
<TrueForgeUI
  server={{
    type: 'trueforge',
    baseUrl: '/',
    fetch: authAwareFetch,
  }}
  layout="sidebar"
  agentConfig={{
    mode: 'AgentLibraryWithComposer',
    defaultAgentSpec: { model: { name: '…' } }, // 通常在启动时从 listModels 种子化
  }}
/>
```

可选：传递 `catalog` 以覆盖内置设置目录，或在 `<TrueForgeUI />` 外部需要工厂函数时，从 `@truefoundry/trueforge-ui/plugins/trueforge-agent-server-adapter` 导入 `createTrueForgeAgentUIServer`。

**自定义** — 自带 `AgentUIServer`

当您自行组合聊天和构建器时使用此方式（例如，仅聊天的网关适配器 + 存根目录，或完整的宿主 BFF）。直接传递服务器对象。

```tsx
import { TrueForgeUI, createTrueFoundryServer } from '@truefoundry/trueforge-ui';
import { createTrueFoundryChatServer } from '@truefoundry/assistant-ui-runtime/plugins/truefoundry-agent-server-adapter';

const chatServer = createTrueFoundryChatServer({
  apiKey: process.env.TFY_API_KEY!,
  baseUrl: process.env.TFY_GATEWAY_URL!,
});

const server = createTrueFoundryServer({
  chatServer,
  getModels: async () => [],
  getSkills: async () => [],
  getMcp: async () => [],
  searchAgents: async () => [],
  saveAgent: async () => ({}),
});

export default function App() {
  return (
    <div style={{ height: '100dvh' }}>
      <TrueForgeUI server={server} layout="sidebar" />
    </div>
  );
}
```

## `<TrueForgeUI />` 属性

`<TrueForgeUI />` 是唯一入口点。每项能力都由一个属性驱动。

```tsx
<TrueForgeUI
  server={{
    type: 'truefoundry',
    apiKey: process.env.TFY_API_KEY!,
    controlPlaneURL: process.env.TFY_CONTROL_PLANE_URL!,
  }} // 或 agentServer / { type: "trueforge", token?, baseUrl?, fetch? }
  layout="sidebar" // 'sidebar' | 'drawer' | 'dock' | 'widget' | CustomLayout
  agentConfig={{
    mode: 'AgentLibraryWithComposer', // 省略时默认
    defaultAgentSpec: { model: { name: 'openai-main/gpt-4.1' } },
  }}
  theme={{
    preset: 'claude',
    brand: { mode: 'icon-title', name: 'Acme', icon: '/icon.svg' },
  }}
  overrides={/* 插槽覆盖 */}
  className="h-full"
/>
```

| 属性               | 类型                       | 必填       | 描述                                                                                                        |
| ------------------ | -------------------------- | ---------- | ------------------------------------------------------------------------------------------------------------------ |
| `server`           | `TrueForgeServerConfig`    | ✅         | 内置配置（`truefoundry` / `trueforge`）**或**现成的 `AgentUIServer`。                                      |
| `layout`           | `LayoutProp`               | ✅         | 内置布局字符串**或**自定义 React 组件。                                                            |
| `agentConfig`      | `AgentConfig`              | —          | Shell 模式：SingleAgent / AgentLibrary / AgentComposer / AgentLibraryWithComposer。默认为 library + composer。 |
| `theme`            | `ThemeConfig`              | —          | 预设、模式、令牌、品牌、图标、**内容 `classNames`**（参见[主题](#主题)）。                            |
| `overrides`        | `SlotOverrides`            | —          | 插槽覆盖映射（参见[覆盖组件](#覆盖组件)）。                                                       |
| `className`        | `string`                   | —          | 应用于布局根元素。                                                                                        |
| `initialSessionId` | `string`                   | —          | 恢复特定会话。                                                                                         |
| `onError`          | `(error: unknown) => void` | —          | 宿主错误钩子（运行时 + 服务器初始化）。                                                                           |

后续章节将 `server` 作为 `TrueForgeServerConfig`（通常为 `type: "truefoundry"`）。对于宿主构建的端口，直接传递 `AgentUIServer`。

---

## 主题

传递预设和/或令牌，整个 UI 将自适应。值映射到产品 CSS 变量（`--primary-bg`、`--text-primary`、`--card-bg`、`--primary-button-bg`、`--failure-bg`、`--radius` 等），因此无需任何组件编写特定于调色板的代码。

```tsx
const server = {
  type: 'truefoundry' as const,
  apiKey: process.env.TFY_API_KEY!,
  controlPlaneURL: process.env.TFY_CONTROL_PLANE_URL!,
};

<TrueForgeUI
  server={server}
  layout="sidebar"
  theme={{
    preset: 'claude', // 'trueforge' | 'claude' | 'chatgpt' | 'gemini'
    mode: 'dark', // 省略则为非受控模式（useTheme().setTheme 可用）
    tokens: {
      primaryBg: 'oklch(0.14 0.02 260)',
      textPrimary: 'oklch(0.97 0 0)',
      secondaryBg: 'oklch(0.22 0.02 260)',
      primaryButtonBg: 'oklch(0.55 0.2 275)',
      primaryButtonHover: 'oklch(0.5 0.2 275)',
      primaryButtonText: 'oklch(0.98 0 0)',
      ghostButtonHover: 'oklch(0.8 0.12 200)',
      radius: '0.5rem',
    },
  }}
/>;
```

您也可以从宿主 CSS 在 `.aui-theme-root` 上覆盖令牌（内联 `theme.tokens` 仍优先于 `:root`）：

```css
.aui-theme-root {
  --font-agent-ui: 'Your Font', ui-sans-serif, system-ui, sans-serif;
  --primary-button-bg: oklch(0.55 0.2 275);
}
```

> _截图：同一布局使用自定义调色板重新品牌化。_

有关预设、受控与非受控模式以及更深入的主题说明，请参阅 [docs/theming.md](./docs/theming.md)。

---

## 内容类名

无需替换插槽即可为内容渲染器（markdown、代码围栏、OpenUI、Monaco）设置样式。在 `<TrueForgeUI />` 上传递 `theme.classNames`——值通过 `cn()` 合并到默认值上：

```tsx
<TrueForgeUI
  server={server}
  layout="sidebar"
  theme={{
    classNames: {
      markdown: 'prose prose-neutral dark:prose-invert max-w-none',
      inlineCode: 'font-semibold',
      syntaxHighlighter: {
        root: 'my-code-block rounded-lg',
        pre: 'bg-zinc-950 p-4',
        code: 'text-sm font-mono',
        lineNumber: 'opacity-60',
      },
      openui: { root: 'my-openui-host', scope: 'p-2' },
      monaco: { root: 'my-monaco h-64', editor: 'rounded-lg', monacoTheme: 'vs-dark' },
    },
  }}
/>
```

| `theme.classNames` 键 | 组件 / 表面                             | 字段                                            |
| ---------------------- | ------------------------------------------- | ------------------------------------------------- |
| `markdown`             | `Markdown`（消息散文根元素）             | `string`                                          |
| `inlineCode`           | `Markdown` 行内 `` `code` ``              | `string`                                          |
| `syntaxHighlighter`    | `SyntaxHighlighter`（非 OpenUI 围栏）     | `root`、`pre`、`code`、`lineNumber`               |
| `openui`               | `OpenUiFenceBlock`                          | `root`、`scope`                                   |
| `monaco`               | `MonacoEditorCore`（代码产物 / 差异） | `root`、`editor`、`monacoTheme`（Monaco 主题 ID） |

Provider 下的自定义布局和宿主原子组件可以通过 `useOptionalContentClassNames()` / `useContentClassNames()`（从包中导出）读取同一映射。宿主 CSS 在 `.aui-markdown` / `.aui-syntax-highlighter` / `.aui-openui` / `.aui-monaco` 上仍然可以作为替代方案使用。

---

## 品牌 / 标志

设置 `brand.mode`，然后传递该模式所需的字段。`name` 始终作为标志的标签（`alt` / `aria-label`）。

| 外观         | `mode`         | 必填                   | 展开的界面           | 折叠 / 紧凑 |
| ------------ | -------------- | ---------------------- | ------------------------- | ------------------- |
| 默认      | 省略 `brand`   | —                          | TrueForge 字标        | TrueForge 方形    |
| 图标 + 标题 | `'icon-title'` | `name`（+ 可选 `icon`） | 方形 + 标题文本       | 方形              |
| 仅图标    | `'icon-only'`  | `name`、`icon`             | 方形（无标题文本）    | 方形              |
| 宽标志    | `'logo'`       | `name`、`icon`、`logo`     | 宽标志（无标题文本） | 方形              |

```tsx
<TrueForgeUI
  server={server}
  layout="sidebar"
  theme={{
    brand: {
      mode: 'icon-title',
      name: 'Acme',
      icon: '/brand/icon.svg',
    },
  }}
/>
```

仅图标界面（`name` 保留用于 alt）：

```tsx
theme={{
  brand: {
    mode: 'icon-only',
    name: 'Acme',
    icon: '/brand/icon.svg',
  },
}}
```

展开界面使用宽标志。需要方形 `icon`，因为折叠界面、欢迎屏幕和 widget 按钮继续使用方形资源：

```tsx
<TrueForgeUI
  server={server}
  layout="sidebar"
  theme={{
    brand: {
      mode: 'logo',
      name: 'Acme',
      icon: '/brand/icon.svg',
      logo: '/brand/wordmark.svg',
      href: '/',
    },
  }}
/>
```

`icon` 和 `logo` 都接受 `{ src, light, dark }`。SDK 会选择与解析后的主题模式匹配的源；仅设置一个 light/dark 源则两者都使用该源。`href` 将配置的图片包裹在同标签页链接中。

**组件标志** — `theme.brand` 仅接受图片 URL。要渲染内联 SVG 或自定义组件，请覆盖 `BrandLogo` 插槽，方式与替换任何其他原子组件相同：

```tsx
<TrueForgeUI server={server} layout="sidebar" overrides={{ BrandLogo: MyMark }} />
```

**自定义布局** — 导入 `BrandLogo`，在紧凑表面使用 `variant="icon"`，在展开界面使用 `variant="logo"`。优先使用 `resolveBrandChrome(useBrand())`，使展开与折叠的选择与基础布局保持一致。当界面应显示标题文本时，配合 `useBrandName()` 使用（参见[自定义布局](#自定义布局)）。

> _截图：在基础布局页眉中渲染的外部品牌标志。_

---

## 智能体模式

`agentConfig` 控制库界面、草稿编辑器以及 New Chat / Clear Chat 的行为。

| 模式                                   | 布局界面                   | 智能体选择 / New Chat                                |
| -------------------------------------- | ------------------------------- | --------------------------------------------------------- |
| `AgentLibraryWithComposer` _（默认）_ | 智能体 + 草稿构建器          | New Chat 打开草稿；库选择命名智能体         |
| `SingleAgent`                         | 仅命名，普通编辑器      | 锁定到 `name`；New Chat / Clear Chat = 新线程      |
| `AgentLibrary`                         | 仅智能体（无草稿）    | 选择前为空；无 New Chat；选择后 Clear Chat |
| `AgentComposer`                        | 仅草稿构建器（无库） | 始终为草稿；New Chat / Clear Chat = 新草稿         |

在库模式下，从 Agents 中选择智能体会切换到该智能体的命名聊天，**并重新挂载运行时**，使新智能体从干净的对话开始。草稿聊天可以通过 **Save agent**（解析后的 `AgentUIServer` 上的 `server.saveAgent`）提升。**Clear Chat**（线程页眉）重置当前命名或草稿会话。

可变编辑器暴露 **Agent Config**