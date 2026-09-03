- `withTransaction` 回调必须只执行本地数据库操作：不得 `await` 一个 `fetch`、SDK 客户端、Redis 或其他远程 I/O（包括通过辅助函数）。在开启事务前完成远程工作。
- `withTransaction` 是 `db.transaction().execute(callback)`：解析时提交，抛出时回滚。从回调中返回领域数据；在其后构建成功的 `c.json(...)`。必须撤销写入的失败操作必须 `throw`（例如 `HTTPException`）；绝不能在其内部 `return c.json({ error: ... }, status)` —— 返回的 Response 会提交事务，而客户端仍会看到错误。

### OpenAPI 命名约定

线上字段保持 `snake_case`。新端点必须遵循此约定；对已发布 Schema 的重命名会破坏 `packages/trueforge-sdk`。

**路径：** `/api/v1/{collection}` 下的复数 kebab 命名，嵌套路径 `/…/{id}/{subcollection}`，设置 `/api/v1/settings/{collection}`，目录 `/api/v1/catalogs/{collection}`，内部 `/api/internal/{collection}`。ID 使用 `{resource}_id`；基于名称的操作使用 `{name}` —— 不要在一个资源的多个动词间混用 ID 和名称。

如果某个设置资源是每租户一个（例如沙箱提供商），为保持 URL 一致性，请保留复数路径 `/api/v1/settings/sandbox-providers`，但 Fern 方法是 `get`/`create_or_update`，返回单个对象 —— 而非返回数组的 `list`。

**动词：** `GET` 列表/获取 → 200；`POST` 创建 → **201**；`PUT` 替换/更新 → 200；`DELETE` → **200**，并返回 `DeleteFooResponse`（`{}`），可选 `DeleteFoosRequestQuery`；操作 `POST /{id}/{action}`。

**Schema**（资源 `Foo`，集合 `foos`；OpenAPI 条目名称是 PascalCase 单数）：

| 操作                     | 请求                                   | 响应                                        |
| ------------------------ | -------------------------------------- | ------------------------------------------- |
| `GET /foos`              | `ListFoosRequestQuery`（如有）         | `ListFoosResponse`                          |
| `POST /foos`             | `CreateFooRequest`                     | `GetFooResponse`（或 `CreateFooResponse`）  |
| `GET /foos/{foo_id}`     | —                                      | `GetFooResponse`                            |
| `PUT` 更新/替换          | `UpdateFooRequest`                     | `GetFooResponse`（或 `UpdateFooResponse`）  |
| `DELETE /foos/{foo_id}`  | `DeleteFoosRequestQuery`（可选）       | `DeleteFooResponse`（`{}`）                 |

当创建/更新返回相同条目时，优先复用 `GetFooResponse`。`GET /auth/me` 是 `GetMeResponse`。OpenAPI 名称中的首字母缩略词保持大写（`MCPServer`，而非 `McpServer`）。

**请求体与清单：** `FooManifest` 仅是持久化的 jsonb 文档。创建/更新的 OpenAPI 请求体必须是 `CreateFooRequest` / `UpdateFooRequest`，并带有显式包装 —— 切勿将清单字段扁平化到请求根级别，也切勿将请求 Schema 别名为 `FooManifest`：

```ts
// CreateFooRequest / UpdateFooRequest
{
  manifest: FooManifest; // 仅存储的文档
  dry_run?: boolean;     // 操作级字段位于 manifest 旁边
}
```

会话/轮次式的创建（非存储清单）保持扁平的 `Create*Request`，不带 `manifest` 键。

设置列表 → `ListFoosResponse`；聊天 → `ListAvailableFoosResponse`。目录是单个 `GET /api/v1/catalogs/{collection}`，返回整个数据块 → `GetFooCatalogResponse` / 条目 `CatalogFoo`（而非 `ListCatalog*` 列表端点）。

嵌套子资源 `Bar`：`ListBarsResponse`、`CreateBarRequest`、`GetBarResponse`；Fern `list_bars`、`create_bar`、…

**信封：** 成功 `{ data: Item | Item[] }`（+ 通过 `fernExtensions.ts` 令牌契约的 `pagination`）；错误 `RequestErrorResponse`。

**Fern：** 设置 `x-fern-sdk-group-name` / `x-fern-sdk-method-name`（`list`/`get`/`create`/`update`/`create_or_update`/`delete` + snake_case 操作）。PUT 创建或替换必须使用 `create_or_update`，绝不要用 `upsert`。不要手动编辑 OpenAPI 或 `packages/trueforge-sdk`。Schema 位于 `src/schemas/`，并带有匹配的 `.openapi('…')` 名称；类型通过 `z.infer` 生成。

**无内联对象 Schema：** 不要在其他对象内嵌套匿名的 `z.object({ … })`。将每个嵌套对象提取为顶层命名 Schema，并赋予有意义的 `.openapi('…')` 名称（例如 `FooAuth`、`FooManifest`），以便 OpenAPI 发出 `$ref`，其他 Schema 可以复用。原始类型、原始类型数组以及对现有命名 Schema 的 `$ref` 可以内联。

**避免 `allOf`：** 新的线上 Schema 必须生成扁平化的 OpenAPI 对象。

- **绝不能使用 `z.merge`** —— 它是一个陷阱（不透明组合、OpenAPI `allOf`、令人意外的覆盖）。优先使用 `z.object({ ...A.shape, ...B.shape })`，或让基类保持匿名并仅使用 `.extend`。
- 不要对已有 `.openapi('Name')` 的 Schema 使用 `.extend()` —— 使用 `z.object({ ...Base.shape, … })` 或让基类保持匿名。
- 不要对命名的 `$ref` 字段使用 `.describe()`（`NameSchema.describe(…)`）—— 只描述一次共享 Schema，或内联约束。
- 真正的多态使用判别式 `oneOf` 是可以的；单变体联合必须是普通对象别名（不能有单成员 `oneOf`）。