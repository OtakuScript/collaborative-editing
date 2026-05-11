# 03-后端 api-service

## 目标

搭建 `apps/api-service` NestJS 服务骨架，建立 Prisma schema 和健康检查接口。

## 工作清单

- [ ] 创建 `apps/api-service`
- [ ] 初始化 NestJS 项目
- [ ] 安装 Prisma
- [ ] 安装 PostgreSQL driver
- [ ] 配置环境变量读取
- [ ] 创建 Prisma schema
- [ ] 定义 `User` model
- [ ] 定义 `Workspace` model
- [ ] 定义 `WorkspaceMember` model
- [ ] 定义 `Document` model
- [ ] 定义 `WorkspaceRole` enum：`owner`、`editor`、`viewer`
- [ ] 创建 `health` module
- [ ] 实现 `GET /health`
- [ ] 创建 `auth` module 占位
- [ ] 创建 `workspaces` module 占位
- [ ] 创建 `members` module 占位
- [ ] 创建 `documents` module 占位
- [ ] 创建 `permissions` module 占位
- [ ] 创建 `storage` module 占位
- [ ] 确认 API 服务可启动
- [ ] 确认 `GET /health` 返回正常

## 验收标准

- `api-service` 可以本地启动。
- `GET /health` 返回 `status: ok`。
- Prisma schema 包含 V1 核心数据模型。
- 业务模块先占位，不提前实现完整业务逻辑。

