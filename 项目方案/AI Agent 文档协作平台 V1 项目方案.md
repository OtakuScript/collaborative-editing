# AI Agent 文档协作平台 V1 项目方案

## Summary

目标是做一个面向研发团队的“文档协作平台 + Agent 辅助编辑”产品，而不是通用笔记工具。V1 聚焦技术方案、接口说明、发布说明、故障复盘四类研发文档，优先解决多人协作、版本沉淀、审阅流程和 Agent 建议修改。

技术栈固定为：
- 前端：`React 18 + TypeScript + Vite + React Router + TanStack Query + Zustand + Ant Design + Tiptap + Yjs`
- 后端：`NestJS + Prisma + PostgreSQL + Redis + BullMQ`
- 实时协作：`Hocuspocus`
- 文件存储：`MinIO`
- 部署：`Docker Compose`

V1 边界固定为：
- 做工作区、成员、文档、评论、版本、发布、Agent 建议
- 不做 Jira/Git/OpenAPI/数据库自动同步
- 不做多 Agent 自主协作
- 不做完整 RAG 知识库平台
- Agent 只能产出“建议”，不能直接覆盖正文

## Key Changes

### 产品与交互

前端采用单页应用，核心路由固定为：
- `/login`
- `/workspaces`
- `/workspaces/:workspaceId/documents`
- `/documents/:documentId`
- `/documents/:documentId/review`
- `/documents/:documentId/history`

文档详情页固定拆成 5 个区域：
- 顶部工具栏：标题、状态、发布、版本、成员在线状态
- 中央编辑区：Tiptap 编辑器 + Yjs 协同
- 右侧评论面板：评论、回复、正文定位
- 右侧 Agent 面板：发起任务、查看建议、接受或拒绝
- 底部状态栏：保存状态、连接状态、当前版本信息

V1 支持 4 类文档模板：
- 技术方案
- 接口说明
- 发布说明
- 故障复盘

V1 支持 5 类 Agent 动作：
- `draft_from_template`
- `polish_selection`
- `summarize_document`
- `check_term_consistency`
- `generate_release_notes`

Agent 输出固定为结构化建议列表，每条建议包含：
- 作用范围
- 修改前摘要
- 修改后摘要
- 原因说明
- 建议类型
- 接受/拒绝状态

### 系统架构与数据模型

系统拆成 4 个服务：
- `web-app`：负责编辑器、协同界面、评论、版本页、Agent 面板
- `api-service`：负责认证、权限、文档、评论、版本、建议、附件
- `collab-service`：负责实时协作和在线 presence
- `worker-service`：消费任务，调用大模型并回写建议结果

核心数据模型固定为：
- `User`
- `Workspace`
- `WorkspaceMember`
- `Document`
- `DocumentVersion`
- `Comment`
- `Suggestion`
- `AgentJob`
- `Attachment`

角色固定为：
- `owner`
- `editor`
- `commenter`
- `viewer`

内容存储规则固定为：
- 编辑态主存 `Tiptap JSON`
- 每次发布生成 `DocumentVersion`
- 发布版本同时持久化 `content_json`、`plain_text`、`markdown_export`
- 评论和建议都绑定到具体 `document_version_id`
- 接受建议后写入当前草稿，不直接改历史版本

### 公共接口与实现约定

REST API 固定为：
- `POST /auth/register`
- `POST /auth/login`
- `GET /workspaces/:id/documents`
- `POST /documents`
- `GET /documents/:id`
- `POST /documents/:id/versions/publish`
- `POST /documents/:id/comments`
- `POST /documents/:id/agent-jobs`
- `GET /agent-jobs/:id`
- `POST /suggestions/:id/accept`
- `POST /suggestions/:id/reject`

前端状态分工固定为：
- `TanStack Query` 管服务端数据、缓存、轮询、失效
- `Zustand` 管当前文档本地 UI 状态，如面板开关、选区、建议高亮、在线成员展示
- 编辑器状态只保留在 Tiptap/Yjs 内，不额外复制到全局 store

前端目录建议固定为：
- `src/app`：路由、providers、全局布局
- `src/features/document`：编辑器、评论、版本、Agent 面板
- `src/features/workspace`：工作区与文档列表
- `src/features/auth`：登录注册
- `src/entities`：基础类型与 API 封装
- `src/shared`：通用组件、hooks、工具函数、样式变量

### 实施顺序

第 1 周：
- 初始化 monorepo、认证、工作区、成员权限、数据库模型、Docker 环境

第 2 周：
- 搭建前端基础骨架、文档列表、文档详情页、模板创建、评论系统

第 3 周：
- 接入 Tiptap + Yjs + Hocuspocus，实现实时协同、在线状态、草稿保存

第 4 周：
- 完成版本发布、历史记录、版本预览、Markdown 导出

第 5 周：
- 接入 Agent worker，完成建议列表、接受/拒绝流转、摘要与润色

第 6 周：
- 补齐附件上传、权限补漏、E2E、演示数据、部署文档

## Test Plan

必须覆盖的场景：
- 用户注册、登录、创建工作区、邀请成员、角色权限生效
- 两个编辑者同时进入同一文档，正文同步、光标 presence、断线重连正常
- commenter 只能评论不能编辑，viewer 只能查看
- 发布版本后形成快照，后续草稿修改不影响历史版本
- 评论绑定到准确版本和正文位置，切换历史版本后仍可查看来源
- Agent 基于当前版本生成建议，接受后写入草稿，拒绝后正文不变
- 附件上传、图片插入、代码块、目录导航正常
- 单文档 50KB 到 300KB 内容下编辑流畅，发布和版本加载正常
- 服务重启后文档、评论、版本、建议、附件数据不丢失

验收演示流程固定为：
- 创建“技术方案”文档
- Agent 生成初稿
- 两名成员协同编辑
- reviewer 发评论
- Agent 做术语一致性检查并产出建议
- 编辑接受部分建议
- 发布版本并导出 Markdown

## Assumptions

默认假设固定为：
- 目标用户是内部研发团队，不是面向公众的 SaaS 首发版本
- 首版语言以中文为主，暂不做多语言
- 登录先用邮箱密码，V2 再补第三方登录或企业 SSO
- 模型层用兼容 OpenAI 的适配器封装，底层供应商可替换
- Agent 上下文只读取当前文档、当前模板和用户选定参考文档，不做全库检索
- V1 仅保证移动端可读，不提供完整移动编辑体验
- UI 以中后台效率风格为主，不追求营销型视觉表达
