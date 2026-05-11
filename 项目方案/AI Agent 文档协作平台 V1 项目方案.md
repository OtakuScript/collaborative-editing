# Markdown 协同编辑平台 V1 技术方案

## 1. 项目目标

本项目 V1 的目标是做一个轻量的 Markdown 协同编辑平台，让团队成员可以登录系统，在工作区内创建、编辑和共同维护 Markdown 文档。

V1 不再定位为完整的研发文档平台，也不做 AI Agent 辅助编辑。第一版只把“多人同时编辑一篇 Markdown 文档，并可靠保存为 `.md` 文件”这条主线跑通。

核心目标：

- 支持用户注册、登录和基础鉴权。
- 支持工作区、成员和文档管理。
- 支持 Markdown 所见即所得编辑。
- 支持多人实时协同编辑和在线状态展示。
- 支持自动保存到服务端真实 `.md` 文件。
- 支持服务重启后从 `.md` 文件恢复文档内容。

## 2. V1 功能边界

### 2.1 V1 要做

- 邮箱密码注册、登录、退出。
- 登录态路由保护。
- 工作区创建和工作区列表。
- 工作区成员关系和基础角色权限。
- Markdown 文档创建、打开、重命名、删除。
- 所见即所得 Markdown 编辑。
- 多人实时协同编辑。
- 在线成员和连接状态展示。
- 自动保存文档内容到服务端 `.md` 文件。
- Docker Compose 本地部署。

## 3. 用户角色与权限

V1 保留三个角色：

- `owner`：工作区拥有者，可以管理工作区、成员和所有文档。
- `editor`：可以创建、编辑、重命名和删除自己有权限的文档。
- `viewer`：只能查看文档，不能修改正文或文档元信息。

权限规则：

- 未登录用户不能进入工作区和文档页面。
- 用户只能访问自己所属工作区内的文档。
- `owner` 可以邀请或移除成员。
- `editor` 可以编辑文档正文。
- `viewer` 可以进入文档页面，但编辑器处于只读状态。
- 协同服务建立连接时必须校验用户登录态和文档访问权限。

## 4. 页面与交互设计

前端采用单页应用，核心路由为：

- `/login`
- `/register`
- `/workspaces`
- `/workspaces/:workspaceId/documents`
- `/documents/:documentId`

### 4.1 登录与注册页

登录页提供：

- 邮箱输入。
- 密码输入。
- 登录按钮。
- 跳转注册入口。

注册页提供：

- 邮箱输入。
- 昵称输入。
- 密码输入。
- 确认密码输入。
- 注册按钮。

登录成功后进入 `/workspaces`。

### 4.2 工作区列表页

工作区列表页展示当前用户可访问的工作区。

主要功能：

- 查看工作区列表。
- 创建新工作区。
- 进入某个工作区的文档列表。
- 显示当前用户在工作区内的角色。

### 4.3 文档列表页

文档列表页展示某个工作区内的 Markdown 文档。

主要功能：

- 查看文档标题、更新时间、创建人。
- 新建 Markdown 文档。
- 重命名文档。
- 删除文档。
- 打开文档编辑页。

### 4.4 文档编辑页

文档编辑页是 V1 的核心页面，采用简洁的在线文档布局。

页面区域：

- 顶部工具栏：文档标题、保存状态、在线成员、当前用户菜单。
- 中央编辑区：所见即所得 Markdown 编辑器。
- 底部状态栏：连接状态、最后保存时间。

编辑体验：

- 用户直接在渲染后的文档上编辑内容。
- 支持标题、段落、列表、引用、加粗、斜体、行内代码、代码块、链接和表格。
- 不提供“点击预览”动作，也不做左右分屏源码预览。
- `viewer` 进入页面时编辑器只读。
- 多人进入同一文档时，内容实时同步，并展示在线成员。

## 5. 技术架构

### 5.1 技术栈

前端：

- `React 18`
- `TypeScript`
- `Vite`
- `React Router`
- `TanStack Query`
- `Zustand`
- `Ant Design`
- `Tiptap`
- `Yjs`

