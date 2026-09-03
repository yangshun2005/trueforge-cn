# 计划：移除 `tfy-web-components`，自持 shadcn 原语与主题

硬性移除 `tfy-web-components`。自持原语（shadcn 风格，仓库内）、Markdown/OpenUI/语法高亮器/Monaco、Lucide 图标（可替换映射 + SVG 变换），以及一个包含预设（`trueforge` | `claude` | `chatgpt` | `gemini`）的**主题对象**，并支持完整的 token / className / 图标 / **品牌** / **内容 classNames** 自定义。Shell 还接受由 `Thread`、线程列表等构建的**自定义布局** React 组件。

## 原则：一切皆可自定义

宿主必须能够在不 Fork SDK 的情况下重新设置样式、品牌和调整界面框架。分层结构（每一层均可独立覆盖）：

| 层级                | 自定义内容                                   | 方式                                           |
| ------------------ | -------------------------------------------- | --------------------------------------------- |
| 预设               | 基础外观                                     | `theme.preset`                                |
| Tokens             | 颜色、圆角、字体、气泡颜色                   | `theme.tokens` + CSS 变量                     |
| 品牌               | 方形图标、宽版 Logo 和显示名称               | `theme.brand`（通过插槽标记）                 |
| 图标               | 操作 / UI 图标集                             | `theme.icons`（Lucide + SVG 变换）            |
| 内容 classNames    | Markdown、语法高亮器、OpenUI、Monaco         | `theme.classNames`                            |
| 根类               | 任意宿主工具类                               | `theme.className`                             |
| 宿主 CSS           | 任意 token / 布局微调                        | 文档化的 `:root` / `.aui-root` 覆盖           |
| 插槽               | 替换任意 React 部件                          | `overrides` / `SlotsProvider`                 |
| 布局               | 整个界面框架布局                             | `layout` 字符串 **或** 自定义 React 组件      |

任何视觉或界面框架元素都不应硬编码为 TrueFoundry 品牌。默认值可以带有 TFY 外观；但每个表面都必须接受宿主的覆盖。

## 约束

所有工作都在 **`@truefoundry/trueforge-ui`** 中完成。此轨道不涉及运行时包更改（与 [`docs/server.md`](./server.md) 正交）。

## 已锁定的决策

| 决策            | 选择                                                                                   |
| --------------- | ---------------------------------------------------------------------------------------- |
| 切换方式        | 硬性移除 — 删除依赖 + peer 依赖                                                          |
| 原语            | shadcn 组件 **复制到** 本仓库中                                                          |
| 覆盖范围        | 当前从 tfy 拉取的所有内容                                                                |
| 插槽 API        | **破坏性变更** — 移除 `Button.Primary` / 图标字符串约定；采用新的 shadcn 对齐属性        |
| 图标            | Lucide 默认；宿主可替换映射并提供 SVG 变换                                               |
| 品牌            | 通过 `theme.brand` 提供图标/Logo URL（按模式）；组件标记通过 `BrandLogo` 插槽            |
| 主题 API        | 对象（不仅是字符串）；每个外观方面均可自定义                                             |
| 预设            | 灵感风格包：`trueforge`（默认）、`claude`、`chatgpt`、`gemini`                          |
| 自定义样式      | CSS tokens + `className`；宿主也可以导入 CSS（文档化）。不提供“加载 CSS 文件”属性        |
| 亮色 / 暗色     | 主题对象上的受控 `mode` **或** 省略 → 非受控 `useTheme().setTheme`                      |
| React 覆盖      | 通过 **`overrides` / 插槽** 覆盖结构性部件；通过 **`layout` 组件** 覆盖整个界面框架      |
| Markdown        | 仓库内实现；保留 OpenUI；代码围栏通过语法高亮器处理                                     |
| Monaco           | 仓库内实现（代码产物 / 富代码界面）；classNames 可覆盖                                  |
| 内容样式        | `theme.classNames` 用于 markdown、高亮器、OpenUI、Monaco                                |
| 布局            | 内置预设 **或** 宿主使用导出的构建块编写 `React` 布局                                    |
| 发布            | 破坏性 **0.x** 版本（插槽 API + 样式 + 移除 tfy）                                       |

