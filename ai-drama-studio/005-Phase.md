你现在开始执行 AI Drama Studio Phase 5。

阶段名称：

Phase 5 — Character & Relationship System + AI Character Generation v1

目标：

在现有 AI Drama Studio 基础上，实现完整的「人物系统」，包括：

1. Character 人物管理
2. Character Relationship 人物关系
3. 人物与 World / Civilization / Faction 的关联
4. 人物 Web / Mobile / Desktop UI
5. AI Character Generation
6. AI Character Generation Preview / Apply
7. GenerationTask 接入 STRUCTURED_OUTPUT
8. 复用现有 AI Provider → Model → Capability 架构
9. 不接入 IMAGE / VIDEO / TTS 等真实生成
10. 为未来人物图片、角色声音、视频角色一致性预留扩展能力

非常重要：

不要修改现有 Monorepo 技术栈。
不要重建项目。
不要删除已有数据。
不要 reset 数据库。
不要修改历史 migration。
不要重新设计现有 World System。
不要破坏 Phase 4、4.5、4.6 已有功能。

继续使用：

Web:
Nuxt 3 + Vue 3 + Tailwind + Pinia + VueUse

Mobile:
Ionic Vue + Capacitor

Desktop:
Tauri 2 + Vue

API:
NestJS

Database:
PostgreSQL + Prisma

Monorepo:
pnpm + Turborepo

AI:
现有 AiProvider → AiModel → AiCapability → ProjectAiConfig → ProviderResolver 架构

当前端口：

Web: 3010
API: 3011

当前已有：

World CRUD
World Generation
AI Provider Management
AI Capability Architecture
Project AI Configuration
GenerationTask
星河碰撞 Demo Project

请在开始编码前：

1. 阅读现有：
   - prisma/schema.prisma
   - packages/types
   - packages/core
   - packages/api-client
   - packages/config
   - apps/api/src/modules/world
   - apps/api/src/modules/generation
   - apps/api/src/modules/ai
   - apps/web/pages/projects/[id]/world.vue
   - apps/web/pages/projects/[id]/settings.vue
   - 现有 Character 相关代码
2. 理解现有代码结构。
3. 尽量复用已有组件、Service、DTO、GenerationExecutor、ProviderResolver。
4. 不要重复实现已有能力。

==================================================
一、Phase 5 数据模型
==================================================

完善 Character 模型。

Character 至少支持：

id
projectId
worldId
civilizationId?
factionId?
name
alias?
gender?
age?
race?
identity?
role?
personality?
appearance?
background?
goal?
motivation?
conflict?
abilities?
relationships?
voiceProfile?
imageProfile?
metadata?
createdAt
updatedAt

注意：

不要把所有东西都设计成 JSON。

适合查询、关联、排序的数据应该使用独立字段。

复杂 AI 扩展字段可以使用 Json。

建议：

appearance Json?
personality Json?
abilities Json?
metadata Json?

但基础字段必须结构化。

==================================================
二、Character 与 World
==================================================

Character 属于 Project。

Character 可以关联：

World
Civilization
Faction

关系：

Project
  ↓
World
  ↓
Civilization / Faction
  ↓
Character

但是：

civilizationId 和 factionId 都应该允许为空。

因为未来可能存在：

- 无文明归属角色
- 中立角色
- 流浪者
- 外星生命
- 独立 AI
- 神秘角色

不要强制要求文明或势力。

==================================================
三、Character Relationship
==================================================

新增：

CharacterRelationship

字段至少：

id
projectId
fromCharacterId
toCharacterId
type
label?
description?
strength?
metadata?
createdAt
updatedAt

例如：

师徒
兄弟
恋人
朋友
敌人
宿敌
上下级
父子
母子
盟友
竞争者
陌生人
未知

不要把 relationship 存成 Character.relationships Json。

关系必须独立建表。

建议增加：

RelationshipType enum

至少：

FAMILY
FRIEND
LOVER
MASTER_STUDENT
ALLY
ENEMY
RIVAL
SUPERIOR_SUBORDINATE
ACQUAINTANCE
OTHER

同时允许：

label

用于自定义关系。

例如：

type = OTHER
label = "前世夫妻"