后端：

- `NestJS`
- `Prisma`
- `PostgreSQL`

实时协作：

- `Hocuspocus`
- `Yjs`

文件存储：

- 服务端本地目录或 Docker 挂载目录。
- 每篇文档保存为一个真实 `.md` 文件。

部署：

- `Docker Compose`

### 5.2 服务拆分

V1 拆成 3 个服务：

- `web-app`：前端单页应用，负责登录、工作区、文档列表和编辑器界面。
- `api-service`：后端 REST API，负责认证、权限、工作区、成员、文档元信息和文件读写。
- `collab-service`：实时协作服务，负责 Yjs 文档同步、在线状态和协同状态持久化触发。

V1 不需要：

- `worker-service`
- `BullMQ`
- `Redis`
- `MinIO`

如果后续需要异步任务、缓存、附件或大规模 presence，再单独引入。

## 6. 数据模型

V1 数据库只保存用户、工作区、成员关系和文档元信息。文档正文以 `.md` 文件形式保存，不直接存入数据库。

### 6.1 User

字段建议：

- `id`
- `email`
- `password_hash`
- `display_name`
- `created_at`
- `updated_at`

约束：

- `email` 全局唯一。
- 密码只保存 hash，不保存明文。

### 6.2 Workspace

字段建议：

- `id`
- `name`
- `owner_id`
- `created_at`
- `updated_at`

约束：

- `owner_id` 指向 `User.id`。

### 6.3 WorkspaceMember

字段建议：

- `id`
- `workspace_id`
- `user_id`
- `role`
- `created_at`
- `updated_at`

角色枚举：

- `owner`
- `editor`
- `viewer`

约束：

- 同一个用户在同一个工作区内只能有一条成员记录。
- `role` 决定该用户对工作区和文档的操作权限。

### 6.4 Document

字段建议：

- `id`
- `workspace_id`
- `title`
- `file_path`
- `created_by`
- `created_at`
- `updated_at`

说明：

- `file_path` 保存服务端 `.md` 文件的相对路径。
- `title` 用于页面展示，默认也用于生成初始文件名。
- 文档正文不进入 `Document` 表。

## 7. Markdown 文件存储方案

### 7.1 文件目录

服务端统一管理 Markdown 文件，建议目录结构：

```text
storage/
  workspaces/
    {workspaceId}/
      documents/
        {documentId}.md
```

示例：

```text
storage/workspaces/ws_001/documents/doc_001.md
```

数据库中的 `Document.file_path` 保存相对路径：

```text
workspaces/ws_001/documents/doc_001.md
```

### 7.2 创建文档

创建文档流程：

1. 用户在文档列表页点击新建文档。
2. 前端提交文档标题。
3. `api-service` 校验用户是否是该工作区的 `owner` 或 `editor`。
4. 后端创建 `Document` 记录。
5. 后端在服务端存储目录创建对应 `.md` 文件。
6. 初始文件内容可以为空，也可以写入一级标题。

初始内容示例：

```md
# 新文档
```

### 7.3 打开文档

打开文档流程：

1. 前端进入 `/documents/:documentId`。
2. 前端请求文档元信息。
3. 后端校验用户是否有访问权限。
4. 前端建立协同连接。
5. `collab-service` 根据 `documentId` 找到 `.md` 文件。
6. 如果当前协同房间尚未初始化，则读取 `.md` 文件并转换为编辑器初始内容。
7. 用户进入编辑器后看到所见即所得内容。

### 7.4 保存文档

保存文档不由每个客户端直接写文件，而由服务端统一处理。

保存规则：

- 多人编辑时，同步的是 Yjs 协同状态。
- `collab-service` 监听文档变化。
- 文档变化后 debounce 保存，例如 2 到 5 秒内没有新的变化再写入文件。
- 保存时将编辑器内容序列化为 Markdown。
- 写文件时使用同一文档维度的串行写入，避免并发覆盖。
- 保存成功后更新 `Document.updated_at`。

