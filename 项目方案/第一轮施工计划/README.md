# Markdown 协同编辑平台 V1 第一轮施工计划

## 1. 第一轮目标

第一轮目标是搭出可运行工程骨架，而不是一次性完成完整 V1。

本轮完成后，项目应该具备：

- `pnpm` monorepo 基础结构。
- 前端 `web-app` 可以启动并访问基础路由页面。
- 后端 `api-service` 可以启动并提供健康检查接口。
- 协同服务 `collab-service` 可以启动 WebSocket 服务。
- PostgreSQL 可以通过 Docker Compose 启动。
- Prisma schema 建立 V1 核心数据模型。
- Docker Compose 能描述本地开发所需服务。

本轮暂不实现完整登录、工作区业务、文档 CRUD、Tiptap 编辑器、Yjs 正文协同和 `.md` 自动保存。

## 2. 技术选型

包管理器：

- `pnpm`

前端：

- `React 18`
- `TypeScript`
- `Vite`
- `React Router`
- `TanStack Query`
- `Zustand`
- `Ant Design`

后端：

- `NestJS`
- `Prisma`
- `PostgreSQL`

协同服务：

- `Hocuspocus`
- `Yjs`

部署与本地环境：

- `Docker Compose`

## 3. 目录结构

第一轮建议创建以下结构：

```text
.
├── apps/
│   ├── web-app/
│   ├── api-service/
│   └── collab-service/
├── packages/
│   └── shared/
├── data/
│   └── storage/
├── 项目方案/
│   ├── AI Agent 文档协作平台 V1 项目方案.md
│   └── 第一轮施工计划/
│       └── README.md
├── docker-compose.yml
├── package.json
├── pnpm-workspace.yaml
├── .env.example
├── .gitignore
├── README.md
└── LICENSE
```

说明：

- `apps/web-app` 存放前端单页应用。
- `apps/api-service` 存放 REST API 服务。
- `apps/collab-service` 存放实时协同 WebSocket 服务。
- `packages/shared` 存放前后端共享类型和工具。
- `data/storage` 作为本地 Markdown 文件存储目录，实际内容不提交 Git。

## 4. Monorepo 基础

根目录需要提供：

- `package.json`
- `pnpm-workspace.yaml`
- `.gitignore`
- `.env.example`
- `docker-compose.yml`

根目录脚本建议：

```json
{
  "scripts": {
    "dev": "pnpm -r --parallel dev",
    "build": "pnpm -r build",
    "lint": "pnpm -r lint",
    "typecheck": "pnpm -r typecheck"
  }
}
```

`pnpm-workspace.yaml` 覆盖：

```yaml
packages:
  - "apps/*"
  - "packages/*"
```

`.gitignore` 至少包含：

```gitignore
node_modules
dist
.env
data/postgres
data/storage/*
!data/storage/.gitkeep
```

## 5. 前端骨架

### 5.1 目标

`apps/web-app` 第一轮只做基础应用壳，不实现真实业务闭环。

需要具备：

- Vite React TypeScript 项目。
- React Router 路由。
- TanStack Query Provider。
- Zustand store 占位。
- Ant Design 全局配置。
- 基础布局和占位页面。

### 5.2 路由

第一轮创建以下路由：

- `/login`
- `/register`
- `/workspaces`
- `/workspaces/:workspaceId/documents`
- `/documents/:documentId`

页面内容可以先是占位文案，但页面必须能正常访问。

### 5.3 页面占位

登录页：

- 显示邮箱、密码输入框。
- 显示登录按钮。
- 第一轮不要求真实提交。

注册页：

- 显示邮箱、昵称、密码输入框。
- 显示注册按钮。
- 第一轮不要求真实提交。

工作区页：

- 显示“工作区列表”标题。
- 显示空状态或假数据。

文档列表页：

- 显示“文档列表”标题。
- 显示新建文档按钮占位。

文档编辑页：

- 显示顶部工具栏占位。
- 显示编辑器区域占位。
- 显示连接状态占位。

## 6. API 服务骨架

### 6.1 目标

`apps/api-service` 第一轮搭建 NestJS 服务和 Prisma 数据模型。

需要具备：

- NestJS 应用可启动。
- `GET /health` 返回健康状态。
- Prisma 可连接 PostgreSQL。
- V1 核心 schema 已定义。
- 业务模块目录已建立。

### 6.2 模块

第一轮建立模块占位：

- `auth`
- `workspaces`
- `members`
- `documents`
- `permissions`
- `storage`
- `health`

第一轮只要求 `health` 有可用接口，其余模块可以先创建目录和基础文件。

### 6.3 Prisma 数据模型

第一轮 schema 包含：

- `User`
- `Workspace`
- `WorkspaceMember`
- `Document`

字段按主技术方案执行：

