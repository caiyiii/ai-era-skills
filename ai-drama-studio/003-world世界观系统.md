现在进入 AI Drama Studio 第三阶段。

第二阶段已经完成：

Project 项目管理 + Project Workspace。

当前：

pnpm lint
pnpm typecheck

均已通过。

不要重新初始化项目。

不要修改技术栈。

不要修改 Monorepo 架构。

本阶段目标：

# World 世界观系统

注意：

本阶段只实现：

World 数据模型
World CRUD
Civilization
World History
Faction
World Location
Power System
World 编辑器 UI

暂时不要接入任何真实 LLM / AI API。

==================================================
一、产品目标
==================================================

用户创建一个漫剧 Project 后：

Project
→ World

World 是整个漫剧的世界观基础。

未来：

World
↓
Characters
↓
Episodes
↓
Scenes
↓
Shots
↓
Images
↓
Videos

人物、剧集、场景等后续系统都将依赖 World。

==================================================
二、World 数据模型
==================================================

新增：

World

字段至少：

id
projectId
title
summary
cosmicBackground
coreConflict
createdAt
updatedAt

Project 与 World：

一对一。

一个 Project 只有一个主 World。

==================================================
三、Civilization
==================================================

新增：

Civilization

字段：

id
worldId
name
description
origin
philosophy
society
culture
technology
createdAt
updatedAt

关系：

World
└── Civilization[]

==================================================
四、WorldHistory
==================================================

新增：

WorldHistory

字段：

id
worldId
title
description
order
createdAt
updatedAt

关系：

World
└── WorldHistory[]

order 用于控制时间线顺序。

==================================================
五、Faction
==================================================

新增：

Faction

字段：

id
worldId
civilizationId
name
description
type
createdAt
updatedAt

civilizationId 可以为空。

因为某些势力可能跨文明。

==================================================
六、WorldLocation
==================================================

新增：

WorldLocation

字段：

id
worldId
name
description
type
civilizationId
createdAt
updatedAt

civilizationId 可以为空。

==================================================
七、PowerSystem
==================================================

新增：

PowerSystem

字段：

id
worldId
name
description
rules
levels
createdAt
updatedAt

rules 和 levels 可以使用 JSON。

原因：

不同世界的能力体系差异非常大。

==================================================
八、Prisma
==================================================

更新：

prisma/schema.prisma

创建 migration。

不要删除现有 Project 数据。

World 与 Project：

一对一。

其他全部是一对多。

==================================================
九、API
==================================================

创建 World API：

GET
/projects/:projectId/world

POST
/projects/:projectId/world

PATCH
/projects/:projectId/world

DELETE
/projects/:projectId/world

Civilization：

GET
/projects/:projectId/world/civilizations

POST
/projects/:projectId/world/civilizations

PATCH
/projects/:projectId/world/civilizations/:id

DELETE
/projects/:projectId/world/civilizations/:id

History：

GET
/projects/:projectId/world/history

POST
/projects/:projectId/world/history

PATCH
/projects/:projectId/world/history/:id

DELETE
/projects/:projectId/world/history/:id

Faction：

GET
/projects/:projectId/world/factions

POST
/projects/:projectId/world/factions

PATCH
/projects/:projectId/world/factions/:id

DELETE
/projects/:projectId/world/factions/:id

Location：

GET
/projects/:projectId/world/locations

POST
/projects/:projectId/world/locations

PATCH
/projects/:projectId/world/locations/:id

DELETE
/projects/:projectId/world/locations/:id

PowerSystem：

GET
/projects/:projectId/world/power-systems

POST
/projects/:projectId/world/power-systems

PATCH
/projects/:projectId/world/power-systems/:id

DELETE
/projects/:projectId/world/power-systems/:id

==================================================
十、Shared Types
==================================================

packages/types

增加：

World
Civilization
WorldHistory
Faction
WorldLocation
PowerSystem

不要在 Web / Mobile / Desktop 重复定义。

==================================================
十一、World API Client
==================================================

packages/api-client

创建 World API 方法。

例如：

getWorld(projectId)
createWorld(projectId,data)
updateWorld(projectId,data)
deleteWorld(projectId)