### 7.5 文件写入安全

必须遵守：

- 不允许客户端传入任意文件路径。
- 文件路径只能由后端根据 `workspaceId` 和 `documentId` 生成。
- 删除文档时先校验权限，再删除数据库记录和对应 `.md` 文件。
- 写入文件时使用临时文件加原子替换，降低写入中断导致文件损坏的概率。

推荐写入流程：

1. 将 Markdown 内容写入 `{documentId}.md.tmp`。
2. 写入完成后替换 `{documentId}.md`。
3. 替换成功后删除临时文件。

## 8. 实时协同方案

### 8.1 协同房间

每篇文档对应一个协同房间：

```text
document:{documentId}
```

前端连接 `collab-service` 时携带：

- `documentId`
- 用户身份 token

服务端连接校验：

- token 是否有效。
- 用户是否存在。
- 用户是否属于该文档所在工作区。
- 用户角色是否允许查看。
- 如果用户是 `viewer`，协同连接只能以只读方式进入。

### 8.2 协同数据

编辑器使用 Tiptap + Yjs。

基本原则：

- 编辑器状态不复制到全局 store。
- 协同正文以 Yjs 文档为准。
- 页面 UI 状态可以放在 Zustand，例如侧栏开关、当前连接状态、当前在线成员。
- 服务端 `.md` 文件是持久化结果，不是多人协同过程中的直接写入对象。

### 8.3 在线状态

在线状态使用 Yjs awareness。

在线成员展示信息：

- 用户 id。
- 用户昵称。
- 用户颜色。
- 当前连接状态。

V1 只要求展示当前在线成员，不要求显示精确光标位置。如果编辑器集成成本可控，可以展示协同光标；如果实现复杂，则放到后续版本。

### 8.4 断线重连

断线重连要求：

- 前端显示连接中、已连接、已断开状态。
- 短暂断线后自动重连。
- 重连后从协同服务同步最新状态。
- 保存状态不能误报为已保存。

## 9. API 设计

### 9.1 Auth

```http
POST /auth/register
POST /auth/login
GET /auth/me
POST /auth/logout
```

`POST /auth/register` 请求体：

```json
{
  "email": "user@example.com",
  "displayName": "User",
  "password": "password"
}
```

`POST /auth/login` 请求体：

```json
{
  "email": "user@example.com",
  "password": "password"
}
```

登录成功返回：

```json
{
  "accessToken": "jwt-token",
  "user": {
    "id": "user_id",
    "email": "user@example.com",
    "displayName": "User"
  }
}
```

### 9.2 Workspace

```http
GET /workspaces
POST /workspaces
GET /workspaces/:workspaceId
GET /workspaces/:workspaceId/members
POST /workspaces/:workspaceId/members
DELETE /workspaces/:workspaceId/members/:memberId
```

V1 可以先只做最小成员管理：

- 创建工作区时，当前用户自动成为 `owner`。
- `owner` 可以添加已注册用户为 `editor` 或 `viewer`。
- 暂不做邮件邀请链接。

### 9.3 Document

```http
GET /workspaces/:workspaceId/documents
POST /workspaces/:workspaceId/documents
GET /documents/:documentId
PATCH /documents/:documentId
DELETE /documents/:documentId
```

`POST /workspaces/:workspaceId/documents` 请求体：

```json
{
  "title": "新文档"
}
```

`PATCH /documents/:documentId` 请求体：

```json
{
  "title": "更新后的标题"
}
```

说明：

- REST API 管文档元信息。
- 文档正文由协同服务加载和保存。
- V1 不提供独立的正文保存 API，避免和协同保存路径冲突。

## 10. 前端实现约定

### 10.1 目录结构

建议目录：

```text
src/
  app/
    router/
    providers/
    layouts/
  features/
    auth/
    workspace/
    document/
    editor/
  entities/
    user/
    workspace/
    document/
  shared/
    api/
    components/
    hooks/
    styles/
    utils/
```

