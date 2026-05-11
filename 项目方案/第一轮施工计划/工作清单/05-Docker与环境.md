# 05-Docker 与环境

## 目标

建立本地开发所需 Docker Compose 和环境变量配置，确保数据库和服务端口约定清晰。

## 工作清单

- [ ] 创建 `docker-compose.yml`
- [ ] 添加 `postgres` 服务
- [ ] 添加 `api-service` 服务定义
- [ ] 添加 `collab-service` 服务定义
- [ ] 添加 `web-app` 服务定义
- [ ] 配置 PostgreSQL 数据卷
- [ ] 配置 `data/storage` 挂载目录
- [ ] 配置端口：`5173`
- [ ] 配置端口：`3000`
- [ ] 配置端口：`1234`
- [ ] 配置端口：`5432`
- [ ] 在 `.env.example` 写入 `DATABASE_URL`
- [ ] 在 `.env.example` 写入 `JWT_SECRET`
- [ ] 在 `.env.example` 写入 `STORAGE_ROOT`
- [ ] 在 `.env.example` 写入 `COLLAB_WS_URL`
- [ ] 在 `.env.example` 写入 `VITE_API_BASE_URL`
- [ ] 在 `.env.example` 写入 `VITE_COLLAB_WS_URL`

## 验收标准

- `docker-compose.yml` 描述 V1 第一轮需要的服务。
- `postgres` 可通过 Docker Compose 启动。
- `.env.example` 包含前端、后端、协同和数据库所需变量。
- 本地端口约定和主施工计划一致。

