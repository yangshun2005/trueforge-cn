# 贡献指南

感谢您对本 SDK 的关注并愿意贡献代码！本文档提供了参与项目贡献的指导方针。

## 快速开始

### 环境要求

- Node.js 20 或更高版本
- pnpm 包管理器

### 安装

安装项目依赖：

```bash
pnpm install
```

### 构建

构建项目：

```bash
pnpm build
```

### 测试

运行测试套件：

```bash
pnpm test
```

运行特定类型的测试：
- `pnpm test:unit` - 运行单元测试
- `pnpm test:wire` - 运行 Wire/集成测试

### 代码检查与格式化

检查代码风格：

```bash
pnpm run lint
pnpm run format:check
```

修复代码风格问题：

```bash
pnpm run lint:fix
pnpm run format:fix
```

或者使用组合检查命令：

```bash
pnpm run check:fix
```

## 关于生成的代码

**重要提示**：本 SDK 中的大部分文件是由 [Fern](https://buildwithfern.com) 根据 API 定义自动生成的。对生成文件的直接修改将在下次 SDK 生成时被覆盖。

### 生成的文件

以下目录包含生成的代码：
- `src/api/` - API 客户端类和类型
- `src/serialization/` - 序列化/反序列化逻辑
- `src/` 中的大部分 TypeScript 文件

### 如何进行自定义

如果您需要自定义 SDK，有两种选择：

#### 方案 1：使用 `.fernignore`

对于需要在 SDK 重新生成后保留的自定义代码：

1. 在项目根目录创建 `.fernignore` 文件
2. 添加您希望保留的文件模式（语法类似于 `.gitignore`）
3. 将您的自定义代码添加到这些文件中

`.fernignore` 中列出的文件在 SDK 重新生成时不会被覆盖。

更多信息，请参阅 [Fern 自定义代码文档](https://buildwithfern.com/learn/sdks/overview/custom-code)。

#### 方案 2：为生成器贡献代码

如果您想改变此 SDK 所有用户的代码生成方式：

1. TypeScript SDK 生成器位于 [Fern 仓库](https://github.com/fern-api/fern) 中
2. 生成器代码位于 `generators/typescript/sdk/` 目录下
3. 遵循 [Fern 贡献指南](https://github.com/fern-api/fern/blob/main/CONTRIBUTING.md)
4. 提交包含您对生成器修改的 Pull Request

此方案最适合以下场景：
- 修复生成代码中的 Bug
- 使所有用户受益的新功能
- 改进代码生成模式

## 进行修改

### 工作流程

1. 为您的修改创建新分支
2. 进行您的修改
3. 运行测试以确保没有破坏任何功能：`pnpm test`
4. 运行代码检查和格式化：`pnpm run check:fix`
5. 构建项目：`pnpm build`
6. 使用清晰的提交信息提交您的修改
7. 推送您的分支并创建 Pull Request

### 提交信息

编写清晰、描述性的提交信息，说明修改了什么以及为什么修改。

### 代码风格

本项目使用自动化代码格式化和检查工具。在提交前运行 `pnpm run check:fix` 以确保您的代码符合项目的风格指南。

## 有疑问或遇到问题？

如果您有疑问或遇到问题：

1. 查阅 [Fern 文档](https://buildwithfern.com)
2. 搜索现有的 [GitHub Issues](https://github.com/fern-api/fern/issues)
3. 如果您的疑问尚未得到解答，请新建一个 Issue

## 许可证

通过为此项目贡献代码，您同意您的贡献将按照与项目相同的许可证进行授权。