## 清单 → 替换方案

### 原语（当前为插槽默认值）

| 当前（tfy）         | 目标（仓库内 shadcn 风格）                                                             |
| ------------------ | -------------------------------------------------------------------------- |
| `Button`           | `Button`（变体：default、secondary、ghost、destructive、outline、size）     |
| `IconButton`       | `Button` size=`icon` **或** 轻量 `IconButton` 包装器                        |
| `Modal` / `Dialog` | `Dialog`（如需抽屉则加 sheet）                                              |
| `Accordion*`       | `Accordion`                                                                |
| `LightTooltip`     | `Tooltip`                                                                  |
| `Skeleton`         | `Skeleton`                                                                 |
| `Avatar*`          | `Avatar`                                                                   |
| `Spinner`          | 小型 `Spinner` / 加载器（FA 或内联 SVG）                                    |
| `DropdownMenu`     | `DropdownMenu`                                                             |

### 包装 tfy 分子的功能原子

在仓库内重建（尽可能保留现有原子**名称** / 插槽，更改属性）：

- 气泡：`AssistantMessageBubble`、`UserMessageBubble`、`UserMessageEdit`
- 工具：`ToolCallCard`、`ToolCallContentBlock`、`ToolApprovalBar`、`ToolGroupCard`、`SubAgentCard`、`SandboxToolCallCard`、`AgentStepsCard`、`ReasoningCard`
- 提示：`AskUserPrompt`、`McpAuthPrompt`、`ResumeUnavailable`
- `Markdown`（OpenUI + 高亮器）
- 使用 `IconProvider` → 本地图标解析器的布局辅助工具

### 主题 / 图标 / 品牌基础设施

| 移除                               | 替换为                                                                      |
| -------------------------------- | ----------------------------------------------------------------------- |
| `tfy-web-components/theme.css`   | 自有调色板 + `styles.css` 中的语义 tokens                                 |
| tfy `ThemeProvider` / `useTheme` | 在此处自持 SDK `ThemeProvider`                                            |
| `IconProvider` + `registerIcons` | `IconRegistry` + `theme.icons` / `setIcons`                              |
| 硬编码的 TFY 标记                 | `theme.brand.icon` / `logo`，由头部、欢迎页和组件 FAB 消费                |

### 代码界面（Monaco + 语法高亮器）

当前在 `src/` 中没有任何导入 — 这是一个休眠的 tfy 界面，但确实存在：tfy 自己的 `Markdown` 围栏代码渲染依赖它，因此移除 tfy 会静默丢失此功能，除非显式移植。

| 当前（tfy）                                                                                                           | 目标（仓库内）                                                                                                                             |
| ----------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `molecules/monacoEditorCore/MonacoEditorCore`（轻量 `monaco-editor` 包装器：`onMount`/`beforeMount`/`theme`/`options`） | 本地 `MonacoEditorCore` 原子，相同的属性接口，直接依赖 `monaco-editor`                                                                       |
| `molecules/SimpleCodeEditor`（在 `MonacoEditorCore` 之上提供复制/下载/展开/全屏/行号）                                   | 本地 `CodeEditor` 原子（基于本地 `MonacoEditorCore` 重建，功能集相同）                                                                       |
| `molecules/CopyField` / `CopyFieldHighlighted`（`SimpleCodeEditor` 使用的复制功能）                                      | 合并到本地 `CodeEditor` 的复制按钮中；无需单独的原语                                                                                         |
| tfy `Markdown`/`MarkdownWithOpenUI` 内部用于非 OpenUI 围栏代码的 `react-syntax-highlighter`                              | 本地 `SyntaxHighlighter` 原子，由仓库内 `Markdown` 原子为非 openui 围栏调用（相同的行号 / 亮暗主题映射）                                      |

由此产生的新直接依赖：`monaco-editor`、`react-syntax-highlighter`（+ `@types/react-syntax-highlighter`）。

## 主题 API（目标）

