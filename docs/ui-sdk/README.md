# TrueForge UI SDK — 文档

`@truefoundry/trueforge-ui` 的 Mintlify 站点。

## 本地开发

```bash
npm i -g mint
mint dev
```

在 http://localhost:3000 打开。发布前运行 `mint broken-links`。

## 目录结构

```
docs.json          导航、主题、品牌
index.mdx          落地页
get-started/       安装、快速入门
concepts/          架构、代理模式、服务器契约
guides/            布局、主题、插槽、自定义服务器、MCP OAuth、故障排查
reference/         TrueForgeUI、主题、容器、原子组件、Hooks、服务器
images/            徽标和截图
snippets/          可复用的 MDX 片段
```

有关编写约定以及哪些 SDK 文件具有权威性，请参阅 `CLAUDE.md`。