- 用户表保存邮箱、昵称、密码 hash。
- 工作区表保存名称和 owner。
- 成员表保存 workspace、user、role。
- 文档表保存 workspace、title、file_path、created_by。

角色枚举：

- `owner`
- `editor`
- `viewer`

### 6.4 健康检查接口

接口：

```http
GET /health
```

返回示例：

```json
{
  "status": "ok",
  "service": "api-service"
}
```

## 7. 协同服务骨架

### 7.1 目标

`apps/collab-service` 第一轮只要求 WebSocket 服务可以启动并接受连接。

需要具备：

- Hocuspocus 服务启动。
- 默认端口 `1234`。
- 支持根据文档名建立房间。
- 打印连接、断开和房间名称日志。

### 7.2 房间命名

后续正式协同房间格式：

```text
document:{documentId}
```

第一轮可以先只记录客户端传入的 document name，不做完整鉴权。

### 7.3 暂缓内容

第一轮暂不做：

- JWT 鉴权。
- viewer 只读控制。
- Markdown 文件读取。
- Markdown 自动保存。
- awareness 在线成员 UI 联动。

这些放到后续协同专项计划中施工。

## 8. Docker 与环境变量

### 8.1 Docker Compose 服务

第一轮 `docker-compose.yml` 至少包含：

- `postgres`
- `api-service`
- `collab-service`
- `web-app`

如果第一轮容器化前端调试成本过高，可以先保证 `postgres` 稳定运行，前端、后端、协同服务使用本机 `pnpm dev` 启动；但 compose 文件仍应保留完整服务定义。

### 8.2 端口

本地端口：

- `web-app`: `5173`
- `api-service`: `3000`
- `collab-service`: `1234`
- `postgres`: `5432`

### 8.3 环境变量

`.env.example` 包含：

```text
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/collab_editing
JWT_SECRET=change-me
STORAGE_ROOT=./data/storage
COLLAB_WS_URL=ws://localhost:1234
VITE_API_BASE_URL=http://localhost:3000
VITE_COLLAB_WS_URL=ws://localhost:1234
```

## 9. 工作清单

后续施工以 `工作清单/` 目录中的细分文档为准，总说明里只保留清单索引。每完成一项，需要同步更新对应细分清单文档，并在对话中说明本次完成了哪些清单项。

清单目录：

```text
项目方案/第一轮施工计划/工作清单/
```

清单文件：

- [ ] `工作清单/00-施工前检查.md`：施工前环境、分支和工作区状态检查。
- [ ] `工作清单/01-Monorepo基础.md`：根目录配置、workspace、基础目录和脚本。
- [ ] `工作清单/02-前端web-app.md`：前端项目骨架、基础 Provider、布局和路由占位页。
- [ ] `工作清单/03-后端api-service.md`：NestJS API 骨架、Prisma schema 和健康检查。
- [ ] `工作清单/04-协同服务collab-service.md`：Hocuspocus/Yjs 服务骨架和连接日志。
- [ ] `工作清单/05-Docker与环境.md`：Docker Compose、环境变量、端口和挂载目录。
- [ ] `工作清单/06-验收.md`：安装、构建、启动、路由、健康检查和 WebSocket 验收。
- [ ] `工作清单/07-完成标志.md`：第一轮完成判定和范围控制。

## 10. 验收标准

第一轮完成后，需要满足：

- `pnpm install` 可以安装依赖。
- `pnpm typecheck` 可以执行并通过。
- `pnpm build` 可以执行并通过。
- `docker compose up postgres` 可以启动数据库。
- `api-service` 本地启动后，`GET /health` 返回正常。
- `web-app` 本地启动后，可以访问 `http://localhost:5173`。
- 前端五个基础路由都能访问。
- `collab-service` 本地启动后，可以监听 `1234` 端口。
- WebSocket 客户端连接协同服务时，服务端能打印连接日志。

## 11. 第一轮不处理的问题

以下内容不进入第一轮：

- 真实注册登录逻辑。
- JWT 鉴权和前端登录态保持。
- 工作区 CRUD。
- 文档 CRUD。
- Tiptap 编辑器真实接入。
- Yjs 正文协同。
- Markdown 文件读写。
- 自动保存。
- 权限系统。
- E2E 测试。

这些会拆到第二轮及之后的深入计划。

## 12. 后续深入计划

后续建议在本目录继续新增：

```text
02-认证与工作区.md
03-文档管理与文件存储.md
04-Markdown编辑器.md
05-实时协同.md
06-自动保存与验收.md
```

推荐施工顺序：

1. 认证与工作区。
2. 文档管理与服务端 `.md` 文件。
3. Markdown 所见即所得编辑器。
4. Yjs/Hocuspocus 实时协同。
5. 自动保存、权限补齐和验收测试。
