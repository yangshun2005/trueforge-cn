# trueforge Helm Chart

部署 TrueForge 服务器，这是一个同时提供 API 和 UI 的单一容器镜像。**生产**镜像从 npm 安装 `@truefoundry/trueforge`（仓库根目录 [`Dockerfile`](../../Dockerfile)，带有 `APP_VERSION`）。**本地冒烟测试 / 源码构建**使用 [`Dockerfile.dev`](../../Dockerfile.dev)（参见 [`docker-compose.yml`](../../docker-compose.yml)）。

该 Chart 始终以**分布式**模式（`STANDALONE=false`）运行服务器，并连接 Postgres 和 Redis。

Chart 的 `version` / `appVersion` / `image.tag` 在 `main` 分支上维护（由 chart-release bot PR 或人工更新）。发布由 git 标签 `charts/trueforge@<version>` 控制。参见 [`RELEASING.md`](../../RELEASING.md)。

## 开发默认值（暴露前请阅读）

裸执行 `helm install` 有意设计为便于本地/私有集群试用。这些默认值**不具备生产环境安全性**：

| 默认值 | 若 Service / Ingress 可达时的风险 |
| --- | --- |
| `configs.oidc.enabled: false` | 无登录；所有调用者均为共享的本地管理员（`trueforge-default`） |
| `postgresql.auth.password: trueforge` | 众所周知的 Postgres 密码（除非您设置了 `existingSecret` / 强密码） |
| `redis.auth.enabled: false` | 集群网络上未认证的 Redis |

