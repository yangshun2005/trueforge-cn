# frontend

面向 harness 服务器（registry + 内联会话）的私有 Agent 聊天 UI。**未发布到 npm**（`"private": true`）。

构建于：

```
@truefoundry/trueforge-sdk  (HTTP 客户端；通过宿主 fetch 实现 cookie/OIDC)
  → @truefoundry/trueforge-ui
       server={{ type: 'trueforge', baseUrl, fetch }}
         → plugins/trueforge-agent-server-adapter
              (聊天 + 构建器 + 设置目录)
           → TrueForgeUI (layout="sidebar")
```

[`App.tsx`](src/App.tsx) 仅负责产品外观：

- 认证门禁 / 欢迎 / 登出（`authFetch`、`authSession`）
- 启动：首个模型 → `defaultAgentSpec`、`initialSettingsOpen`
- `<TrueForgeUI server={{ type: 'trueforge', baseUrl: '/', fetch: authAwareFetch }} />`

聊天、Agent 库和设置目录均在 UI SDK 插件内组合——宿主**不**调用 `createTrueFoundryServer`，也不维护 harness 适配器。

## 本地开发

完整的宿主工作流（独立或非独立）请参阅根目录
[`README.md`](../../README.md)。

```bash
pnpm standalone:dev   # 零环境变量：SQLite、Vite :3000 + API :8790
# 或：
pnpm dev:infra        # 然后在另一个终端中：
pnpm dev              # Postgres + Redis、Vite :3000 + API :8790
```

打开 `http://localhost:3000`：Vite 从源码提供 UI（编辑热重载），并将 `/api/*` 代理到 `VITE_SERVER_URL`（默认 `http://localhost:8790`）。
`FRONTEND_PORT` 可将 Vite 移出 `:3000`。Vite 使用 `strictPort`——如果该端口已被占用，开发服务器将退出，而不是选择其他端口。代理配置位于 [`vite.config.ts`](vite.config.ts)。

`predev` / `prebuild` / `pretypecheck` 会先构建 `@truefoundry/trueforge-sdk` 和
`@truefoundry/trueforge-ui`，确保全新检出不依赖过时的 `dist/`。

### 认证

[`authFetch.ts`](src/authFetch.ts) 封装了 `fetch`，并在 HTTP 401 时重定向到 OIDC 登录。
[`authSession.ts`](src/authSession.ts) 使用非重定向客户端探测 `/me`，以便在登录前显示欢迎界面。当 `me().data.type` 为 `oidc-connected` 时，显示登出按钮。将感知认证的 `fetch` 传入
`server={{ type: 'trueforge', fetch }}`，使内置的 Harness 适配器共享相同的会话 cookie。

### 服务器适配器（位于 UI 包中）

Harness ↔ `AgentUIServer` 的映射位于
[`@truefoundry/trueforge-ui/plugins/trueforge-agent-server-adapter`](../trueforge-ui/src/plugins/trueforge-agent-server-adapter/)。
需要在 `<TrueForgeUI />` 之外使用工厂函数的宿主，可从该子路径导入
`createTrueForgeAgentUIServer`。

## 生产环境

没有前端镜像。`pnpm build` 会生成 `dist/` 以及 `.br`/`.gz` 压缩文件
（`vite-plugin-compression2`），服务器从 `FRONTEND_DIR` 提供这些文件，并与 API 共享同一源。
Monaco workers 位于 Vite 资源管道之外，无法预压缩，因此服务器会即时对这些文件进行 gzip 压缩。

```bash
docker compose up --build   # UI + API 位于 http://localhost:8791
```

## 差距

| 项目                                  | 状态                                     |
| ------------------------------------- | ---------------------------------------- |
| 订阅回合 SSE                          | 已推迟（路由已定义，未注册）             |
| 附件 / 沙箱下载                       | 已关闭                                   |
| 会话 `type` / `created_by_subject`    | 软性——此前端不强制要求                  |
| `turn.created.created_by`             | 软性——流式采纳容忍缺失                   |

## 脚本

| 脚本             | 操作                          |
| ---------------- | ----------------------------- |
| `pnpm dev`       | Vite 开发服务器               |
| `pnpm build`     | 类型检查 + 生产构建           |
| `pnpm typecheck` | `tsc --noEmit`                |
| `pnpm test`      | 认证单元测试                  |