==================================================
四、人物图片相关字段
==================================================

当前阶段不要真正实现图片生成。

但是必须为未来 AI Image Generation 预留结构。

例如：

CharacterImageProfile

或者合理的 Json 结构。

至少预留：

visualStyle
referencePrompt
negativePrompt
identityPrompt
seed?
referenceAssetId?
consistencyConfig?

未来用于：

Character
 ↓
IMAGE capability
 ↓
角色定妆图
 ↓
IMAGE_TO_VIDEO
 ↓
视频中的角色一致性

本阶段不要调用 IMAGE Provider。

==================================================
五、人物声音相关字段
==================================================

同样不要实现真实 TTS。

但是可以预留：

voiceProfile Json

例如：

{
  "voiceId": null,
  "providerId": null,
  "modelId": null,
  "language": "zh-CN",
  "gender": "male",
  "style": "calm"
}

未来由：

TTS
VOICE_CLONE

能力使用。

==================================================
六、Character API
==================================================

实现：

GET
/projects/:projectId/characters

POST
/projects/:projectId/characters

GET
/projects/:projectId/characters/:id

PATCH
/projects/:projectId/characters/:id

DELETE
/projects/:projectId/characters/:id

支持：

分页
搜索 name
按 role 筛选
按 civilization 筛选
按 faction 筛选

如果当前项目规模还很小，可以先实现简单分页：

page
pageSize
search

但 API 结构必须方便未来扩展。

==================================================
七、Relationship API
==================================================

实现：

GET
/projects/:projectId/character-relationships

POST
/projects/:projectId/character-relationships

PATCH
/projects/:projectId/character-relationships/:id

DELETE
/projects/:projectId/character-relationships/:id

创建关系时必须验证：

fromCharacterId 属于当前 project
toCharacterId 属于当前 project

禁止跨项目关系。

禁止：

fromCharacterId === toCharacterId

除非未来明确支持自我关系，本阶段禁止。

==================================================
八、Shared Types
==================================================

packages/types 增加：

Character
CharacterInput
UpdateCharacterInput

CharacterRelationship
CharacterRelationshipInput
UpdateCharacterRelationshipInput

CharacterListQuery

RelationshipType

CharacterGenerationInput

CharacterGenerationResult

不要在 Web / Mobile / Desktop 重复定义这些类型。

==================================================
九、API Client
==================================================

packages/api-client 增加：

getCharacters()
getCharacter()
createCharacter()
updateCharacter()
deleteCharacter()

getCharacterRelationships()
createCharacterRelationship()
updateCharacterRelationship()
deleteCharacterRelationship()

以及：

createCharacterGeneration()
getCharacterGeneration()
applyCharacterGeneration()

页面不得直接 fetch。

==================================================
十、Character Generation
==================================================

新增：

POST

/projects/:projectId/generations/character

必须复用：

GenerationExecutor

AiService

ProviderResolver

Capability Resolver

本次 AI 能力：

STRUCTURED_OUTPUT

绝对不要：

自己读取 AI_API_KEY

自己读取 AI_MODEL

自己读取 AI_BASE_URL

自己选择 DeepSeek

必须：

resolveForCapability(
  projectId,
  AiCapability.STRUCTURED_OUTPUT
)

==================================================
十一、Character Generation 输入
==================================================

支持用户输入：

name
role
gender
age
civilizationId?
factionId?
personality
appearance
background
goal
motivation
conflict

以及：

prompt
style
detailLevel

例如：

{
  "prompt": "一个来自修仙文明的年轻天才",
  "style": "东方玄幻",
  "detailLevel": "high"
}

如果用户已经选择 Civilization / Faction：

把相关 World 数据注入 Prompt。

例如：

World：

星河碰撞

Civilization：

修仙文明

Faction：

太虚宗

AI 必须知道这些上下文。

==================================================
十二、Character Generation Prompt
==================================================

新增：

apps/api/src/modules/ai/prompts/character-generation.prompt.ts

Prompt 必须要求：

你是专业的 AI 漫剧角色设计师。

根据项目世界观生成角色。

角色必须：