在将 TrueForge 暴露到可信网络之外前：启用 OIDC，更改 Postgres 密码或改用 Secret 存储，并优先使用外部带密码的 Redis（或保持 Redis 仅为 ClusterIP 并受 NetworkPolicy 限制）。参见[生产环境检查清单](#production-checklist)和[设置登录](https://github.com/truefoundry/trueforge/blob/main/docs/authentication/overview.mdx)文档。

## 依赖项

Postgres 和 Redis 作为**捆绑**依赖项提供（Bitnami `postgresql` 和 `redis` charts，从公共 Bitnami OCI 归档拉取并由 `Chart.lock` 固定版本）。它们默认启用，因此基本安装**无需任何必需值**。该 Chart 自动将服务器的 `POSTGRES_*` 和 `REDIS_URL` 环境变量连接到捆绑服务。

Bitnami **charts** 仍然是公开的，但它们引用的**容器镜像**已移至 `docker.io/bitnamilegacy`，因此 `values.yaml` 将子 chart 镜像覆盖为镜像到 TrueFoundry JFrog 镜像仓库（`tfy.jfrog.io/tfy-mirror/bitnamilegacy/...`）的固定标签。这会触发 Bitnami 的镜像仓库防护机制，因此为子 charts 设置了 `global.security.allowInsecureImages: true` 以使其能够渲染。

禁用任一依赖项可改为指向外部服务（见下文）。

## 安装

```bash
helm install trueforge oci://tfy.jfrog.io/tfy-helm/trueforge \
  --version <x.y.z>
```

可选：暴露服务后设置公共源（MCP OAuth 和 OIDC 回调需要）：

```bash
helm upgrade --install trueforge oci://tfy.jfrog.io/tfy-helm/trueforge \
  --version <x.y.z> \
  --set server.publicBaseUrl=https://trueforge.example.com
```

## Postgres

默认捆绑（`postgresql.enabled=true`）。该 Chart 附带一个**开发**密码（`trueforge`）。在任何共享/公共部署前更改 `postgresql.auth.password` 或设置 Bitnami 的 `postgresql.auth.existingSecret`。同时根据需要设置 `postgresql.auth.username` 和 `postgresql.auth.database`；服务器自动连接到捆绑实例。

要使用**外部** Postgres，设置 `postgresql.enabled=false` 并提供 `externalPostgres.host`（+ `port`、`database`、`user`）。将 `externalPostgres.password` 设置为字符串（内联为环境变量 `value`）或 `valueFrom.secretKeyRef`（生产环境推荐——您需自行创建 Secret）：

```yaml
postgresql:
  enabled: false
externalPostgres:
  host: postgres.databases.svc
  password:
    valueFrom:
      secretKeyRef:
        name: my-postgres-secret
        key: password
```

## Redis

服务器始终以对等模式运行（`STANDALONE=false`），因此始终需要 Redis。默认捆绑（`redis.enabled=true`，**认证已禁用**——仅当 Redis 在集群信任边界外不可达时才可接受）。要使用**外部** Redis，设置 `redis.enabled=false` 并提供 `externalRedis.url` 作为字符串或 `valueFrom.secretKeyRef`：

```yaml
redis:
  enabled: false
externalRedis:
  url:
    valueFrom:
      secretKeyRef:
        name: my-redis-secret
        key: redis-url
```

对于带密码的 Redis，推荐使用外部实例并通过 `valueFrom` 加载 `REDIS_URL`。

## OIDC

在 `configs.oidc` 下配置 IdP 登录。当 `enabled` 为 false（默认值）时，不设置 `OIDC_*` 环境变量，服务器使用**固定的本地管理员**身份——任何能访问 API/UI 的人都拥有完全管理员权限。任何共享或公共部署都应启用 OIDC。启用时，设置字符串 `issuerUrl` 和 `clientId`，并将 `clientSecret` 设置为字符串或 `valueFrom.secretKeyRef`（生产环境推荐使用 valueFrom）。

同时将 `server.publicBaseUrl` 设置为公共源，并在您的 IdP 中注册 `{publicBaseUrl}/api/v1/auth/callback`。

```yaml
server:
  publicBaseUrl: https://trueforge.example.com
configs:
  oidc:
    enabled: true
    issuerUrl: https://idp.example.com/oauth2/default
    clientId: trueforge
    clientSecret:
      valueFrom:
        secretKeyRef:
          name: trueforge-oidc
          key: client-secret
    # 可选的声明覆盖（显示默认值）：
    # userReferenceClaim: sub
    # userDisplayNameClaim: name
    # userRoleClaim: groups
    # adminRoleValue: admin
    # scopes: "openid,profile,email,groups"
    # 可选的电子邮件允许列表（精确 + * 通配符）。空 = 不限制。
    # allowedEmails: "alice@acme.com,*@partner.com"
```

## 使用 Secrets

生产环境中，推荐使用 Kubernetes Secrets 而非 values 文件中的字面量。此 Chart **不会**为 Chart 自有字段创建 Secrets——请提供 `valueFrom.secretKeyRef`（或自行创建 Secrets 并指向它们）。

接受字符串 | `valueFrom.secretKeyRef` 的字段：`externalPostgres.password`、`externalRedis.url`、`configs.oidc.clientSecret`。`configs.oidc.issuerUrl` 和 `clientId` 仅为纯字符串。

**捆绑的 Postgres 密码**仍使用 Bitnami 的 API（`postgresql.auth.existingSecret`，键为 `password`）：

```yaml
postgresql:
  auth:
    existingSecret: my-postgres-secret
    # 设置 existingSecret 时，请将 password 留空/不使用
```

## 额外对象

通过 `extraObjects` 可部署与服务器并行的任意清单（例如 Istio `VirtualService`、`Gateway`、Ingress 或 NetworkPolicy）。每个条目都是一个完整对象，通过 `tpl` 渲染，因此 Helm 模板和 Chart 辅助函数可以解析：

```yaml
extraObjects:
  - apiVersion: networking.istio.io/v1
    kind: VirtualService
    metadata:
      name: '{{ include "trueforge.fullname" . }}'
    spec:
      hosts:
        - trueforge.example.com
      gateways:
        - istio-system/public-gateway
      http:
        - route:
            - destination:
                host: '{{ include "trueforge.fullname" . }}'
                port:
                  number: '{{ .Values.service.port }}'
```

## 常用值

| 值                 | 默认值                             | 描述                           |
| --------------------- | ----------------------------------- | ------------------------------------- |
| `server.replicaCount` | `1`                                 | 服务器副本数。            |
| `image.repository`    | `tfy.jfrog.io/tfy-images/trueforge` | 镜像仓库。                     |
| `image.tag`           | chart `appVersion`                  | 镜像标签；发布时标记。        |
| `server.publicBaseUrl`| `""`                                | OAuth/OIDC 回调的公共源（MCP OAuth / OIDC 必需）。 |
| `configs.oidc.enabled`| `false`                             | 注入 `OIDC_*` 环境变量以启用 IdP 登录。    |
| `postgresql.enabled`  | `true`                              | 捆绑 Bitnami Postgres 子 chart。 |
| `redis.enabled`       | `true`                              | 捆绑 Bitnami Redis 子 chart。    |
| `service.type`        | `ClusterIP`                         | Service 类型。                         |
| `service.port`        | `8790`                              | Service 端口。                         |
| `server.port`         | `8790`                              | 容器端口（`PORT`）。              |
| `autoscaling.enabled` | `false`                             | 启用 HorizontalPodAutoscaler。     |
| `podDisruptionBudget.enabled` | `false`                       | 启用 PodDisruptionBudget（`minAvailable` 默认为 `1`）。 |
| `podSecurityContext`  | 非 root UID/GID `10001`            | Pod 级别的受限安全默认值。 |
| `securityContext`     | 只读根文件系统 + 丢弃所有能力 | 容器级别的受限安全默认值。 |
| `resources`           | 100m/256Mi 请求，200m/512Mi 限制 | 容器 CPU、内存和临时存储请求/限制。 |

服务器默认使用 RollingUpdate 策略（`server.strategy`）；控制器固定为单副本并使用 `Recreate`，两者均不暴露。

其他可用值（默认不生效）：`priorityClassName`、`topologySpreadConstraints`、`initContainers`、`extraContainers`、`extraVolumes`、`extraVolumeMounts`、`service.annotations`、`service.labels`、`startupProbe`。

服务器容器默认在 `/tmp` 挂载一个 `emptyDir`，以便镜像可以在 `readOnlyRootFilesystem: true` 下运行。设置 `resources.limits.ephemeral-storage` 时，也会设置 `/tmp` 的 `emptyDir.sizeLimit`。

## 生产环境检查清单

- **启用 `configs.oidc`** — 保持禁用将向任何能访问服务器的人授予共享管理员权限。
- **替换捆绑的 Postgres 密码**（`trueforge`）或设置 `postgresql.auth.existingSecret`。
- 将捆绑的 Redis（`redis.auth.enabled: false`）视为仅限集群内部使用，或通过 `externalRedis.url` 切换到外部带密码的 Redis。
- 在使用 MCP OAuth 或 OIDC 之前，将 `server.publicBaseUrl` 设置为真实的公共源。
- 推荐使用 `valueFrom.secretKeyRef` 存储 Postgres 密码、Redis URL 和 OIDC 客户端密钥；不要将密钥提交到 values 文件中。
- 生产环境 HA 推荐使用外部托管的 Postgres/Redis，而非捆绑的子 charts。
- 在启用 HPA 之前调整容器 `resources`（尤其是 CPU 请求）。
- 默认的 `tfy.jfrog.io` 镜像和 Helm chart 可匿名拉取——仅当您覆盖为私有镜像仓库时才设置 `imagePullSecrets`。
- 运行多个副本时启用 `podDisruptionBudget`（默认为 `minAvailable: 1`；请仅设置 `minAvailable` 或 `maxUnavailable` 中的一个）。