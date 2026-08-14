现在进入 AI Drama Studio 的第二阶段开发。

第一阶段项目骨架已经完成。

请不要重新初始化项目，不要修改现有 Monorepo 架构，不要更换技术栈。

当前技术栈：

Web:
Nuxt 3 + Vue 3 + TypeScript + Tailwind + Pinia

Mobile:
Ionic Vue + Capacitor

Desktop:
Tauri 2 + Vue

Backend:
NestJS

Database:
PostgreSQL + Prisma

Monorepo:
pnpm + Turborepo

==================================================
本阶段目标
==================================================

只实现：

# Project 项目管理 + Project Workspace

Project 是整个 AI Drama Studio 的核心根实体。

未来：

Project
├── World
├── Characters
├── Locations
├── Episodes
├── Storyboards
├── Assets
└── GenerationTasks

本阶段暂时不要实现这些子系统。

==================================================
一、数据库 Project 模型
==================================================

检查现有 Prisma Project。

调整为至少包含：

id
name
description
genre
cover
status
currentStep
createdAt
updatedAt

status：

DRAFT
IN_PROGRESS
COMPLETED
ARCHIVED

currentStep：

WORLD
CHARACTERS
LOCATIONS
EPISODES
SCRIPT
STORYBOARD
IMAGES
VIDEOS
VOICES
RENDER

如果 Prisma 当前使用 enum，请使用 enum。

不要删除现有 Project 数据。

如有必要，创建 migration。

==================================================
二、Project API
==================================================

保持现有 API：

GET /projects
GET /projects/:id
POST /projects
PATCH /projects/:id
DELETE /projects/:id

POST /projects 支持：

name
description
genre

创建项目时：

status = DRAFT
currentStep = WORLD

PATCH 支持：

name
description
genre
cover
status
currentStep

==================================================
三、共享类型
==================================================

更新：

packages/types

增加：

ProjectStatus
ProjectStep
Project

确保 Web / Mobile / Desktop 使用共享类型。

禁止在三个客户端重复定义 ProjectStatus 和 ProjectStep。

==================================================
四、Project API Client
==================================================

检查：

packages/api-client

确保提供：

getProjects()
getProject(id)
createProject(data)
updateProject(id, data)
deleteProject(id)

不要让页面直接调用 fetch。

==================================================
五、Web 项目列表
==================================================

页面：

/projects

设计成真正的产品界面。

顶部：

我的漫剧

右侧：

+ 新建漫剧

项目使用 Card 展示。

每个项目显示：

- 封面
- 名称
- 简介
- 类型
- 当前状态
- 制作进度
- 更新时间
- 继续制作按钮

制作进度根据 currentStep 计算。

10个步骤：

WORLD
CHARACTERS
LOCATIONS
EPISODES
SCRIPT
STORYBOARD
IMAGES
VIDEOS
VOICES
RENDER

例如：

WORLD = 10%
CHARACTERS = 20%
...
RENDER = 100%

如果 status = COMPLETED：

显示 100%。

==================================================
六、创建 Project
==================================================

点击：

+ 新建漫剧

打开 Modal 或 Drawer。

字段：

漫剧名称
简介
类型

类型暂时支持：

科幻
修仙
赛博朋克
都市
爱情
悬疑
玄幻
其他

创建成功：

自动进入：

/projects/:id

==================================================
七、Project Workspace
==================================================

页面：

/projects/:id

这是整个产品未来最重要的页面。

Desktop：

左侧 Sidebar：

项目概览
世界观
人物
场景
剧集
剧本
分镜
图片
视频
配音
成片
素材

顶部：

返回项目
项目名称
当前步骤
设置

主区域：

显示：

项目封面
项目名称
项目简介
制作进度

创作流程：

01 世界观
02 人物
03 场景
04 剧集
05 剧本
06 分镜
07 图片
08 视频
09 配音
10 成片

当前阶段所有步骤页面都可以是 Placeholder。

不要实现真正业务。

==================================================
八、Continue Production
==================================================

Project Card：

[继续制作]

点击以后：

根据 currentStep 自动跳转。

例如：

currentStep = WORLD

跳转：

/projects/:id/world

currentStep = CHARACTERS

跳转：

/projects/:id/characters

依此类推。

==================================================
九、Step Navigation
==================================================

点击 Sidebar：

世界观
人物
场景
剧集
剧本
分镜
图片
视频
配音
成片

都可以进入对应页面。

但是当前阶段这些页面只需要显示：

页面标题
页面说明
当前步骤状态
“即将开始开发”占位内容

不要实现真正业务。

==================================================
十、Mobile
==================================================

Mobile 不要直接复制 Desktop Sidebar。

项目详情页面使用 Mobile Layout。

顶部：

返回
项目名称

内容：

项目封面
项目名称
项目简介
制作进度

然后显示：

创作流程列表。

每个 Step：

名称
状态
进度

底部 Navigation：

首页
项目
任务
素材
我的

本阶段任务 / 素材 / 我的可以是 Placeholder。

==================================================
十一、Responsive
==================================================

必须支持：

Mobile < 768px

Tablet 768px - 1199px

Desktop >= 1200px

Desktop：

Sidebar

Tablet：

折叠 Sidebar

Mobile：

Top Navigation + Bottom Navigation

不要简单缩小 Desktop。

==================================================
十二、状态
==================================================

Project 状态显示：

DRAFT
草稿

IN_PROGRESS
制作中

COMPLETED
已完成

ARCHIVED
已归档

使用共享类型。

==================================================
十三、删除项目
==================================================

删除操作必须二次确认。

例如：

确定删除《星河碰撞》？

删除后项目数据将无法恢复。

确认删除
取消

当前阶段可以直接删除。

未来再实现软删除。

==================================================
十四、Loading / Empty / Error
==================================================

所有页面必须考虑：

Loading

Empty

Error

Success

例如：

没有项目：

还没有创建漫剧

[创建第一部漫剧]

==================================================
十五、不要做的事情
==================================================

本阶段禁止：

AI生成
LLM
Agent
世界观编辑
人物编辑
场景编辑
剧集编辑
剧本生成
分镜生成
图片生成
视频生成
TTS
Redis
BullMQ
登录
支付
权限系统
文件上传
S3

不要修改技术栈。

不要安装大型 UI 框架。

不要重新创建 Monorepo。

==================================================
十六、代码质量
==================================================

要求：

- TypeScript strict
- 不使用 any
- 页面不要直接请求 API
- API 请求统一通过 packages/api-client
- 业务状态统一通过 Pinia / composables
- 可复用组件放 components
- 不复制共享类型
- 不重复实现 Project API

==================================================
十七、完成标准
==================================================

完成后：

Web：

/projects

可以：

创建项目
查看项目
编辑项目
删除项目
进入项目

/projects/:id

可以：

查看项目
查看制作进度
查看创作流程
点击步骤
点击继续制作

Mobile：

可以：

查看项目
创建项目
查看项目详情
查看制作流程

Desktop：

保持现有 App Shell。

==================================================
十八、完成后检查
==================================================

运行：

pnpm lint

pnpm typecheck

修复所有错误。

然后告诉我：

1. 修改了哪些文件
2. 修改了哪些数据库字段
3. 是否执行 migration
4. 新增了哪些 API
5. Web 完成了什么
6. Mobile 完成了什么
7. Desktop 完成了什么
8. 是否存在 TypeScript 错误
9. 是否存在 lint 错误

不要继续开发第三阶段。