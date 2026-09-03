# 发布

本仓库发布 npm 包、生产容器镜像、Helm Chart、沙箱镜像，以及可选的从源码构建的 **开发** 镜像。

| 内容                                | 触发方式                                                                           | 工作流                                                                                       |
| ----------------------------------- | ---------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| npm 包                        | 推送到 `main` 分支 (Changesets)                                                        | [`release.yml`](.github/workflows/release.yml)                                                 |
| 生产镜像 + Chart 发布 PR       | 在 `@truefoundry/trueforge` npm 发布后（可复用工作流），或手动触发 | [`build-and-prepare-chart-release.yml`](.github/workflows/build-and-prepare-chart-release.yml) |
| Chart 标签、GitHub Release、OCI 推送 | 合并 `release-chart/trueforge` 分支，或推送/触发 `charts/trueforge@*`       | [`release-chart.yml`](.github/workflows/release-chart.yml)                                     |
| 沙箱镜像 + 固定版本 PR              | 当 `scripts/sandbox/**` 变更时推送到 `main`，或手动触发                      | [`push-sandbox-image.yml`](.github/workflows/push-sandbox-image.yml)                           |
| 开发（从源码构建）镜像             | 手动 `workflow_dispatch`                                                         | [`build-dev-image.yml`](.github/workflows/build-dev-image.yml)                                 |

## 版本管理

| 构件                     | 标识                                                                                                  |
| ---------------------------- | --------------------------------------------------------------------------------------------------------- |
| npm `@truefoundry/trueforge` | SemVer `X.Y.Z` — 应用代码的版本来源                                                             |
| Chart `appVersion`           | 一个**已发布**的 npm 版本                                                                               |
| 生产镜像                   | 根目录 [`Dockerfile`](Dockerfile)：`npm install @truefoundry/trueforge@$APP_VERSION`                        |
| 生产镜像标签               | `{appVersion}-{shortSha}`（构建提交的短 SHA）                                                  |
| Chart `version`              | 独立的 SemVer；git 标签 `charts/trueforge@A.B.C` 必须与之匹配                                           |
| 沙箱镜像                | [`sandbox.Dockerfile`](packages/trueforge-core/scripts/sandbox/sandbox.Dockerfile)；标签 = 完整提交 SHA |
| 开发镜像                    | [`Dockerfile.dev`](Dockerfile.dev)；标签 = 完整提交 SHA                                                 |

安装已发布的 Chart：

```bash
helm install trueforge oci://tfy.jfrog.io/tfy-helm/trueforge --version <chart-semver>
```

---

# npm 包

| 包                       | 源码                    | 说明                                         |
| ----------------------------- | ------------------------- | --------------------------------------------- |
| `@truefoundry/trueforge-core` | `packages/trueforge-core` | 库                                       |
| `@truefoundry/trueforge`      | `packages/trueforge`      | 应用 + CLI；tarball 包含 `dist/_frontend/` |
| `@truefoundry/trueforge-sdk`  | `packages/trueforge-sdk`  | Fern 生成 — 请勿手动编辑             |
| `@truefoundry/trueforge-ui`   | `packages/trueforge-ui`   | 可嵌入的聊天 UI                            |

`packages/frontend` 不发布；其构建产物会复制到 `@truefoundry/trueforge` 的 tarball 中。
发布时 `workspace:*` 会被重写为精确版本。

## 流程

无 `v*` 标签发布。[`release.yml`](.github/workflows/release.yml) 同时处理版本更新和发布
（`select-mode` → `version` \| `pack` → `publish`）：

1. 在与代码变更相同的 PR 中添加 changeset（`pnpm changeset`，或
   `pnpm change --bump patch --summary "…" <pkg>`）。SDK 重新生成已通过 `pnpm changeset:sdk-regen` 添加 `@truefoundry/trueforge-sdk`。
2. 合并到 `main`。待处理的 changesets → **Version Packages** PR
   （`pnpm run version`）。审查并合并。
3. 无待处理 changesets 时，**打包**（构建/测试）然后通过 npm
   可信发布（OIDC；无需 `NPM_TOKEN`）**发布**。
