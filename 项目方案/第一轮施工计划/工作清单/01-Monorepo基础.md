# 01-Monorepo 基础

## 目标

创建 `pnpm` monorepo 基础结构，为前端、后端、协同服务和共享包提供统一工程入口。

## 工作清单

- [ ✅️ ] 创建 `package.json`
- [ ✅️ ] 创建 `pnpm-workspace.yaml`
- [ ✅️ ] 创建 `.gitignore`
- [ ✅️ ] 创建 `.env.example`
- [ ✅️ ] 创建 `apps/`
- [ ✅️ ] 创建 `packages/`
- [ ✅️ ] 创建 `packages/shared`
- [ ✅️ ] 创建 `data/storage`
- [ ✅️ ] 添加 `data/storage/.gitkeep`
- [ ✅️ ] 配置根脚本：`dev`
- [ ✅️ ] 配置根脚本：`build`
- [ ✅️ ] 配置根脚本：`lint`
- [ ✅️ ] 配置根脚本：`typecheck`

## 验收标准

- 根目录存在 monorepo 必要配置文件。
- `pnpm-workspace.yaml` 覆盖 `apps/*` 和 `packages/*`。
- `data/storage` 目录存在，实际存储内容不会提交 Git。
- 根目录脚本能作为后续统一命令入口。