符合当前世界观
符合文明背景
符合势力背景
具有明确的人物目标
具有明确人物冲突
能够用于连续剧集
具有可视觉化特征
避免空泛描述
避免与现有角色完全重复

要求输出严格 JSON。

==================================================
十三、Character Generation JSON Schema
==================================================

必须建立：

character-generation.schema.ts

根结构：

{
  "character": {
    "name": "...",
    "alias": "...",
    "gender": "...",
    "age": "...",
    "race": "...",
    "identity": "...",
    "role": "...",
    "personality": {},
    "appearance": {},
    "background": "...",
    "goal": "...",
    "motivation": "...",
    "conflict": "...",
    "abilities": []
  },
  "relationships": []
}

禁止未知字段。

必须 Schema Validation。

非法 JSON：

GenerationTask = FAILED

Schema 不通过：

GenerationTask = FAILED

绝对不能写入 Character。

==================================================
十四、Character Preview / Apply
==================================================

流程：

用户：

项目
 ↓
人物
 ↓
AI 生成人物
 ↓
填写需求
 ↓
生成
 ↓
Preview
 ↓
修改
 ↓
Apply

生成后：

GenerationTask.output

只保存结果。

不要立即创建 Character。

用户点击：

「应用」

之后：

POST

/projects/:projectId/generations/:id/apply

才真正创建 Character。

==================================================
十五、Apply Transaction
==================================================

使用：

prisma.$transaction

执行：

1. 创建 Character
2. 如果 AI 返回 civilizationName：
   尝试关联 Civilization
3. 如果 AI 返回 factionName：
   尝试关联 Faction
4. 创建 CharacterRelationship
5. GenerationTask 标记为已应用

任何一步失败：

全部 rollback。

不要出现：

Character 创建成功
Relationship 创建失败
GenerationTask 没更新

这种脏数据。

==================================================
十六、人物去重
==================================================

Apply 前检查：

同一个 project 中是否存在同名角色。

如果存在：

不要直接覆盖。

返回：

CHARACTER_NAME_CONFLICT

前端提示：

「项目中已经存在名为 XXX 的角色」

用户可以：

取消
或者修改名称后重新生成。

本阶段不要实现复杂 Merge。

==================================================
十七、Character Web UI
==================================================

新增：

/projects/:id/characters

页面结构：

顶部：

人物
+ 创建人物
+ AI生成人物

搜索框：

搜索人物姓名

筛选：

全部
主角
配角
NPC

可进一步筛选：

文明
势力

人物 Card：

头像占位
姓名
身份
角色定位
文明
势力
性格标签

点击进入：

/projects/:id/characters/:characterId

==================================================
十八、Character Detail
==================================================

详情页分：

基本信息

身份信息

外貌

性格

背景

目标

动机

核心冲突

能力

所属文明

所属势力

人物关系

AI生成记录

未来能力：

角色图片
角色声音

当前显示：

「即将开放」

不要实现假的功能。

==================================================
十九、Relationship UI
==================================================

人物详情页面显示：

人物关系图 / Relationship List

例如：

沈星河
 │
 ├── 师父 → 太虚真人
 ├── 宿敌 → 林天骄
 ├── 盟友 → 陈墨
 └── 未知 → 赛博文明角色

如果实现关系图成本过高：

Phase 5 可以先实现关系 Card/List。

但数据模型必须支持未来图谱。

==================================================
二十、AI Character Generation UI
==================================================

新增：

CharacterGenerateModal.vue

字段：

角色定位
姓名（可选）
性别
年龄
文明
势力
性格
外貌
背景
目标
冲突

Prompt：

自由描述

风格：

东方玄幻
赛博朋克
科幻
现代
其他

详细程度：

LOW
MEDIUM
HIGH

按钮：

生成

生成过程中：

显示：

正在构思人物...
正在结合世界观...
正在设计人物背景...
正在生成角色卡...

不要伪造真实进度。

可以显示统一：

「AI 正在生成」

==================================================
二十一、Preview UI
==================================================

生成完成后：

左侧：

AI生成角色卡

右侧：

人物详细信息

提供：

重新生成

应用人物

取消

应用前确认：

「应用后将创建人物及其关系，是否继续？」

