# Collaborative Editing
Collaborative-Editing in agent project

## 项目文档入口

V1 文档集中放在：

```text
项目方案/v1/README.md
```

其中包含：

- `V1 项目方案.md`
- `V1 技术方案.md`
- `V1 接口文档.md`
- `V1 测试验收方案.md`
- `ADR/`
- `第一轮施工计划/`

## 环境要求

本项目第一轮施工和本地开发需要：

- Node.js 20+
- pnpm
- Docker Desktop
- Docker Compose

验证命令：

```powershell
node -v
pnpm -v
docker version
docker compose version
```

当前已验证环境：

- Node.js: v24.9.0
- pnpm: 10.31.0
- Docker Desktop: 4.73.0
- Docker Engine: 29.4.3
- Docker Compose: v5.1.3

## Monorepo 基础说明

- `package.json`：根目录工程入口，提供统一的 `dev`、`build`、`lint`、`typecheck` 脚本。
- `pnpm-workspace.yaml`：声明 pnpm workspace 范围，目前覆盖 `apps/*` 和 `packages/*`。
- `apps/`：应用项目目录，后续用于放置前端、后端 API、协同服务等子项目。
- `packages/`：共享包目录，后续用于放置跨应用复用的代码。
- `packages/shared`：预留的共享基础包目录，可放公共类型、常量和工具函数。
- `data/storage`：本地运行时存储目录，实际存储内容不提交 Git。
- `data/storage/.gitkeep`：用于保留空目录结构。

根目录脚本说明：

- `pnpm dev`：递归执行 workspace 内子项目的 `dev` 脚本，用作统一开发入口。
- `pnpm typecheck`：递归执行 workspace 内子项目的 `typecheck` 脚本，用于统一 TypeScript 类型检查。
