# 02-前端 web-app

## 目标

搭建 `apps/web-app` 前端应用骨架，完成基础路由、Provider、布局和占位页面。

## 工作清单

### 1. 应用创建与基础依赖

- [ ] 创建 `apps/web-app`
- [ ] 初始化 Vite React TypeScript 项目
- [ ] 配置 `apps/web-app/package.json`
- [ ] 添加 `dev` 脚本
- [ ] 添加 `build` 脚本
- [ ] 添加 `typecheck` 脚本
- [ ] 安装 React Router
- [ ] 安装 TanStack Query
- [ ] 安装 Zustand
- [ ] 安装 Ant Design

### 2. 应用入口与全局配置

- [ ] 配置 `main.tsx`
- [ ] 配置根组件 `App`
- [ ] 配置 React Router 路由入口
- [ ] 配置 TanStack Query `QueryClientProvider`
- [ ] 配置 Ant Design 全局 Provider
- [ ] 配置全局样式入口

### 3. 基础 Layout

- [ ] 创建基础页面布局组件
- [ ] 配置顶部导航或标题区域
- [ ] 配置页面内容容器
- [ ] 配置路由占位渲染区域
- [ ] 登录和注册页面不强制套用业务 Layout

### 4. 基础路由

- [ ] 配置 `/login` 路由
- [ ] 配置 `/register` 路由
- [ ] 配置 `/workspaces` 路由
- [ ] 配置 `/workspaces/:workspaceId/documents` 路由
- [ ] 配置 `/documents/:documentId` 路由
- [ ] 配置默认路由跳转到 `/login` 或 `/workspaces`

### 5. 占位页面

- [ ] 创建登录页
- [ ] 登录页显示邮箱、密码、登录按钮
- [ ] 创建注册页
- [ ] 注册页显示邮箱、昵称、密码、注册按钮
- [ ] 创建工作区页
- [ ] 工作区页显示占位列表
- [ ] 创建文档列表页
- [ ] 文档列表页显示占位列表
- [ ] 文档列表页显示新建按钮
- [ ] 创建文档编辑页
- [ ] 文档编辑页显示工具栏
- [ ] 文档编辑页显示编辑区
- [ ] 文档编辑页显示连接状态占位

### 6. 验收确认

- [ ] 确认 `web-app` 可以本地启动
- [ ] 确认 `http://localhost:5173` 可以访问
- [ ] 确认 `/login` 可以访问
- [ ] 确认 `/register` 可以访问
- [ ] 确认 `/workspaces` 可以访问
- [ ] 确认 `/workspaces/:workspaceId/documents` 可以访问
- [ ] 确认 `/documents/:documentId` 可以访问
- [ ] 确认前端暂不接真实登录、文档和协同业务

## 验收标准

- `web-app` 可以本地启动。
- 浏览器可以访问 `http://localhost:5173`。
- 五个基础路由都能正常渲染占位页面。
- 前端暂不接真实登录、文档和协同业务。