### 10.2 状态分工

`TanStack Query` 负责：

- 当前用户信息。
- 工作区列表。
- 文档列表。
- 文档元信息。
- 成员列表。
- 服务端请求缓存和失效。

`Zustand` 负责：

- 当前编辑页 UI 状态。
- 连接状态。
- 在线成员展示数据。
- 本地弹窗和面板状态。

Tiptap/Yjs 负责：

- 编辑器正文。
- 协同同步。
- undo/redo 编辑历史。

不要把编辑器正文复制一份到 Zustand 或 React state。

### 10.3 编辑器能力

V1 编辑器必须支持：

- 一级到三级标题。
- 段落。
- 有序列表。
- 无序列表。
- 引用块。
- 加粗。
- 斜体。
- 行内代码。
- 代码块。
- 链接。
- 表格。

V1 可以暂不支持：

- 图片上传。
- Mermaid。
- 数学公式。
- 目录自动生成。
- 复杂嵌套块。

### 10.4 Markdown 转换

编辑器内部可以使用 Tiptap 的结构化文档模型，但持久化结果必须是 Markdown。

转换要求：

- 打开文档时：Markdown 转编辑器内容。
- 保存文档时：编辑器内容转 Markdown。
- 常用 Markdown 结构往返转换后不能明显丢失。

需要重点验证：

- 标题层级。
- 列表嵌套。
- 代码块语言。
- 表格。
- 链接。
- 引用块。

## 11. 后端实现约定

### 11.1 认证

认证采用 JWT。

要求：

- 密码使用 bcrypt 或 argon2 hash。
- 登录成功返回 access token。
- 前端请求 API 时携带 token。
- 后端通过 guard 校验 token。
- 协同服务连接时也必须校验 token。

### 11.2 权限校验

权限校验统一封装，不在各个 controller 中散落实现。

建议提供方法：

- `canViewDocument(userId, documentId)`
- `canEditDocument(userId, documentId)`
- `canManageWorkspace(userId, workspaceId)`

所有文档 API 和协同连接都必须经过权限校验。

### 11.3 文件服务

文件服务负责：

- 根据 `workspaceId` 和 `documentId` 生成文件路径。
- 创建初始 `.md` 文件。
- 读取 `.md` 文件。
- 原子写入 `.md` 文件。
- 删除 `.md` 文件。

文件服务禁止：

- 使用用户传入路径直接读写文件。
- 允许 `../` 这类路径穿越。
- 将正文内容写入数据库替代 `.md` 文件。

### 11.4 协同服务

协同服务负责：

- 建立文档房间。
- 初始化 Yjs 文档。
- 同步多人编辑。
- 维护 awareness 在线状态。
- 将 Yjs 文档序列化为 Markdown。
- debounce 保存到 `.md` 文件。

协同服务不负责：

- 用户注册登录。
- 工作区创建。
- 文档元信息 CRUD。

## 12. 部署方案

Docker Compose 至少包含：

- `web-app`
- `api-service`
- `collab-service`
- `postgres`

建议挂载目录：

```text
./data/postgres
./data/storage
```

环境变量：

```text
DATABASE_URL=postgresql://...
JWT_SECRET=...
STORAGE_ROOT=/app/storage
COLLAB_WS_URL=ws://collab-service:1234
```

本地开发端口建议：

- `web-app`: `5173`
- `api-service`: `3000`
- `collab-service`: `1234`
- `postgres`: `5432`

## 13. 实施计划

### 第 1 阶段：项目骨架与基础设施

- 初始化前后端项目。
- 配置 TypeScript、lint、format、环境变量。
- 配置 Docker Compose。
- 接入 PostgreSQL 和 Prisma。
- 建立基础数据模型和迁移。

### 第 2 阶段：认证与工作区

- 完成注册、登录、当前用户接口。
- 完成 JWT 鉴权。
- 完成工作区创建和列表。
- 完成工作区成员关系。
- 完成前端登录态路由保护。

