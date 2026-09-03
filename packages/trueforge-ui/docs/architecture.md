# 架构

本包将 UI 拆分为 **atoms**（展示层）和 **containers**（运行时接线），并通过 **精选插槽注册表** 实现覆盖。

## Atoms（`src/atoms/**`）

展示型组件。一个 atom：

- **不**导入 `@assistant-ui/*` 或 TrueFoundry 运行时钩子。
- 通过 props 接收数据和回调。
- 拥有其渲染内容的 Tailwind 类 / 标记。
- 精选默认值位于 `defaultSlots` 中；其键定义了公开的 `SlotOverrides` 契约。
- 每个 atom 的 `AtomSlots` 增强在内部验证 `defaultSlots` 是否包含所有已注册的 atom。
- 底层原语和专用 chrome API 保持直接导入。

默认值在 [`src/theme/defaultSlots.ts`](../src/theme/defaultSlots.ts) 中注册。

## Containers（`src/containers/**`）

有状态胶水层。一个 container：

- 读取 assistant-ui / TrueFoundry 运行时钩子和原语。
- 派生纯 props 和回调。
- 使用 `useSlot("Name")` 解析公开的视觉 atoms；底层原语、提供者以及专用品牌/布局 API 则直接导入。
- 避免超出布局胶水之外的装饰性样式。

## 插槽（`SlotsProvider`）

```tsx
function MyWelcome({ heading = 'Welcome' }: WelcomeScreenProps) {
  return <h1>{heading}</h1>;
}

<SlotsProvider overrides={{ WelcomeScreen: MyWelcome }}>
  <Thread />
</SlotsProvider>;
```

嵌套的提供者会回退到父级覆盖项，然后是 `defaultSlots`。快速开始：`TrueForgeUI` 在聊天提供者外部挂载 `SlotsProvider`。有关保留列表，请参阅 [customization.md](./customization.md)。

## 主题

默认 atoms 使用自动注入样式表中的 CSS 变量（作用域为 `.aui-theme-root`）。可通过 `theme.tokens` 或 `.aui-theme-root` / `.aui-theme-root.dark` 上的宿主 CSS 覆盖颜色。

## 通知

`ToasterProvider` 显示成功和错误提示。来自网关错误的描述会被截断以用于显示。