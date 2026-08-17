现在开始开发项目 Phase 5：Character & Relationship System v1。

当前项目已经完成：

Phase 1：
Monorepo 基础架构

Phase 2：
Project Management + Project Workspace

Phase 3：
World System

Phase 4：
AI Generation Engine v1
目前支持 AI World Generation

Phase 4.5：
AI Provider Management
目前支持：
- Project Provider
- Default Provider
- System .env Provider
- Provider Resolver
- OpenAI Compatible Provider
- DeepSeek
- Provider 加密存储
- Provider Test Connection

当前 AI 世界观生成已经真实接入 DeepSeek 并验证成功。

==================================================
本阶段目标
==================================================

实现：

Character & Relationship System v1

包括：

1. Character 数据模型
2. Character CRUD API
3. Character Relationship 数据模型
4. Character Relationship CRUD API
5. Web 人物工作台
6. Mobile 人物页面
7. Desktop 人物基础面板
8. 世界观与人物关联
9. 人物与文明 / 势力关联
10. 人物关系展示
11. 为下一阶段 AI Character Generation 预留接口

本阶段：

❌ 不接入 LLM
❌ 不接入 DeepSeek
❌ 不实现 AI 生成人物
❌ 不实现剧集
❌ 不实现剧本
❌ 不实现分镜
❌ 不实现图片生成
❌ 不实现视频生成
❌ 不实现 TTS
❌ 不实现 BullMQ
❌ 不实现 Worker

本阶段只建立可靠的人物基础数据系统。

==================================================
一、架构约束
==================================================

禁止：

- 重建 Monorepo
- 更换技术栈
- 修改现有 Web / Mobile / Desktop 技术栈
- 删除已有 World System
- 删除已有 Generation Engine
- 删除 AI Provider Management
- 修改 Provider Resolver 的优先级
- 修改现有 World Generation API 行为
- 修改已有 World 数据

继续使用：

Web：
Nuxt 3 + Vue 3 + Tailwind + Pinia + VueUse

Mobile：
Ionic Vue + Capacitor

Desktop：
Tauri 2 + Vue

API：
NestJS

Database：
PostgreSQL + Prisma

Monorepo：
pnpm + Turborepo

Shared：
packages/types
packages/api-client
packages/core
packages/utils
packages/config

==================================================
二、数据库模型
==================================================

新增 Character。

建议：

Character

id
projectId
civilizationId?
factionId?
name
alias?
gender?
age?
role?
description?
personality?
appearance?
background?
motivation?
goal?
ability?
status
createdAt
updatedAt

要求：

projectId 必须存在。

civilizationId 可为空。

factionId 可为空。

不要强制人物必须属于文明或势力。

因为部分角色可能是：

- 星际流浪者
- 中立人物
- 神秘角色
- AI
- 未知文明角色

==================================================
三、Character Status
==================================================

增加：

CharacterStatus

ACTIVE
INACTIVE
ARCHIVED

默认：

ACTIVE

不要使用过度复杂的状态系统。

==================================================
四、Character Relationship
==================================================

新增：

CharacterRelationship

id
projectId
fromCharacterId
toCharacterId
type
description?
strength?
createdAt
updatedAt

要求：

fromCharacterId
toCharacterId

都必须属于同一个 Project。

禁止跨 Project 创建人物关系。

==================================================
五、Relationship Type
==================================================

建立枚举：

CharacterRelationType

FRIEND
ENEMY
ALLY
RIVAL
MASTER
DISCIPLE
FAMILY
LOVER
COLLEAGUE
TEACHER
STUDENT
PARTNER
UNKNOWN

注意：

MASTER / DISCIPLE

以及：

TEACHER / STUDENT

可以同时存在。

不要在数据库层面自动强制反向关系。

例如：

A MASTER B

不自动创建：

B DISCIPLE A

后续可以由业务层处理。

==================================================
六、数据库约束
==================================================

Character：

projectId 建立 index。

civilizationId 建立 index。