4. 如果 `@truefoundry/trueforge` 已发布，**Release** 会在同一提交上调用 **Build and
   prepare chart release** 作为可复用工作流（因此较新的 `main` 推送不会改变 Dockerfile / shortSha）。GitHub 的
   `workflow_dispatch` API 仅接受分支或标签名，不接受 SHA。
5. 在早期 `0.x` 阶段将依赖项固定到精确版本。

在 **Release** 上执行 `workflow_dispatch` 会重新运行同一工作流。

## 预发布模式

通过 `.changeset/pre.json` 在仓库范围内设置（不存在 = 发布到 `latest`）：

| 目标     | 命令                       | 后续操作                                                                      |
| -------- | ----------------------------- | ------------------------------------------------------------------------- |
| 进入 RC | `pnpm changeset pre enter rc` | 提交 `pre.json`，合并；版本显示为 `x.y.z-rc.N`（`rc` dist-tag） |
| 退出 RC  | `pnpm changeset pre exit`     | 提交删除，合并；下一次 Version Packages 合并发布 `latest`  |

`pre enter` / `pre exit` 不会进行版本升级或发布。不要在一个 `changeset version` 中混合稳定版和 RC 包。

## 可信发布

每个公共包必须在 npmjs.com 上将此仓库 + 工作流列为可信发布者：

- 仓库：`truefoundry/trueforge`
- 工作流：`release.yml`（精确文件名）
- 无 GitHub 环境名称

不要在发布作业上设置 `NPM_TOKEN` / `_authToken` — 这会禁用 OIDC。
只有 **publish** 作业使用 npm OIDC（`id-token: write`）。

发布会附加 npm 来源证明（发布作业上的 `NPM_CONFIG_PROVENANCE`，以及
每个公共包上的 `publishConfig.provenance: true`）。这会在 npmjs.com 上公开证明
源仓库和提交。

## 本地发布（不推送）

```bash
pnpm clean && pnpm build && pnpm standalone:start
# 或：在 packages/trueforge-core / packages/trueforge 内执行 pnpm pack
```

## 故障排查

- **无 Version Packages PR** — `main` 上没有 `.changeset/*.md`。添加一个，或重新运行 **Release**。
- **发布需要标签** — RC 需要 `rc` dist-tag（在 `pre.json` 存在时自动设置）。
- **403** — 版本已存在于 npm，或可信发布者配置不匹配。
- **OIDC 失败** — pnpm >= 11.0.7；移除 registry `_authToken`。
- **缺少 `dist/_frontend/index.html`** — 根目录 `pnpm build` 必须先构建 `frontend`。
- **Version PR 上 SDK 未重新生成** — 仅当 `@truefoundry/trueforge-sdk` 版本变动时
  （`scripts/version.mjs`；需要 Docker）。
- **npm 发布后缺少生产镜像** — 在**分支或标签**（非 SHA）上触发 chart 工作流：
  `gh workflow run build-and-prepare-chart-release.yml --ref main -f app_version=X.Y.Z -f update_app_version=true`。

---

# 镜像和 Helm Chart

```text
npm publish @truefoundry/trueforge@X.Y.Z
  → 调用 build-and-prepare-chart-release（与发布同一提交）
  → 构建 Dockerfile（APP_VERSION=X.Y.Z）→ 推送 X.Y.Z-<shortSha>
  → 在 release-chart/trueforge 分支上打开/更新 PR
  → 合并 PR → 打标签 + GH Release + OCI 推送（release-chart.yml）

手动重建（相同或其他应用版本）
  → workflow_dispatch build-and-prepare-chart-release
  → 相同的 PR 路径

仅 Chart
  → 人工 PR 更新 Chart.yaml version
  → 人工打标签 charts/trueforge@A.B.C（或 gh release create）
  → release-chart.yml 发布 OCI（不重建镜像）
```

## Dockerfile

| 文件                               | 角色                                                                                    |
| ---------------------------------- | --------------------------------------------------------------------------------------- |
| [`Dockerfile`](Dockerfile)         | 生产/OSS。`ARG APP_VERSION` → `npm install @truefoundry/trueforge@$APP_VERSION`         |
| [`Dockerfile.dev`](Dockerfile.dev) | 从源码构建。用于 [`docker-compose.yml`](docker-compose.yml) 和 **Build dev image** |

