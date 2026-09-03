# 贡献指南

感谢您为 `@truefoundry/trueforge-ui` 贡献代码。

## 开发

在 monorepo 根目录下执行：

```bash
pnpm install
pnpm --filter @truefoundry/trueforge-ui typecheck
pnpm --filter @truefoundry/trueforge-ui test
pnpm --filter @truefoundry/trueforge-ui build
pnpm --filter @truefoundry/trueforge-ui lint
```

- 建议提交小而专注的 Pull Request。
- 遵循 `src/atoms`（展示型）和 `src/containers`（运行时接线）中的现有模式。
- Atoms 不得导入 `@assistant-ui/*` 或 TrueFoundry 运行时 Hook。
- Containers 通过 `useSlot` 解析 atoms，而非直接导入 atoms（类型导入不受限制）。
- 对于用户可见的更改，请更新 `CHANGELOG.md` 中的 **Unreleased** 部分。

## Pull Request

1. 向 `main` 分支发起 Pull Request。
2. 确保 CI 通过（`typecheck`、`test`、`build`、`lint`）。
3. 如果更改对用户可见，请确认 Unreleased 变更日志条目。

## 报告 Bug

请提供包版本、对等依赖版本，以及尽可能小的复现示例。

## 安全

请勿针对漏洞提交公开 Issue。请参阅 [SECURITY.md](./SECURITY.md)。