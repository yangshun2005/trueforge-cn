# 兼容性

`@truefoundry/trueforge-ui` 的同类包和依赖范围（权威版本请参阅 `package.json`）。

**状态：** 此包为 `0.x` 版本（API 可能会发生变化）。目前它依赖于 `truefoundry-gateway-sdk@^0.4.0-rc.6` —— 在采用稳定的网关发布版本之前，请使用锁文件固定版本以确保可重现的安装。

## 必需的同类包

| 包名                          | 范围           |
| ----------------------------- | -------------- |
| `react` / `react-dom`         | `^18 \|\| ^19` |

在应用根目录安装一次。

## 捆绑依赖（随 SDK 安装）

| 包名                                        | 范围        |
| ------------------------------------------- | ----------- |
| `@assistant-ui/core`                        | `^0.2.22`   |
| `@assistant-ui/react`                       | `^0.14.24`  |
| `@truefoundry/assistant-ui-runtime`         | `0.1.25`    |
| `truefoundry-gateway-sdk`                   | `^0.4.0`    |
| `lucide-react`                              | `^0.562.0`  |

`@assistant-ui/*` 也被列为**可选**同类包，以便直接安装它们进行自定义的主机能够保持在兼容的版本范围内。

运行时和网关由 `TrueFoundryChatProvider` 连接，并重新导出以供高级使用。`lucide-react` 通过 `IconRegistry` 提供默认的 UI 图标；主机可以通过 `theme.icons` 覆盖，或注册自定义的 SVGR 组件。

## 可选直接安装（自定义）

Chrome 钩子（`useAui`、`useAuiState`、`useTheme`）从 `@truefoundry/trueforge-ui` 重新导出 —— 建议优先使用这些，以便主机共享 SDK 实例。

当从主机代码中导入深层的 `@assistant-ui/react` 原语或 `@assistant-ui/core` 时，请将它们作为直接依赖添加，版本需在上述范围内，以便解析以应用根目录为准。强制使用单例（Vite `dedupe`、Yarn `resolutions`、npm `overrides`），使 SDK 和主机共享同一副本 —— 请参阅 README 中的“故障排除”部分。

## 单例

以下内容在依赖图中必须解析为同一物理副本：

- `react` / `react-dom`
- `@assistant-ui/core` / `@assistant-ui/react` / `@assistant-ui/store`