==================================================
二十二、Generation History
==================================================

复用现有 Generation History。

显示：

时间
类型
状态
Provider
Model
耗时

例如：

Character
SUCCEEDED
DeepSeek
deepseek-chat
2.4s

禁止显示：

API Key

==================================================
二十三、Usage
==================================================

继续使用：

GenerationTask.usage

记录：

durationMs

如果 Provider 返回 token usage：

可以记录：

inputTokens
outputTokens
totalTokens

但不要实现真实计费。

可以预留：

estimatedCost

但是不要把它当真实账单。

==================================================
二十四、AI Cost Architecture
==================================================

非常重要：

本阶段必须继续遵守：

平台默认 Provider ≠ 平台承担用户长期 AI 成本。

当前：

Platform Provider / System Provider

只是开发、Demo、测试用途。

未来：

User
 ↓
自己的 Provider
 ↓
自己的 API Key
 ↓
自己的模型
 ↓
自己承担费用

因此代码结构必须保证：

ProjectAiConfig

可以指定：

capability
provider
model

未来登录完成后：

AiProvider.userId = 当前用户

Resolver：

Project Config
→ User Provider
→ Platform Default
→ System

现有 Resolver 不能破坏。

==================================================
二十五、不要实现以下功能
==================================================

本阶段禁止：

IMAGE Generation

VIDEO Generation

TTS

Voice Clone

Music

Storyboard

Episode

Script

Editing

Billing

Payment

Login

Register

JWT

Subscription

Redis

BullMQ

Worker

真正的异步任务队列

这些放到后续阶段。

==================================================
二十六、Migration
==================================================

新增 migration：

使用新的时间戳。

不要修改：

20260814100000_init
20260814160000_project_workspace
20260814164000_world_system
20260817020000_generation_engine
20260817040000_ai_provider_management
20260817130000_ai_capability_architecture

如果 Character 当前已经存在：

只做必要的 schema 增量。

不得 reset 数据库。

不得删除已有 World。

不得删除已有 Project。

不得删除 GenerationTask。

==================================================
二十七、Seed
==================================================

完善 Demo Seed。

为：

「星河碰撞」

增加至少：

修仙文明：

主角：
沈星河

赛博文明：

主要角色：
可以创建一个示例角色，例如：

艾尔

但不要过度设计剧情。

增加：

至少 3 个 Character

至少 3 条 Relationship

用于测试人物系统。

Seed 必须是：

显式执行

不能每次启动自动执行。

==================================================
二十八、测试
==================================================

必须增加测试。

至少覆盖：

Character CRUD

Character project isolation

Relationship CRUD

Relationship project validation

Self relationship rejection

Character generation

Provider Resolver

STRUCTURED_OUTPUT

Schema validation

Invalid JSON

GenerationTask FAILED

GenerationTask SUCCEEDED

Preview 不创建 Character

Apply 创建 Character

Apply 创建 Relationship

Apply transaction rollback

Duplicate character name

API Key 不泄漏

==================================================
二十九、质量检查
==================================================

完成后必须执行：

pnpm prisma validate

pnpm lint

pnpm typecheck

pnpm test

如果 migration 已经可以访问数据库：

pnpm exec prisma migrate deploy

并验证：

GET /health

项目列表

星河碰撞

世界观

AI Provider

AI Capability

人物

人物关系

AI Character Generation

==================================================
三十、最终输出
==================================================

完成后不要只说「完成」。

请输出：

1. 修改文件
2. 新增数据库模型
3. 新增 API
4. 新增 Shared Types
5. 新增 API Client
6. Character Generation 架构
7. Preview / Apply 流程
8. Provider Resolver 如何工作
9. UI 完成情况
10. Migration 是否执行
11. Seed 是否执行
12. 测试结果
13. lint 结果
14. typecheck 结果
15. prisma validate 结果
16. 当前项目数据是否保留
17. 当前 AI Provider 是否保留
18. 当前 GenerationTask 是否保留
19. 是否修改 Monorepo
20. 明确列出本阶段没有实现的功能

完成 Phase 5 后立即停止。

不要自行开始 Phase 5.1、Episode、Script、Storyboard 或 Image Generation。