```ts
type ThemeMode = "light" | "dark" | "system";

type ThemePreset = "trueforge" | "claude" | "chatgpt" | "gemini";

type SemanticTokens = {
  // 跨产品
  sidebarBg: string;
  topbarBg: string;
  primaryBg: string;
  secondaryBg: string;
  border: string;
  fontFamily: string; // 映射到 --font-agent-ui
  // 构建块
  inputBoxBg: string;
  inputBorder: string;
  textPrimary: string;
  textSecondary: string;
  cardBg: string;
  dropdownSelectedItemBg: string;
  dropdownSelectedItemText: string;
  // 聊天
  userMessageBg: string;
  userMessageText: string;
  assistantMessageBg: string;
  assistantMessageText: string;
  // 按钮
  primaryButtonBg: string;
  primaryButtonHover: string;
  primaryButtonText: string;
  secondaryButtonBg: string;
  secondaryButtonHover: string;
  secondaryButtonText: string;
  ghostButtonBg: string;
  ghostButtonHover: string;
  ghostButtonText: string;
  // 状态
  successBg: string;
  successText: string;
  failureBg: string;
  failureText: string;
  warningBg: string;
  warningText: string;
  // 保留的内部项
  focusRing: string;
  radius: string;
  composerRadius: string;
  overlay: string;
  shadowColor: string;
  scrollbarThumb: string;
};

/** UI 操作图标（发送、附件、复制等）— Lucide、React 节点或 SVG 包装器 */
type IconMap = Record<
  string,
  | LucideIcon
  | React.ReactNode
  | ((props: IconProps) => React.ReactNode)
  | React.FC<React.SVGProps<SVGSVGElement>>
>;

/**
 * 品牌图片来源。`light` / `dark` 根据解析后的主题模式选择，并相互回退，然后回退到 `src`。
 */
type BrandLogoConfig = {
  src?: string;
  light?: string;
  dark?: string;
};

type BrandImage = string | BrandLogoConfig;

/**
 * 产品品牌 — 与 `icons`（UI 界面）不同。
 * 在出现产品标记的任何地方消费（头部、欢迎页、组件 FAB、空状态、助手头像回退）。
 *
 * 仅限图片：要渲染组件，请改为覆盖 `BrandLogo` 插槽，这样品牌标记与其他原子走相同的替换路径。
 *
 * 设置 `mode`，然后传入该模式所需的字段：
 * - 省略 `brand`：TrueForge 文字商标 / 方形标记
 * - `icon-title`：`name` + 可选 `icon`
 * - `icon-only`：`name` + `icon`
 * - `logo`：`name` + `icon` + `logo`
 */
type BrandMode = "icon-title" | "icon-only" | "logo";

type BrandConfig =
  | { mode: "icon-title"; name: string; icon?: BrandImage; logo?: never; href?: string }
  | { mode: "icon-only"; name: string; icon: BrandImage; logo?: never; href?: string }
  | { mode: "logo"; name: string; icon: BrandImage; logo: BrandImage; href?: string };

type ThemeConfig = {
  preset?: ThemePreset; // 默认值："trueforge"
  mode?: ThemeMode; // 省略 = 非受控（useTheme().setTheme）
  tokens?: Partial<SemanticTokens>;
  brand?: BrandConfig; // 设置 brand.mode，然后传入必填字段
  className?: string; // 应用于 .aui-root（或主题根元素）
  icons?: IconMap; // 完整/部分 UI 图标替换 + SVG 变换
  /** 内容渲染器的逐界面 className 钩子 */
  classNames?: ContentClassNames;
};

/**
 * 覆盖内容引擎上的 CSS 类（以及可选的样式属性）。
 * 所有字段均为可选；通过 `cn()` 与默认值合并。
 */
type ContentClassNames = {
  /** 渲染后 markdown 的包装器 / prose 根元素 */
  markdown?: string;
  inlineCode?: string; // Markdown 渲染的行内代码
  /** 非 openui 围栏代码块（语法高亮器根元素 / pre / code） */
  syntaxHighlighter?: {
    root?: string;
    pre?: string;
    code?: string;
    lineNumber?: string;
  };
  /** OpenUI 围栏块宿主 + 公共子钩子 */
  openui?: {
    root?: string;
    scope?: string;
  };
  /** Monaco 编辑器 / diff 界面 */
  monaco?: {
    root?: string;
    editor?: string;
    /** 可选的 Monaco `theme` id 或 defineTheme 钩子名称 — 样式相邻 */
    monacoTheme?: string;
  };
};

type BuiltInLayout = "sidebar" | "drawer" | "dock" | "widget";

/**
 * Shell 布局：内置字符串 **或** 宿主组件，组合导出的构建块（`Thread`、`ThreadListContainer` 等）。
 */
type LayoutProp = BuiltInLayout | React.ComponentType<{ className?: string }>;

// 用法 — 主题 + 自定义布局
function MyLayout({ className }: { className?: string }) {
  return (
    <div className={cn("flex h-full", className)}>
      <aside className="w-64 border-r">
        <ThreadListContainer />
      </aside>
      <main className="flex min-w-0 flex-1 flex-col">
        <Thread />
      </main>
    </div>
  );
}

<TrueForgeUI
  layout={MyLayout} // 或 'sidebar' | 'drawer' | 'dock' | 'widget'
  theme={{
    preset: 'claude',
    mode: 'dark',
    tokens: { primaryButtonBg: '#…', fontFamily: '"My Font", system-ui' },
    brand: {
      mode: 'logo',
      name: 'Acme Agent',
      icon: { light: '/acme-icon.svg', dark: '/acme-icon-dark.svg' },
      logo: { light: '/acme-wordmark.svg', dark: '/acme-wordmark-dark.svg' },
    },
    className: 'my-chat',
    icons: { send: MySendSvg, paperclip: MyAttachSvg },
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

**解析顺序：** 预设默认值 → `tokens` → `brand` / `icons` → `classNames` → 宿主 CSS 变量（文档化）→ `className` 工具类。

组件解析是独立的：最近的 `SlotsProvider` 覆盖优先，然后是父级覆盖，最后是 `defaultSlots`。覆盖会收到该原子的确切 props 对象；不要假设容器组合的内容会作为 `children` 传入。

**不使用对象的自定义样式：** 宿主 CSS 仍然有效：

```css
.aui-theme-root {
  --primary-button-bg: #e11d48;
  --font-agent-ui: 'My Font', system-ui, sans-serif;
}