factionId 建立 index。

CharacterRelationship：

projectId 建立 index。

fromCharacterId 建立 index。

toCharacterId 建立 index。

建议增加：

@@unique([
  fromCharacterId,
  toCharacterId,
  type
])

避免重复创建：

A → B → FRIEND

多次。

==================================================
七、删除行为
==================================================

删除 Character 时：

必须正确处理 CharacterRelationship。

推荐：

关系自动删除。

即：

onDelete: Cascade

例如：

删除：

林玄

那么：

林玄 ↔ 艾琳

相关 CharacterRelationship 自动删除。

不要留下孤儿关系。

==================================================
八、Character 与 World
==================================================

Character 不直接添加：

worldId

因为：

Project → World

已经存在。

人物通过：

projectId

属于项目。

同时：

Character.civilizationId

关联 World.Civilization。

Character.factionId

关联 World.Faction。

这样结构：

Project
  ↓
World
  ├── Civilization
  │
  └── Faction

Project
  ↓
Character
  ├── Civilization
  └── Faction

==================================================
九、Migration
==================================================

创建新的 Prisma migration。

不要修改已有 migration：

20260814100000_init
20260814160000_project_workspace
20260814164000_world_system
20260817020000_generation_engine
20260817040000_ai_provider_management

新增：

2026xxxx_character_system

使用当前时间生成合理 migration timestamp。

必须支持：

全新数据库：

init
→ project_workspace
→ world_system
→ generation_engine
→ ai_provider_management
→ character_system

现有数据库：

直接应用 character_system migration。

不要 reset database。

不要删除已有数据。

==================================================
十、Shared Types
==================================================

增加：

Character
CharacterInput
CharacterUpdateInput

CharacterRelationship
CharacterRelationshipInput
CharacterRelationshipUpdateInput

CharacterStatus

CharacterRelationType

不要在 Web / Mobile / Desktop 中重复定义类型。

统一放：

packages/types/src/index.ts

==================================================
十一、Core
==================================================

增加：

packages/core/src/character.ts

定义：

Character 状态相关逻辑。

以及：

CharacterRelationship

相关纯函数。

不要把 HTTP 或 Prisma 逻辑放到 core。

==================================================
十二、API Client
==================================================

packages/api-client

增加：

getCharacters(projectId)

getCharacter(projectId, characterId)

createCharacter(projectId, input)

updateCharacter(projectId, characterId, input)

deleteCharacter(projectId, characterId)

以及：

getCharacterRelationships(projectId)

getCharacterRelationship(projectId, relationshipId)

createCharacterRelationship(projectId, input)

updateCharacterRelationship(projectId, relationshipId, input)

deleteCharacterRelationship(projectId, relationshipId)

所有 Web / Mobile / Desktop 都必须使用共享 API Client。

不要直接 fetch。

==================================================
十三、API 路由
==================================================

使用：

/projects/:projectId/characters

GET
POST

/projects/:projectId/characters/:characterId

GET
PATCH
DELETE

关系：

/projects/:projectId/character-relationships

GET
POST

/projects/:projectId/character-relationships/:relationshipId

GET
PATCH
DELETE

==================================================
十四、API 返回数据
==================================================

Character GET 建议返回：

{
  id,
  projectId,
  civilizationId,
  factionId,
  name,
  alias,
  gender,
  age,
  role,
  description,
  personality,
  appearance,
  background,
  motivation,
  goal,
  ability,
  status,
  createdAt,
  updatedAt
}

如果返回 civilization / faction 信息，可以使用：

civilization:
{
  id,
  name,
  type
}

faction:
{
  id,
  name
}

不要把整个 World 对象嵌套进去。

==================================================
十五、Character API 校验
==================================================

创建 Character：

必须验证：

projectId 存在。

如果提供 civilizationId：

必须验证该 Civilization 属于当前 Project 的 World。

如果提供 factionId：

必须验证该 Faction 属于当前 Project 的 World。