getCivilizations(projectId)
createCivilization(projectId,data)
updateCivilization(projectId,id,data)
deleteCivilization(projectId,id)

其他实体同样处理。

页面禁止直接 fetch。

==================================================
十二、Web World 页面
==================================================

页面：

/projects/:id/world

做成真正的世界观工作台。

Desktop：

左侧：

世界概览
宇宙背景
文明体系
能力体系
历史
势力
地点
核心冲突

右侧：

当前编辑内容。

==================================================
十三、World Overview
==================================================

显示：

世界名称
世界简介
宇宙背景
核心冲突

每个字段都可以：

编辑
保存

==================================================
十四、文明体系
==================================================

显示 Civilization Card。

例如：

修仙文明
赛博科技文明

Card 显示：

名称
简介
起源
哲学
社会
文化
科技

支持：

新增
编辑
删除

==================================================
十五、历史
==================================================

使用 Timeline UI。

例如：

星系碰撞
↓
文明首次接触
↓
第一次战争
↓
文明融合

支持：

新增事件
编辑
删除
调整顺序

==================================================
十六、势力
==================================================

Card/List。

显示：

名称
所属文明
类型
简介

支持：

新增
编辑
删除

==================================================
十七、地点
==================================================

显示：

名称
类型
所属文明
简介

支持：

新增
编辑
删除

==================================================
十八、能力体系
==================================================

PowerSystem：

名称
简介
规则
等级

规则和等级可以编辑 JSON。

但是 UI 不要直接要求用户输入 JSON。

应该提供简单的结构化表单。

例如：

等级：

炼气
筑基
金丹
元婴

最终再转换成 JSON 保存。

==================================================
十九、Mobile
==================================================

Mobile 不要复制 Desktop Sidebar。

使用：

World
概览
文明
历史
势力
地点
能力体系

使用 Mobile List / Card。

每个模块可以进入详情。

支持基本：

查看
创建
编辑
删除

==================================================
二十、Desktop
==================================================

Desktop 复用现有 App Shell。

尽可能复用：

packages/types
packages/api-client
packages/core

不要重复实现业务逻辑。

==================================================
二十一、AI按钮
==================================================

本阶段可以在 UI 中预留：

“AI生成”

按钮。

但是：

点击后暂时显示：

“AI世界观生成将在下一阶段开放。”

不要调用任何 AI API。

==================================================
二十二、Demo数据
==================================================

不要自动向数据库写入 Demo 数据。

可以提供：

seed script

但不要默认执行。

可准备一个 Demo：

Project：

星河碰撞

World：

星河碰撞

Civilizations：

修仙文明
赛博科技文明

Core Conflict：

两个星系因宇宙碰撞发生文明接触，
两个文明发现彼此选择了完全不同的生存与发展道路。

但不要在应用启动时自动创建。

==================================================
二十三、错误处理
==================================================

所有 CRUD：

Loading
Empty
Error
Success

必须处理。

删除：

二次确认。

==================================================
二十四、禁止
==================================================

禁止：

LLM API
OpenAI API
Claude API
Gemini API
DeepSeek API
任何 AI Provider
Redis
BullMQ
Worker
Agent
RAG
人物系统
剧集系统
剧本系统
分镜系统
图片生成
视频生成
TTS

不要继续开发第四阶段。

==================================================
二十五、完成标准
==================================================

Web：

可以创建 Project World。

可以编辑：

世界名称
简介
宇宙背景
核心冲突

可以管理：

Civilizations
History
Factions
Locations
Power Systems

Mobile：

可以查看和编辑 World。

Desktop：

可以进入 World 页面。

API：

所有 CRUD API 正常。

Prisma：

migration 创建完成。

Shared Types：

已完成。

API Client：

已完成。

最后执行：

pnpm lint
pnpm typecheck

修复所有错误。

完成后告诉我：

1. 修改文件
2. 新增数据库模型
3. Migration
4. API
5. Shared Types
6. API Client
7. Web
8. Mobile
9. Desktop
10. lint
11. typecheck

不要继续开发 AI 生成功能。