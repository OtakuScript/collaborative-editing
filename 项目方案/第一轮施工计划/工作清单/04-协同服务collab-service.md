# 04-协同服务 collab-service

## 目标

搭建 `apps/collab-service` 协同服务骨架，确认 Hocuspocus WebSocket 服务可以启动和接受连接。

## 工作清单

- [ ] 创建 `apps/collab-service`
- [ ] 初始化 TypeScript Node 项目
- [ ] 安装 Hocuspocus
- [ ] 安装 Yjs
- [ ] 配置服务入口
- [ ] 默认监听端口 `1234`
- [ ] 支持创建协同房间
- [ ] 记录连接日志
- [ ] 记录断开日志
- [ ] 记录房间名称
- [ ] 确认服务可启动
- [ ] 确认 WebSocket 可以连接

## 验收标准

- `collab-service` 可以本地启动。
- 服务监听 `1234` 端口。
- WebSocket 客户端连接时，服务端能打印连接和房间日志。
- 第一轮暂不实现 JWT 鉴权、只读权限、Markdown 文件读写和自动保存。