否则返回明确业务错误。

例如：

CIVILIZATION_NOT_IN_PROJECT

FACTION_NOT_IN_PROJECT

不要允许：

Project A Character
关联
Project B Civilization

==================================================
十六、Relationship API 校验
==================================================

创建 Relationship 时：

验证：

fromCharacterId 存在。

toCharacterId 存在。

两个人物都属于：

projectId。

否则：

CHARACTER_NOT_IN_PROJECT

不能创建：

Project A 的人物

关联

Project B 的人物。

==================================================
十七、禁止自己关联自己
==================================================

禁止：

A → A

返回：

INVALID_CHARACTER_RELATIONSHIP

==================================================
十八、Web 页面
==================================================

新增：

/projects/[id]/characters

作为项目人物工作台。

导航结构保持：

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

点击：

人物

进入：

/projects/[id]/characters

==================================================
十九、Web UI
==================================================

人物页面采用：

左侧：

人物列表

右侧：

人物详情

Desktop：

┌──────────────────────────────────────────┐
│ 人物                              + 新建 │
├──────────────┬───────────────────────────┤
│ 人物列表      │ 人物详情                 │
│              │                           │
│ 林玄         │ 林玄                      │
│ 艾琳·07      │ 修仙文明 · 主角           │
│ 诺亚         │                           │
│              │ 基本信息                  │
│              │ 性格                      │
│              │ 外貌                      │
│              │ 背景                      │
│              │ 目标                      │
│              │ 能力                      │
│              │                           │
│              │ 人物关系                  │
│              │ 林玄 → 艾琳·07            │
└──────────────┴───────────────────────────┘

==================================================
二十、Character Card
==================================================

人物列表显示：

名称

Alias

角色定位

文明

势力

状态

例如：

┌──────────────────────────┐
│ 林玄                     │
│ 主角                     │
│ 修仙文明 · 天玄宗         │
│ ACTIVE                   │
└──────────────────────────┘

==================================================
二十一、Character Form
==================================================

新建 / 编辑人物。

字段：

名称 *
别名
性别
年龄
角色定位

文明
势力

人物简介
性格
外貌
背景
动机
目标
能力

状态

保存。

不要在本阶段增加：

头像上传

图片生成

声音

视频

这些放到后续阶段。

==================================================
二十二、文明选择
==================================================

文明使用：

World.Civilization

下拉选择。

例如：

修仙文明

赛博文明

如果当前项目还没有 World：

提示：

请先创建世界观。

如果 World 存在但没有 Civilization：

显示：

暂无文明。

==================================================
二十三、势力选择
==================================================

势力使用：

World.Faction。

如果选择文明：

可以优先过滤：

该文明下的势力。

例如：

文明：

修仙文明

势力：

天玄宗
太虚宫

如果 faction 没有 civilization：

仍然允许选择。

==================================================
二十四、人物关系 UI
==================================================

人物详情增加：

人物关系

例如：

林玄

关系：

艾琳·07
[敌对]
双方互相不信任

诺亚
[盟友]
共同调查星系碰撞

操作：

+ 添加关系

编辑

删除

==================================================
二十五、添加关系 Modal
==================================================

字段：

目标人物 *

关系类型 *

描述

关系强度

关系强度可以使用：

1
2
3
4
5

默认：

3

不能选择当前人物作为目标人物。

==================================================
二十六、关系展示
==================================================

显示：

林玄
  ↓
敌对
  ↓
艾琳·07

而不是只显示：

Enemy

UI 必须显示中文名称。

Relationship enum 可以：

FRIEND → 朋友
ENEMY → 敌对
ALLY → 盟友
RIVAL → 竞争对手
MASTER → 师父
DISCIPLE → 弟子
FAMILY → 家人
LOVER → 恋人
COLLEAGUE → 同事
TEACHER → 老师
STUDENT → 学生
PARTNER → 搭档
UNKNOWN → 未知

统一放 shared config / utils。

不要 Web / Mobile 各自写一份映射。

