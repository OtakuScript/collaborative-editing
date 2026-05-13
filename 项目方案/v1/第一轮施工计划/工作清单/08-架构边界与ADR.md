# 08-V1 架构边界补充方案与 ADR

## 目标

作为 V1 技术方案的补充，明确服务边界、Markdown 文件所有权、权限校验归属、服务协作方式和共享包使用范围，避免后续实现时 `api-service`、`collab-service` 和 `packages/shared` 的职责混淆。

## 文档状态

- [ ✅️ ] 已补充 V1 架构边界方案
- [ ✅️ ] 已补充服务协作流程
- [ ✅️ ] 已记录 ADR-001
- [ ✅️ ] 已独立拆分 ADR 文档
- [ ✅️ ] 已在 `V1 接口文档.md` 补充内部文档访问校验接口契约
- [ ✅️ ] 已在 `V1 技术方案.md` 补充 `Document.file_path` 生成规则
- [ ✅️ ] 已在 `V1 技术方案.md` 补充 viewer 只读协同控制
- [ ✅️ ] 已在 `V1 技术方案.md` 补充自动保存失败处理

## 落地清单

- [ ] 明确 `api-service` 职责边界并落实到目录和模块
- [ ] 明确 `collab-service` 职责边界并落实到目录和模块
- [ ] 明确 Markdown 文件读写所有权并落实到接口和模块
- [ ] 明确权限校验归属并落实到 `api-service`
- [ ] 明确跨服务鉴权契约并落实到内部接口
- [ ] 明确 `packages/shared` 允许内容并落实到包结构
- [ ] 明确 `packages/shared` 禁止内容并在实现中遵守
- [ ] 第一轮骨架按该边界预留位置

## V1 架构边界补充方案

### 方案定位

V1 采用三服务结构：`web-app`、`api-service` 和 `collab-service`。三服务拆分不是为了追求复杂架构，而是因为系统同时存在三类不同职责：

- 页面交互和编辑器体验。
- 用户、工作区、成员、文档元信息和权限规则。
- 多人实时协同、Yjs 状态同步和 Markdown 正文保存。

本补充方案用于约束这三类职责的所有权和协作方式。实现时优先保证职责清晰和可验证，不提前引入 worker、队列、缓存或对象存储。

### 服务协作总览

核心数据流：

```text
web-app
  -> api-service: 登录、工作区、成员、文档元信息、文档访问校验
  -> collab-service: 文档协同连接和实时编辑

collab-service
  -> api-service: 内部文档访问校验，获取 readonly、role、filePath
  -> data/storage: 基于授权后的 filePath 读取和保存 Markdown 正文

api-service
  -> PostgreSQL: 用户、工作区、成员、文档元信息
```

V1 的核心原则：

- 业务权限以 `api-service` 为准。
- 协同正文以 Yjs 运行时状态为准。
- 持久化结果以服务端 `.md` 文件为准。
- 文档正文不通过 REST 保存 API 由客户端直接提交。

## 架构边界清单

### `api-service`

`api-service` 是 V1 的业务事实来源，负责：

- 用户认证和登录态签发。
- 工作区、成员、角色和权限规则。
- 文档元信息，包括 `Document` 记录、标题、所属工作区、创建人和 `file_path`。
- Markdown 文件路径生成规则。
- 向 `collab-service` 提供内部文档访问校验能力。

`api-service` 不负责：

- 实时协同连接管理。
- Yjs 房间和协同状态同步。
- 客户端编辑过程中的正文保存节流。
- 直接接收客户端提交的正文保存 API。

### `collab-service`

`collab-service` 是 V1 的协同运行时，负责：

- Hocuspocus / Yjs WebSocket 服务。
- 根据 `document:{documentId}` 建立协同房间。
- 连接、断开和房间名称日志。
- 后续阶段的 Yjs 文档初始化、变更监听和 debounce 保存触发。
- 基于 `api-service` 返回的授权结果决定用户是否可以进入房间，以及是否只读。