### 第 3 阶段：文档管理与文件存储

- 完成文档列表。
- 完成新建、重命名、删除文档。
- 创建文档时生成 `.md` 文件。
- 打开文档时读取文档元信息。
- 完成文件服务的路径生成、读取、写入和删除。

### 第 4 阶段：所见即所得 Markdown 编辑器

- 接入 Tiptap。
- 配置基础 Markdown 编辑能力。
- 接入 Markdown 到编辑器内容的转换。
- 接入编辑器内容到 Markdown 的转换。
- 完成只读模式。

### 第 5 阶段：实时协同

- 接入 Yjs。
- 接入 Hocuspocus。
- 按 `documentId` 建立协同房间。
- 完成协同连接鉴权。
- 完成多人实时同步。
- 完成在线成员展示。
- 完成断线重连状态展示。

### 第 6 阶段：自动保存与验收

- 协同服务监听文档变化。
- debounce 保存 Markdown 到 `.md` 文件。
- 更新文档 `updated_at`。
- 验证服务重启后的内容恢复。
- 补齐权限测试和协同测试。
- 准备演示数据和部署说明。

## 14. 测试与验收标准

### 14.1 功能测试

必须覆盖：

- 用户注册成功。
- 用户登录成功。
- 未登录访问工作区或文档页面会跳转登录页。
- 登录用户可以创建工作区。
- 工作区 owner 可以添加成员。
- 用户只能看到自己所属工作区。
- editor 可以创建和编辑文档。
- viewer 可以打开文档但不能编辑。
- 创建文档后服务端生成 `.md` 文件。
- 重命名文档后列表和编辑页标题更新。
- 删除文档后列表中不再出现，服务端文件同步删除。

### 14.2 编辑器测试

必须覆盖：

- 标题编辑。
- 段落编辑。
- 加粗和斜体。
- 有序列表和无序列表。
- 引用块。
- 行内代码。
- 代码块。
- 链接。
- 表格。
- 刷新页面后内容不丢失。

### 14.3 协同测试

必须覆盖：

- 两个 editor 同时打开同一文档，可以实时看到对方修改。
- 在线成员列表正确显示。
- 一个用户断线后，其他用户仍可继续编辑。
- 断线用户重连后可以同步最新内容。
- viewer 进入同一文档时能看到更新，但不能修改。
- 多人连续编辑时，最终 `.md` 文件内容完整，不出现截断或覆盖。

### 14.4 持久化测试

必须覆盖：

- 编辑后等待自动保存，服务端 `.md` 文件内容更新。
- 重启 `api-service` 后文档元信息仍存在。
- 重启 `collab-service` 后可以从 `.md` 文件恢复正文。
- 重启全部服务后，用户、工作区、文档列表和正文内容都可恢复。

### 14.5 验收演示流程

验收演示固定流程：

1. 注册两个用户。
2. 用户 A 创建工作区。
3. 用户 A 添加用户 B 为 editor。
4. 用户 A 创建 Markdown 文档。
5. 用户 A 和用户 B 同时进入文档。
6. 两人共同编辑标题、列表、代码块和表格。
7. 页面显示两名在线成员。
8. 等待自动保存完成。
9. 刷新页面，内容仍然存在。
10. 重启服务后再次打开文档，内容仍然存在。

## 15. 后续扩展方向

V1 完成后，可以按实际需求选择扩展：

- 历史版本：基于快照表或 Git 实现。
- 评论审阅：在文档位置上绑定评论。
- Agent 能力：基于当前 Markdown 内容做总结、润色或生成建议。
- Git 同步：将工作区文档映射到 Git 仓库。
- 附件和图片：引入对象存储。
- Markdown 源码模式：提供源码编辑和所见即所得切换。
- 导出能力：导出 HTML、PDF 或压缩包。
- 企业登录：接入 SSO 或 LDAP。

后续扩展不能影响 V1 的核心原则：文档正文以 Markdown 文件为最终持久化结果，协同编辑通过 Yjs 统一管理实时状态。