==================================================
二十七、Mobile
==================================================

新增：

人物入口。

项目详情：

世界观
人物
场景
剧集
...

人物页面：

人物列表。

点击：

人物详情。

人物关系使用：

List / Card。

不要直接把 Desktop 双栏布局缩小。

保持 Mobile 独立体验。

==================================================
二十八、Desktop
==================================================

保持现有 Tauri App Shell。

侧栏增加：

人物

进入：

CharacterPanel

提供：

人物列表

基础详情

人物关系

完整复杂编辑仍可以优先 Web。

==================================================
二十九、Empty State
==================================================

如果项目没有人物：

显示：

还没有人物

创建第一个角色

以及：

AI 生成人物将在后续版本开放。

注意：

本阶段不要真正实现 AI Generation。

==================================================
三十、AI Generation 预留
==================================================

为了 Phase 5.5：

AI Character Generation

请保证 Character 数据结构未来可以被 AI Generation Engine 使用。

但：

本阶段不要新增：

POST /generations/character

不要接入 Provider。

不要修改 Generation Engine。

只确保模型字段足够表达人物。

==================================================
三十一、Character Context
==================================================

增加一个纯函数：

buildCharacterContext(character)

或者类似：

serializeCharacterContext(character)

用于未来 AI Prompt。

但是：

不要调用 AI。

不要发送网络请求。

返回结构化对象或文本均可。

==================================================
三十二、测试
==================================================

必须增加测试。

至少覆盖：

1.
创建 Character

2.
查询 Character

3.
更新 Character

4.
删除 Character

5.
Character 不属于 Project 时拒绝

6.
Civilization 不属于 Project 时拒绝

7.
Faction 不属于 Project 时拒绝

8.
创建 Relationship

9.
跨 Project Relationship 被拒绝

10.
Character 自关联被拒绝

11.
删除 Character 后 Relationship 正确删除

12.
重复 Relationship 被拒绝

13.
更新 Relationship

14.
删除 Relationship

==================================================
三十三、数据完整性
==================================================

禁止破坏已有：

Project

World

Civilization

WorldHistory

Faction

WorldLocation

PowerSystem

GenerationTask

AiProvider

数据。

不要执行：

prisma migrate reset

不要执行：

DROP DATABASE

不要删除已有项目。

==================================================
三十四、现有「星河碰撞」
==================================================

必须兼容已有：

「星河碰撞」

如果它已经有：

World

Civilization

Faction

那么人物页面可以直接创建人物。

建议至少手动验证：

林玄

文明：

修仙文明

艾琳·07

文明：

赛博文明

然后创建：

林玄 → 艾琳·07

关系：

RIVAL

或者：

ENEMY

==================================================
三十五、质量要求
==================================================

完成后执行：

pnpm prisma validate

pnpm lint

pnpm typecheck

pnpm test

全部通过。

==================================================
三十六、不要自动继续下一阶段
==================================================

本阶段完成后停止。

不要继续开发：

AI Character Generation

Episode

Script

Storyboard

Image

Video

TTS

Worker

Queue

==================================================
三十七、最终报告
==================================================

完成后告诉我：

1. 新增了哪些 Prisma Model
2. 新增了哪些 Enum
3. Migration 名称
4. 新增了哪些 API
5. 新增了哪些 Shared Types
6. 新增了哪些 API Client
7. Web 修改
8. Mobile 修改
9. Desktop 修改
10. Character Relationship 如何实现
11. Cascade Delete 如何实现
12. Cross Project Validation 如何实现
13. 新增测试数量
14. pnpm prisma validate
15. pnpm lint
16. pnpm typecheck
17. pnpm test
18. 是否修改了现有 World / Generation / Provider 数据
19. 是否修改 Monorepo
20. 是否接入 AI

如果发现现有代码结构与本 Prompt 存在冲突：

优先保持现有项目架构。

不要重构。

不要擅自更换技术栈。

不要删除现有功能。