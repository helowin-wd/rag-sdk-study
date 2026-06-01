# RAG SDK

一个模块化、类型安全的 RAG（检索增强生成）SDK，使用 TypeScript 构建。本项目提供一组可组合的包，用于构建生产级的 RAG 流水线。

## 项目状态

**阶段：** 初始脚手架搭建完成 — 包结构已定义，待实现具体功能。

## 架构概览

```
rag-sdk/
├── packages/
│   ├── core/           # 核心类型、接口和抽象
│   ├── adapters/       # 提供商适配器（LLM、Embedding、向量存储）
│   ├── indexing/       # 文档加载、分块和索引
│   ├── runtime/        # 流水线执行和编排
│   ├── eval/           # RAG 评估框架
│   ├── observability/  # 链路追踪、日志和监控
│   └── utils/          # 共享工具函数
├── docs/               # 项目文档
└── package.json        # 根工作区配置
```

## 包说明

| 包 | 描述 | 层级 |
|---------|-------------|-------|
| `@rag-sdk/core` | 核心类型、接口和基础抽象 | 基础层 |
| `@rag-sdk/adapters` | 提供商集成（LLM、Embedding、向量存储） | 集成层 |
| `@rag-sdk/indexing` | 文档加载、分块、元数据提取 | 数据处理层 |
| `@rag-sdk/runtime` | 流水线 DAG 执行和编排 | 编排层 |
| `@rag-sdk/eval` | RAG 质量评估指标和测试框架 | 质量层 |
| `@rag-sdk/observability` | 链路追踪、日志和监控 | 可观测层 |
| `@rag-sdk/utils` | 共享工具函数 | 基础设施层 |

## 技术栈

- **语言：** TypeScript（ESM）
- **包管理器：** pnpm（workspaces）
- **运行时：** Node.js（规划中）
- **构建工具：** 待定（tsup / esbuild / tsc）

## 快速开始

```bash
# 安装依赖
pnpm install

# 构建所有包
pnpm -r build

# 运行测试
pnpm -r test
```

## 开发

```bash
# 为特定包添加依赖
pnpm --filter @rag-sdk/core add <dependency>

# 在特定包中运行命令
pnpm --filter @rag-sdk/core run build

# 链接本地包
pnpm --filter @rag-sdk/runtime add @rag-sdk/core
```

## 文档

详细文档请参见 [docs](./docs) 目录：

- [架构说明](./docs/architecture/ARCHITECTURE.md) — 系统设计和包之间的关系
- [项目背景](./docs/context/PROJECT_CONTEXT.md) — 背景、目标和范围
- [SDK 概述](./docs/sdk/OVERVIEW.md) — SDK 设计和开发指南
- [流水线](./docs/pipeline/PIPELINE.md) — RAG 流水线架构
- [决策记录](./docs/decisions/) — 架构决策记录