如果该 npm 版本不存在，生产构建会失败（无 workspace 回退），因此即使 `main` 已前进，
`appVersion` 仍保持真实。

## 构建并准备 Chart 发布

[`build-and-prepare-chart-release.yml`](.github/workflows/build-and-prepare-chart-release.yml)
（从 **Release** 通过 `workflow_call` 调用，或手动 `workflow_dispatch`）：

| 输入                | 默认值                   | 含义                                                              |
| -------------------- | ------------------------- | -------------------------------------------------------------------- |
| `app_version`        | `Chart.yaml` `appVersion` | 要安装到镜像中的 npm 版本                                |
| `update_app_version` | `false`                   | 同时将该版本写入机器人 PR 上的 `Chart.yaml` `appVersion` |

始终：构建/推送 `{appVersion}-{shortSha}`，补丁升级 chart `version`，设置 `image.tag`，
在 `release-chart/trueforge` 上打开/更新一个 PR。

```bash
gh workflow run build-and-prepare-chart-release.yml
gh workflow run build-and-prepare-chart-release.yml -f app_version=0.1.0
# 在新应用版本 npm 发布后：
gh workflow run build-and-prepare-chart-release.yml \
  -f app_version=0.1.0 -f update_app_version=true
```

你可以在合并前在 PR 上编辑 chart SemVer（minor/major）；标签在合并时跟随
`Chart.yaml` `version`。每次运行都会从 `main` 重建 `release-chart/trueforge` 分支，
但分支上高于补丁升级的 chart `version` 会被保留，因此手动升级在后续镜像重建后仍然有效。
该分支上的其他手动编辑不会保留 — 请将其提交到 `main`。

## 发布 Helm Chart

[`release-chart.yml`](.github/workflows/release-chart.yml) 是一个包含三个入口点的作业：

| 触发方式                                                   | 功能                                                        |
| --------------------------------------------------------- | ------------------------------------------------------------------- |
| 从 `release-chart/trueforge` 合并的 PR（仅限同一仓库） | 创建 `charts/trueforge@<version>` + GitHub Release，然后 OCI 推送 |
| 推送标签 `charts/trueforge@*`                          | 仅 OCI 推送（标签已存在）                                  |
| 带 `tag=` 的 `workflow_dispatch`                           | 为现有标签执行 OCI 推送（重试）                                |

只有 `release-chart/trueforge` 分支会自动打标签。普通合并永远不会创建 chart 标签。

仅 Chart 示例：

```bash
# 在合并了更新 Chart.yaml version 的 PR 后：
git tag charts/trueforge@0.1.0
git push origin charts/trueforge@0.1.0
```

## 开发 / 浮动 main

外部部署仓库拥有 `truefoundry.yaml`（`git-helm-repo` @ `main`）。构建从源码镜像：

```bash
gh workflow run build-dev-image.yml --ref main
# → tfy.jfrog.io/tfy-images/trueforge:<fullSha>
```

将该 SHA 修补到 `image.tag` 中。密钥仅通过 `secretKeyRef` — 切勿在 git 中使用明文。
不要使用 SHA 标签镜像作为生产 chart 默认值。

## 捆绑的 Chart 依赖

可选的 Postgres/Redis Bitnami 子 chart（`Chart.lock`）。发布时运行 `helm dependency build`。
使用 `postgresql.enabled=false` / `redis.enabled=false` 禁用。

子 chart **镜像** 在 JFrog 上镜像（`values.yaml`）。每次固定版本时镜像一次：

```bash
for img in \
  postgresql:17.6.0-debian-12-r4 \
  redis:8.2.1-debian-12-r0; do
  crane copy "docker.io/bitnamilegacy/${img}" "tfy.jfrog.io/tfy-mirror/bitnamilegacy/${img}"
done
```

升级：编辑 `Chart.yaml` 依赖 → `pnpm chart:deps` → 更新 `values.yaml` 镜像标签 → 镜像。

## 本地验证 Chart

```bash
pnpm chart:deps
pnpm chart:lint
pnpm chart:template
pnpm chart:package
```