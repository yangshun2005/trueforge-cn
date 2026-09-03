# 技术规范（PostgreSQL）

- 时间列**必须**使用 `timestamptz`（带时区的 `timestamp`）。禁止使用不带时区的 `timestamp`。
- 应用时间戳**必须**按 UTC 时刻处理。使用 `Date.prototype.toISOString()` 进行序列化（始终为带毫秒的 `...Z` 格式）。
- 禁止在循环内执行数据库查询（避免 N+1 问题）。应优先使用单次批量查询、`JOIN` 或 `IN`/`ANY` 查找，而不是逐条往返查询。
- Postgres 迁移**必须**在 `up`/`down` 开头执行 `SET LOCAL lock_timeout = '5s'`，使等待中的 DDL 快速失败，避免在锁队列中阻塞后续查询（包括 `SELECT`）。
- 所有应用表及 Kysely 迁移记录均存放于 `trueforge` schema 中（而非 `public`）。`runMigrations` 会在 Migrator 运行前引导创建该 schema（创建并迁移旧的 `public` 表）；`createDb` 会设置 `search_path=trueforge`。