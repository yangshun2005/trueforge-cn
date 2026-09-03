# 为 TrueForge 做贡献

感谢您有兴趣参与贡献！本文档涵盖如何搭建开发环境、运行测试套件以及让 Pull Request 被合并。

参与即表示您同意遵守我们的[行为准则](CODE_OF_CONDUCT.md)。贡献即表示您同意您的贡献根据 [MIT 许可证](LICENSE) 授权。

## 贡献方式

> [!重要]
> 请仅在获得维护者批准后，才针对某个 Issue 提交 Pull Request。
> 虽然我们重视良好的意图，但我们对社区代码贡献要求先获得批准，以确保我们有效利用每个人的时间。

- **报告 Bug** - 提交一个 Issue，包含复现步骤、预期与实际行为以及您的环境信息。
- **改进文档** - 我们始终欢迎拼写错误修正和内容澄清。
- **修复 Bug / 构建功能** - 对于任何非琐碎的改动，请先提交一个 Issue，以便在您投入时间之前，我们可以讨论实现方法。

对于安全漏洞，**请勿**提交公开 Issue - 请参阅 [SECURITY.md](SECURITY.md)。

### 为什么我们对社区代码贡献要求先获得批准

> [!提示]
> 每个新 Issue 默认带有 [needs-maintainer-attention](https://github.com/truefoundry/trueforge/issues?q=is%3Aissue%20state%3Aopen%20label%3Aneeds-maintainer-attention) 标签。维护者将审查该 Issue 并讨论后续步骤。创建 Issue 时请不要移除此标签。

对 TrueForge 的有效更改需要架构上下文、对系统级约束的理解以及对项目路线图的了解。社区 Pull Request 通常关注优先级较低、影响用户数量较少或需要进行大量更改才能适配更广泛系统的问题。审查和迭代这些更改可能比直接实施修复花费更多时间，从而分散了对更高优先级工作的注意力。

当社区通过详细的 Bug 报告、复现步骤、日志、根本原因分析和 Issue 中的设计讨论来分享专业知识时，其价值最高。理解问题、确定正确的解决方案以及确定工作优先级通常是困难的部分；而借助编码辅助工具，实施相对简单。

基于这些原因，我们将社区贡献的重点放在 Issue 报告、分析和反馈上，而非大型代码更改。

> [!注意]
> 如果您仍然希望贡献代码，我们维护了一组范围明确且标记为 [help-wanted](https://github.com/truefoundry/trueforge/issues?q=is%3Aissue%20state%3Aopen%20label%3A%22help%20wanted%22) 的 Issue。

### Issue 标签

| 标签                                                                                                                                          | 含义                                                                  |
| ---------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| [help-wanted](https://github.com/truefoundry/trueforge/issues?q=is%3Aissue%20state%3Aopen%20label%3A%22help%20wanted%22)                       | 范围明确的 Issue，欢迎社区贡献。             |
| [needs-maintainer-attention](https://github.com/truefoundry/trueforge/issues?q=is%3Aissue%20state%3Aopen%20label%3Aneeds-maintainer-attention) | 在开始实施之前，需要与维护者讨论的 Issue。 |

### 报告 Bug

在提交新 Issue 之前，请搜索 Issue 跟踪器，查看该问题是否已被报告。如果已报告，请将任何新信息添加到现有 Issue 中。

报告 Bug 时，请尽可能包含相关的详细信息：

- 清晰、详细的复现问题的步骤。
- 预期行为和实际行为。
- 您的 TrueForge 版本、操作系统以及其他相关的环境详细信息。
- 日志、错误消息或其他诊断信息，并移除敏感信息。
- 根本原因分析、技术观察或潜在的修复方法（如有）。

### 请求功能

在 Issue 跟踪器中提交功能请求，或为描述相同需求的现有请求点赞。请说明您的用例、您期望的行为以及它将如何改善您的工作流程。

## 前提条件

- **Node.js 22.14+** （参见 [`.nvmrc`](.nvmrc)；pnpm 11.16 需要 22.13+，`better-sqlite3` v13 需要 Node-API 10）
- **pnpm** （版本通过 [`package.json`](package.json) 中的 `packageManager` 固定；`corepack enable` 可处理）
- **Docker** - 仅用于 Postgres/Redis 开发基础设施、冒烟测试和本地 SDK 生成（维护者）。Fork 贡献者无需生成 SDK。

## 仓库结构

这是一个 pnpm workspace：

| Workspace 包                     | 发布名称                        | 路径                                                 | 说明                                         |
| ----------------------------- | ----------------------------- | ---------------------------------------------------- | -------------------------------------------- |
| `@truefoundry/trueforge`      | `@truefoundry/trueforge`      | [`packages/trueforge`](packages/trueforge)           | Agent 服务器 + 捆绑的 UI                    |
| `@truefoundry/trueforge-core` | `@truefoundry/trueforge-core` | [`packages/trueforge-core`](packages/trueforge-core) | 库：Agent 核心、会话和流式传输 |
| `@truefoundry/trueforge-ui`   | `@truefoundry/trueforge-ui`   | [`packages/trueforge-ui`](packages/trueforge-ui)     | 可嵌入的 Agent 聊天 UI SDK                 |
| `@truefoundry/trueforge-sdk`  | `@truefoundry/trueforge-sdk`  | [`packages/trueforge-sdk`](packages/trueforge-sdk)   | 生成的 TypeScript API 客户端              |
| `frontend`                    | -                             | [`packages/frontend`](packages/frontend)             | 聊天 UI 应用（捆绑到服务器中）        |

## 设置

```bash
pnpm install
cp packages/trueforge/.env.example packages/trueforge/.env
```

## 从源码运行

有两种开发拓扑结构。两者都在 [http://localhost:3000](http://localhost:3000) 上提供 UI 服务（Vite，支持实时重载，代理 `/api/*`），并在 [http://localhost:8790](http://localhost:8790) 上提供 API 服务。

### 独立模式（零基础设施）

仅使用 SQLite - 适用于大多数日常开发。独立模式旨在本地使用；它不是生产安全的拓扑结构（默认无登录，本地 SQLite）。请不要将其暴露在 localhost 之外。

```bash
pnpm standalone:dev
```

### 多副本模式（Postgres + Redis）

在处理涉及 Postgres 存储或 Redis 对等通信（跨副本轮次取消和流交接）的任何工作时，请使用此模式。

终端 1 - 启动 Postgres（`:5432`）和 Redis（`:6379`）：

```bash
pnpm dev:infra
```

终端 2：

```bash
pnpm dev
```

|           | 独立模式                             | 多副本模式                                   |
| --------- | -------------------------------------- | ----------------------------------------------- |
| 进程   | 单个服务器                             | 一个或多个具有 Redis 对等通信的副本         |
| 数据库  | SQLite                                 | Postgres                                        |
| 基础设施     | 无                                   | Postgres + Redis（`pnpm dev:infra` 或您自己的） |
| 开发       | `pnpm standalone:dev`                  | `pnpm dev`（基础设施启动后）                        |
| 类生产 | `pnpm build` → `pnpm standalone:start` | `pnpm build` → `pnpm start`                     |

本地服务器脚本从源码解析 `@truefoundry/trueforge-core`，因此您不需要为 `pnpm dev` / `standalone:dev` 构建 trueforge-core 的 `dist/`。前端脚本在 Vite 启动前构建 `@truefoundry/trueforge-ui` 的 `dist/`（workspace 包导出指向 `dist/`）。

### UI 的提供方式

开发模式和类生产模式使用不同的进程拓扑结构。独立模式与多副本模式仅更改存储/对等通信（`STANDALONE`）；它不会改变 UI 的提供方式。

**开发模式** - `pnpm standalone:dev` / `pnpm dev` 并行运行两个进程：Vite（UI，支持实时重载）和 Hono API。浏览器与 Vite 通信；Vite 将 `/api/*` 代理到 Hono。

```
  browser                     Vite (:3000)                   Hono (:8790)
     │                             │                              │
     │  GET /  (UI)                │                              │
     │────────────────────────────>│                              │
     │                             │                              │
     │  /api/*                     │  proxy                       │
     │────────────────────────────>│─────────────────────────────>│
```

**类生产模式** - 在 `pnpm build` 之后，`pnpm standalone:start` / `pnpm start` 运行单个 Hono 进程。它为非 API 路由提供构建好的前端捆绑包，并自行处理 `/api/*`。Docker 容器的工作方式相同：一个进程，一个来源。

```
  browser                                    Hono (:8790)
     │                                            │
     │  GET /  (来自 dist/_frontend 的静态 UI)    │
     │───────────────────────────────────────────>│
     │                                            │
     │  /api/*                                    │
     │───────────────────────────────────────────>│
```

在单一来源上（start / Docker）：`/api/*`（包括 OpenAPI）和 `/healthz` 是 API；其他所有内容都是 UI。服务器优先使用打包的 `dist/_frontend`，然后是 `packages/frontend/dist`。如果没有 UI 构建产物，则仅提供 API（对于由 Vite 支持的 `pnpm dev` 来说是正常的）。

### 配置

有关每个环境变量，请参阅 [`packages/trueforge/.env.example`](packages/trueforge/.env.example)。有用的开发覆盖项：

- `PORT` - API 端口（默认 `8790`）
- `FRONTEND_PORT` - 开发模式下 Vite UI 端口（默认 `3000`）；参见 [`packages/frontend/README.md`](packages/frontend/README.md)
- `VITE_SERVER_URL` - 将 Vite 代理指向不同的 API
- `PUBLIC_BASE_URL` - MCP OAuth / OIDC 回调的公共来源。`pnpm standalone:dev` / `pnpm dev` 以及分布式模式（例如 Vite 的 `http://localhost:3000`）需要此设置。非开发独立模式回退到 `http://localhost:$PORT`。
- `FRONTEND_DIR` - 服务器要提供的已构建 UI 的目录
- `SQLITE_PATH` - 独立模式下 SQLite 文件的位置
- `REDIS_URL` / `POSTGRES_*` - 当 `STANDALONE=false` 时使用

### 数据库迁移

数据库迁移在服务器启动时运行。要在不启动 HTTP 的情况下运行 Postgres 迁移：

```bash
pnpm --filter @truefoundry/trueforge migrate
```

该脚本设置 `STANDALONE=false` 并使用来自 `packages/trueforge/.env` 的 `POSTGRES_*` 变量。它不会在独立模式下运行（SQLite 迁移改为在启动时进行）。

## Workspace 脚本

Workspace 任务通过 `package.json` 脚本进行 - 如果缺少可重复的工作流程，请添加脚本，而不是记录临时命令。

| 脚本                                               | 用途                                               |
| ---------------------------------------------------- | ----------------------------------------------------- |
| `pnpm standalone:dev` / `pnpm dev`                   | 本地 UI + API（参见上述模式）                      |
| `pnpm standalone:dev:no-watch` / `pnpm dev:no-watch` | 相同，但服务器不会在文件更改时重启 |
| `pnpm dev:server:ui`                                 | 打包的 UI + 监听的 API（无 Vite）                     |
| `pnpm build`                                         | 构建所有包                                    |
| `pnpm test` / `pnpm typecheck`                       | Workspace 检查                                      |
| `pnpm lint` / `pnpm format`                          | ESLint（带修复）/ Prettier                        |
| `pnpm smoke` / `pnpm smoke:down`                     | 完整的 Docker Compose 栈 + 健康检查              |
| `pnpm chart:lint` / `pnpm chart:template`            | 验证 Helm chart                               |
| `pnpm clean` / `pnpm clean:all`                      | 移除构建输出（`:all` 还包括 `node_modules`）    |

## 测试

```bash
pnpm test         # 所有包的单元测试
pnpm typecheck    # 所有包的 TypeScript 检查
pnpm lint         # ESLint
pnpm format:check # Prettier
```

### 存储契约测试

存储层更改应通过两个存储套件（当存储路径更改时 CI 会运行它们）：

```bash
pnpm --dir packages/trueforge test:store:sqlite
pnpm test:store:local   # 针对本地基础设施的 Postgres 套件
```

### 冒烟测试（Docker）

容器中的完整技术栈 - 构建的服务器镜像提供 API + UI，以多副本模式（`STANDALONE=false`）运行：

```bash
pnpm smoke       # 构建，等待服务健康，检查 /healthz 和 UI
pnpm smoke:down
```

打开 [http://localhost:8791](http://localhost:8791)。凭据来自 `packages/trueforge/.env`。主机端口与本地开发端口偏移，以避免冲突：Postgres `:5433`，Redis `:6380`，应用 `:8791`。

## 生成的代码 - 请勿手动编辑

- [`packages/trueforge-sdk`](packages/trueforge-sdk)、[`.github/fern/openapi/openapi.json`](.github/fern/openapi/openapi.json) 和 [`docs/openapi.json`](docs/openapi.json) 是生成并提交的。请改为编辑 `packages/trueforge/src/routes/` 下的路由处理器；CI 会重新生成输出。要在本地重新生成（需要 Docker）：`pnpm sdk:generate`。
- `packages/trueforge/catalog/` 下的 Catalog YAML 文件由 `build:gen` 脚本在构建时内联。

## 服务器入口点

| 文件           | 使用者                                       | 角色                                        |
| -------------- | --------------------------------------------- | ------------------------------------------- |
| `dist/main.js` | Docker、`pnpm start`、`pnpm standalone:start` | 仅环境变量的服务器启动                        |
| `dist/cli.js`  | `npx @truefoundry/trueforge`                  | CLI（`--help`、`--port`），然后加载 `main` |

## 编码约定

仓库级规则位于 [`AGENTS.md`](AGENTS.md) 中（某些包有自己的嵌套 `AGENTS.md`）。要点：

- 不允许使用类型断言逃逸（`as T`、非空 `!`）来消除错误 - 请修复类型。
- 包装并重新抛出必须通过 `{ cause }` 保留原始错误。
- 基于 Zod 的类型使用 `z.infer<typeof Schema>` 派生；不要将 Schema 重复定义为接口。
- 仅使用静态 `import` / `import type` - 不使用 `require()`。
- 测试位于顶层 `test`/`tests` 目录下，镜像 `src` 结构，绝不内联。
- 网络结构（HTTP/OpenAPI）和数据库标识符使用 `snake_case`。
- 服务器环境变量读取通过 `packages/trueforge/src/config.ts`，绝不直接使用 `process.env`。
- 在使代码变为死代码的同一更改中移除死代码。

Prettier 和 ESLint 通过 husky + lint-staged 作为 pre-commit 钩子运行。

## Pull Requests

1. Fork（或创建分支）并创建主题分支：`git checkout -b my-fix`。
2. 进行您的更改，并在合理的地方添加测试。不要提交生成的 OpenAPI/SDK 输出（Fork：仅源码；`main` 分支在合并后重新生成）。
3. 确保 `pnpm build`、`pnpm test`、`pnpm typecheck`、`pnpm lint:ci` 和 `pnpm format:check` 通过 - CI 恰好运行这些检查。
4. 保持 PR 聚焦且规模合理；在描述中说明**更改了什么**以及**为什么**。
5. 提交 PR。维护者将进行审查；squash 合并是常态。
6. **一旦维护者开始审查，请避免变基或强制推送。** 这可能会使迭代审查更加困难。要更新您的分支，请改为合并 `main`。我们最终会进行 squash 合并，因此 PR 内混乱的历史是可以接受的。

## 发布版本

维护者：有关 npm、容器镜像和 Helm chart 的发布流程，请参阅 [RELEASING.md](RELEASING.md)。