`collab-service` 不负责：

- 用户注册登录。
- 工作区和成员管理。
- 自行复制一套权限规则。
- 接收客户端传入的任意文件路径。
- 让客户端绕过权限校验直接读写 Markdown 文件。

### Markdown 文件所有权

Markdown 文件的安全边界由 `api-service` 决定，运行时读写由 `collab-service` 执行。

- `api-service` 负责生成和保存 `Document.file_path`。
- `api-service` 负责判断用户是否能访问、编辑或管理文档。
- `collab-service` 只能使用 `api-service` 返回的文档授权结果和文件路径。
- `collab-service` 后续执行正文加载和保存时，必须基于服务端生成的相对路径。
- 客户端不得传入文件系统路径。

### 关键流程

#### 创建文档

1. `web-app` 调用 `api-service` 创建文档。
2. `api-service` 校验用户是否可以在工作区创建文档。
3. `api-service` 创建 `Document` 记录。
4. `api-service` 根据 `workspaceId` 和 `documentId` 生成 `file_path`。
5. `api-service` 初始化空 Markdown 文件或带标题的 Markdown 文件。
6. `web-app` 跳转到文档编辑页并连接 `collab-service`。

#### 打开文档

1. `web-app` 先通过 `api-service` 获取文档元信息。
2. `api-service` 校验用户是否可以查看该文档。
3. `web-app` 使用 token 和 `documentId` 连接 `collab-service`。
4. `collab-service` 调用 `api-service` 的内部文档访问校验能力。
5. `api-service` 返回 `allowed`、`readonly`、`role` 和 `filePath`。
6. `collab-service` 基于授权后的 `filePath` 初始化 Yjs 文档。
7. `web-app` 进入协同编辑或只读模式。

#### 保存文档

1. 多个客户端通过 `collab-service` 同步 Yjs 状态。
2. `collab-service` 监听文档变化。
3. `collab-service` debounce 后将 Yjs 文档序列化为 Markdown。
4. `collab-service` 使用授权后的 `filePath` 原子写入 `.md` 文件。
5. `collab-service` 通知或调用 `api-service` 更新 `Document.updated_at`。

### 权限校验归属

权限规则只在 `api-service` 中实现。

- REST API 请求由 `api-service` 自己校验。
- WebSocket 连接由 `collab-service` 调用 `api-service` 的内部校验能力。
- `collab-service` 不直接判断 workspace membership 规则，只消费校验结果。

建议内部校验返回最小结构：

```ts
type DocumentAccessResult = {
  allowed: boolean;
  readonly: boolean;
  documentId: string;
  workspaceId: string;
  filePath: string;
  role: 'owner' | 'editor' | 'viewer';
};
```

完整接口契约见 `../../V1 接口文档.md` 的 `3.1 INT-001 文档访问校验接口`。

### `packages/shared`

`packages/shared` 只放跨应用稳定契约：

- DTO 类型。
- 枚举。
- API 路由常量。
- WebSocket 房间命名规则。
- 环境变量 schema 类型。
- 前后端都需要理解的错误码。

`packages/shared` 禁止放：

- Prisma client。
- 数据库访问逻辑。
- NestJS service / controller。
- React 组件。
- 文件系统读写逻辑。
- 权限规则实现。
- Hocuspocus / Yjs 运行时代码。

## 第一轮落地约束

第一轮仍然只交付可运行骨架：

- `api-service` 只需要健康检查和模块占位。
- `collab-service` 只需要 WebSocket 服务和连接日志。
- Markdown 正文加载、保存、JWT 鉴权和 viewer 只读控制继续暂缓。
- 但目录和占位命名应遵守本文件的边界，避免后续迁移。
- 第一轮创建模块占位时，应为内部文档访问校验、文件路径生成、协同房间命名和共享契约预留清晰位置。

## ADR

完整 ADR 独立维护在：

```text
../../ADR/ADR-001-服务边界与Markdown文件所有权.md
```