/* 或针对文档化的内容钩子 */
.aui-root .aui-markdown {
  /* … */
}
.aui-root .aui-syntax-highlighter {
  /* … */
}
.aui-root .aui-openui {
  /* … */
}
.aui-root .aui-monaco {
  /* … */
}
```

## 品牌系统

1. 所有产品标记都通过 `<BrandLogo />` 渲染 — 一个组件，绝不在布局或原子中硬编码 TFY 资源。布局通过 `useSlot` 解析它，因此宿主的覆盖可以到达每个调用点。
2. 宿主通过 `brand.mode`（`icon-title` | `icon-only` | `logo`）选择界面样式。布局界面使用 `resolveBrandChrome(brand)` 来确定展开/折叠的标记变体以及是否显示文本标题。自定义布局应调用相同的辅助函数，而不是重新推导字段。
3. `BrandLogo` 在紧凑界面中渲染 `icon`。当 `variant="logo"` 时，它渲染可选的更宽 Logo，并在没有时回退到方形图标。
4. `theme.brand` 仅携带**图片来源**（URL 或 `{ src, light, dark }`）。组件形式的标记通过 `BrandLogo` 插槽处理，因此品牌替换与所有其他原子使用相同的机制，而不是在主题配置中提供第二个节点形式的逃生舱。
5. `theme.brand.name` 在每种模式下都是必填的，并且始终为配置的图片提供标签。可见的标题文本仅在 `mode: 'icon-title'` 时出现。对于 `icon-only` 和 `logo`，`name` 仅作为 alt 文本。
6. 省略 `brand` 时，展开的界面保留默认的 TrueForge 文字商标，折叠时保留方形标记。`href` 将配置的图片包装为同标签页链接。
7. `{ light, dark }` 图片来源根据提供者的主题模式解析，因此宿主标记无需自定义组件即可跟随亮/暗模式。单个配置的来源