你现在开始执行：

# Phase 6 — Season & Episode + Story Bible 基础架构

目标：
在现有 AI Drama Studio 基础上，建立「作品圣经 Story Bible + Season 季 + Episode 集」的故事生产基础架构。

本阶段是整个 AI 漫剧生产流水线的核心中间层。

最终目标：

Project
  ↓
Story Bible
  ↓
Season
  ↓
Episode
  ↓
未来 Phase 7 Script
  ↓
未来 Phase 8 Storyboard
  ↓
未来 Phase 9 Image
  ↓
未来 Phase 10 Video
  ↓
未来 Phase 11 Voice / Music
  ↓
未来 Phase 12 Editing
  ↓
Final Video

本阶段只实现：

1. Story Bible 基础架构
2. Season 管理
3. Episode 管理
4. Season → Episode 关系
5. Story Bible → World / Character / Season / Episode 的上下文关系
6. 基础 AI Season 拆集能力
7. 基础 AI Episode 大纲生成能力
8. Story Continuity / 状态快照基础架构
9. GenerationTask 接入现有 Provider → Model → Capability 架构
10. Web 完整工作台
11. Mobile / Desktop 基础入口与查看能力

禁止在本阶段实现：

- 完整 Script Generation
- Storyboard
- Image Generation
- Video Generation
- TTS
- Voice Clone
- Music
- Editing
- Redis
- BullMQ
- Worker
- 真正异步队列
- Billing
- Payment
- Login
- Register
- JWT
- Subscription
- 用户 Provider 所有权体系
- 多租户权限系统
- 自动发布平台
- YouTube / TikTok / Bilibili 等平台 API
- 任何新的 AI Provider
- 修改现有 Monorepo 技术栈

不要重建项目。
不要 reset 数据库。
不要删除现有数据。
不要修改已有历史 migration。
必须通过增量 migration 实现数据库变化。

==================================================
一、必须先理解当前项目架构
==================================================

当前项目：

pnpm + Turborepo Monorepo

apps/
  web       → Nuxt 3 + Vue 3
  mobile    → Ionic Vue + Capacitor
  desktop   → Tauri 2 + Vue
  api       → NestJS

packages/
  types
  api-client
  core
  utils
  config

数据库：
PostgreSQL + Prisma

当前 AI 架构：

Provider
  ↓
Model
  ↓
Capability
  ↓
ProjectAiConfig
  ↓
Provider Resolver
  ↓
Generation

现有 AiCapability 至少包括：

CHAT
STRUCTURED_OUTPUT
IMAGE
VIDEO
IMAGE_TO_VIDEO
TTS
VOICE_CLONE
MUSIC
EMBEDDING

当前文本类 AI 生成必须使用：

STRUCTURED_OUTPUT

不得绕过 Provider Resolver。

不得在业务代码中读取：

AI_API_KEY
AI_BASE_URL
AI_MODEL

不得写死 DeepSeek。

当前 Provider Resolver 优先级：

ProjectAiConfig
→ Legacy Project.aiProviderId
→ User Provider
→ Platform Default
→ System .env
→ NO_AI_PROVIDER_CONFIGURED

本阶段所有 AI 生成继续使用：

resolveForCapability(
  projectId,
  STRUCTURED_OUTPUT
)

==================================================
二、核心产品概念
==================================================

本阶段建立：

Project
  │
  ├── Story Bible
  │
  ├── World
  │
  ├── Characters
  │
  ├── Seasons
  │     ├── Episode 1
  │     ├── Episode 2
  │     ├── Episode 3
  │     └── ...
  │
  └── Generation Tasks

Story Bible 不是简单的一段 text。

它是整个作品的「故事上下文中心」。

未来所有：

Script
Storyboard
Image
Video
Voice
Editing

都应该可以从 Story Bible 获取一致的上下文。

==================================================
三、Story Bible 数据模型
==================================================

新增：

StoryBible

建议字段：

id
projectId
title
logline
premise
theme
tone
style
audience
storyPromise
rules
timelineSummary
continuityNotes
metadata
createdAt
updatedAt

关系：

Project 1 ── 1 StoryBible

projectId 必须 unique。

字段说明：

title：
作品名称。

logline：
一句话故事概述。

premise：
核心故事前提。

theme：
主题。

tone：
整体基调，例如：

- 热血
- 黑暗
- 史诗
- 喜剧
- 悬疑
- 科幻

style：
作品风格。

audience：
目标受众。

storyPromise：
作品向观众承诺的核心体验。

rules：
Json。

用于存储创作规则，例如：

{
  "worldRules": [],
  "characterRules": [],
  "narrativeRules": [],
  "forbidden": []
}

timelineSummary：
整体时间线摘要。

continuityNotes：
连续性约束。

metadata：
预留扩展字段。

不要把 World / Character 全部复制进 StoryBible。

StoryBible 是「上下文定义」，不是 World / Character 的替代品。

==================================================
四、Story Bible 与现有 World / Character 的关系
==================================================

不要复制数据。

关系应该是：

Project
 ├── StoryBible
 ├── World
 ├── Character
 └── Season

StoryBible 通过 projectId 间接关联：

World
Character
Season
Episode

未来 Context Builder 可以：

getStoryContext(projectId)

返回：

{
  storyBible,
  world,
  civilizations,
  factions,
  locations,
  powerSystems,
  characters,
  relationships,
  seasons,
  episodes
}

但不要一次把所有内容无限制塞给 AI。

本阶段只建立 Context Builder 基础接口。

例如：

StoryContextBuilder

buildProjectContext(projectId)

buildSeasonContext(projectId, seasonId)

buildEpisodeContext(projectId, episodeId)

未来 Phase 7 Script 会直接使用。

==================================================
五、Season 数据模型
==================================================

新增：

Season

字段：

id
projectId
number
title
synopsis
outline
status
metadata
createdAt
updatedAt

status：

DRAFT
PLANNING
READY
IN_PROGRESS
COMPLETED
ARCHIVED

约束：

projectId + number unique

例如：

Project
  Season 1
  Season 2
  Season 3

number：

1
2
3

title：

星河初遇

synopsis：

第一季整体故事简介。

outline：

第一季详细故事大纲。

outline 可以使用 Text。

metadata：

Json。

未来可以存：

season arc
theme
ending
foreshadowing
etc.

==================================================
六、Episode 数据模型
==================================================

新增：

Episode

字段建议：

id
seasonId
number
title
synopsis
outline
status
durationSeconds
storyState
continuityNotes
metadata
createdAt
updatedAt

status：

DRAFT
OUTLINED
SCRIPTING
READY
IN_PRODUCTION
COMPLETED
ARCHIVED

约束：

seasonId + number unique

durationSeconds：

预期成片时长。

例如：

300

表示 5 分钟。

不要实现实际视频时长计算。

==================================================
七、Episode Story State
==================================================

这是本阶段非常重要的设计。

新增：

EpisodeStoryState

或者如果你认为不需要独立表，可以作为 Episode.storyState Json。

优先使用 Episode.storyState Json，避免过度设计。

用于保存：

“这一集结束以后，故事世界变成什么状态”。

例如：

{
  "characters": [
    {
      "characterId": "...",
      "state": "炼气九层",
      "location": "天玄宗",
      "condition": "healthy",
      "goal": "调查星系异常"
    }
  ],
  "relationships": [],
  "worldChanges": [],
  "factionChanges": [],
  "unresolvedThreads": [],
  "revealedSecrets": [],
  "foreshadowing": []
}

必须支持：

currentState

以及未来：

previousEpisodeState

这样未来生成下一集时：

Episode N
  ↓
Story State
  ↓
Episode N+1

==================================================
八、Story Continuity 基础架构
==================================================

新增：

StoryContinuityService

至少提供：

getEpisodeContext(projectId, episodeId)

getPreviousEpisodeState(seasonId, episodeNumber)

getCurrentStoryState(episodeId)

validateEpisodeContinuity(...)

本阶段不需要实现复杂 AI Continuity Checker。

至少实现：

1. Episode 是否属于指定 Season
2. Season 是否属于指定 Project
3. Episode number 是否连续
4. Character 是否属于当前 Project
5. World 是否属于当前 Project
6. 不能跨 Project 引用数据
7. Previous Episode 状态可以读取

未来：

Script
Storyboard
Image
Video

全部依赖这一层。

==================================================
九、Season CRUD API
==================================================

新增：

GET
/projects/:projectId/seasons

GET
/projects/:projectId/seasons/:id

POST
/projects/:projectId/seasons

PATCH
/projects/:projectId/seasons/:id

DELETE
/projects/:projectId/seasons/:id

请求：

POST

{
  "number": 1,
  "title": "星河初遇",
  "synopsis": "...",
  "outline": "..."
}

创建 Season 时：

校验 Project 存在。

number 不允许重复。

删除 Season 时：

必须明确处理 Episode。

建议：

如果 Season 下存在 Episode：

默认拒绝删除。

返回：

SEASON_HAS_EPISODES

不要级联删除 Episode。

==================================================
十、Episode CRUD API
==================================================

新增：

GET
/projects/:projectId/seasons/:seasonId/episodes

GET
/projects/:projectId/seasons/:seasonId/episodes/:id

POST
/projects/:projectId/seasons/:seasonId/episodes

PATCH
/projects/:projectId/seasons/:seasonId/episodes/:id

DELETE
/projects/:projectId/seasons/:seasonId/episodes/:id

创建：

{
  "number": 1,
  "title": "星系碰撞",
  "synopsis": "...",
  "outline": "...",
  "durationSeconds": 300
}

必须验证：

Project
→ Season
→ Episode

三层关系正确。

禁止：

Project A
Season A
Episode B

这种跨项目引用。

删除规则：

如果 Episode 已经存在未来生成任务 / 成片资产，则不要物理删除。

本阶段如果还没有相关资产，可以允许删除。

但代码结构要预留：

EPISODE_HAS_DEPENDENCIES

==================================================
十一、Season AI 拆集
==================================================

实现：

POST

/projects/:projectId/generations/season-outline

作用：

根据 Season 大纲生成 Episode 列表。

输入：

{
  "seasonId": "...",
  "instruction": "...",
  "episodeCount": 12,
  "targetDurationSeconds": 300
}

AI 使用：

resolveForCapability(
  projectId,
  STRUCTURED_OUTPUT
)

不得写死 Provider。

Prompt 必须包含：

Story Bible
World summary
Characters summary
当前 Season 信息

但是必须控制 Context 大小。

不要把所有历史 GenerationTask 全部注入。

==================================================
十二、Season AI 输出 Schema
==================================================

结构：

{
  "season": {
    "title": "...",
    "synopsis": "...",
    "coreConflict": "...",
    "beginning": "...",
    "middle": "...",
    "ending": "..."
  },
  "episodes": [
    {
      "number": 1,
      "title": "...",
      "synopsis": "...",
      "outline": "...",
      "keyCharacters": [],
      "keyLocations": [],
      "conflict": "...",
      "cliffhanger": "...",
      "storyStateChanges": {
        "characters": [],
        "worldChanges": [],
        "unresolvedThreads": [],
        "foreshadowing": []
      }
    }
  ]
}

严格 Schema Validation。

禁止未知字段。

非法 JSON：

FAILED

Schema 失败：

FAILED

不要创建 Episode。

==================================================
十三、Season Generation Preview / Apply
==================================================

遵循之前 World / Character 的模式。

流程：

Generate
 ↓
GenerationTask
 ↓
Preview
 ↓
User Confirm
 ↓
Apply
 ↓
Episode

绝对不能：

AI 生成完成
→ 自动创建 Episode

必须：

Preview
→ 用户确认
→ Apply

Apply 使用：

prisma.$transaction

事务中：

1. 校验 Season
2. 校验 episode numbers
3. 检查重复
4. 创建 Episode
5. 写入 storyState
6. 标记 GenerationTask.appliedAt

任何失败：

全部 rollback。

==================================================
十四、Episode AI 大纲生成
==================================================

实现：

POST

/projects/:projectId/generations/episode

输入：

{
  "episodeId": "...",
  "instruction": "..."
}

或者：

如果 Episode 还没有创建：

支持：

Generate Episode Outline Preview

但不要重复设计 API。

优先：

先创建 Episode Draft
→ AI 生成大纲
→ Preview
→ Apply

AI Context：

StoryBible
World
Characters
Season
Previous Episode
Current Episode

输出：

{
  "title": "...",
  "synopsis": "...",
  "outline": "...",
  "opening": "...",
  "middle": "...",
  "ending": "...",
  "cliffhanger": "...",
  "keyCharacters": [],
  "keyLocations": [],
  "conflict": "...",
  "storyState": {
    "characters": [],
    "worldChanges": [],
    "unresolvedThreads": [],
    "revealedSecrets": [],
    "foreshadowing": []
  }
}

本阶段仍然不是 Script。

只是：

Episode Outline。

==================================================
十五、GenerationTask
==================================================

扩展 GenerationTaskType。

增加：

SEASON_OUTLINE
EPISODE_OUTLINE

GenerationTask.capability：

STRUCTURED_OUTPUT

GenerationTask usage：

继续使用现有 usage Json。

记录：

durationMs

未来可以增加：

inputTokens
outputTokens
totalTokens
estimatedCost
actualCost

本阶段不要实现真实计费。

==================================================
十六、GenerationTask API
==================================================

继续复用：

GET
/projects/:projectId/generations

GET
/projects/:projectId/generations/:id

不要重新创建 Generation History 系统。

GenerationTask 必须记录：

type
status
provider
model
capability
input
output
usage
error
createdAt
updatedAt
appliedAt

API Key 永远不能返回。

==================================================
十七、Story Bible API
==================================================

新增：

GET
/projects/:projectId/story-bible

POST
/projects/:projectId/story-bible

PATCH
/projects/:projectId/story-bible

DELETE
/projects/:projectId/story-bible

因为 Project : StoryBible = 1 : 1。

如果没有：

GET 返回 404。

POST 重复：

409 STORY_BIBLE_EXISTS

删除时：

不要删除 World / Character / Season。

==================================================
十八、Story Bible AI Generation
==================================================

本阶段可以实现一个：

AI 生成 / 完善 Story Bible

API：

POST

/projects/:projectId/generations/story-bible

输入：

{
  "instruction": "...",
  "tone": "...",
  "style": "...",
  "audience": "..."
}

AI 使用：

STRUCTURED_OUTPUT

输出：

{
  "title": "...",
  "logline": "...",
  "premise": "...",
  "theme": "...",
  "tone": "...",
  "style": "...",
  "audience": "...",
  "storyPromise": "...",
  "rules": {
    "worldRules": [],
    "characterRules": [],
    "narrativeRules": [],
    "forbidden": []
  },
  "timelineSummary": "...",
  "continuityNotes": []
}

依然：

Generate
→ Preview
→ Apply

不允许自动覆盖已有 Story Bible。

已有 Story Bible：

Apply 前必须确认覆盖。

==================================================
十九、Story Context Builder
==================================================

新增：

StoryContextBuilder

必须尽可能与 AI Provider 解耦。

提供：

buildStoryBibleContext(projectId)

buildWorldContext(projectId)

buildCharacterContext(projectId)

buildSeasonContext(projectId, seasonId)

buildEpisodeContext(projectId, episodeId)

buildProjectContext(projectId)

返回结构化对象。

不要直接拼接最终 Prompt。

Context Builder 负责：

数据聚合。

Prompt Builder 负责：

将 Context 转换为 AI Prompt。

保持职责分离。

未来：

Phase 7 Script

只需要：

const context =
  await storyContextBuilder.buildEpisodeContext(...)

然后：

scriptPrompt.build(context)

==================================================
二十、上下文大小控制
==================================================

必须考虑 AI Context Window。

不要把整个 Project 全量发送。

提供：

summary mode

例如：

World：

只发送：

name
description
cosmicBackground
coreConflict

Civilization：

name
description
philosophy
technology

Character：

name
role
identity
personalityProfile
goal
conflict

不要默认发送：

metadata
imageProfile
voiceProfile
全部历史数据

Episode：

只发送：

当前 Episode
Previous Episode
必要的 Season 信息

==================================================
二十一、Web UI
==================================================

新增：

/projects/:id/story-bible

页面：

Story Bible 工作台。

模块：

基本信息
故事一句话
故事前提
主题
风格
受众
故事承诺
世界规则
人物规则
叙事规则
禁止事项
时间线
连续性规则

提供：

编辑
保存
AI生成
AI完善

AI 生成：

Modal
→ Prompt
→ Preview
→ Apply

==================================================
二十二、Season Web
==================================================

新增：

/projects/:id/seasons

页面：

Season 列表。

例如：

Season 1
星河初遇

Season 2
机械天庭

Season 3
终焉之战

支持：

创建
编辑
删除
查看

==================================================
二十三、Season Detail
==================================================

新增：

/projects/:id/seasons/:seasonId

页面：

Season Header

标题
简介
大纲
状态

Episode Timeline：

E01
E02
E03
...

支持：

拖动排序。

但注意：

Episode.number 必须和排序保持一致。

如果实现拖动：

必须通过后端事务重新编号。

==================================================
二十四、AI 拆集 UI
==================================================

Season 页面提供：

「AI 拆分剧集」

Modal：

剧集数量
目标单集时长
额外要求

例如：

12 集
5 分钟 / 集

点击：

开始生成

然后：

Preview

显示：

E01
标题
简介
冲突
悬念

E02
...

用户可以：

确认全部

或者：

取消

本阶段不要实现逐集复杂编辑器。

==================================================
二十五、Episode Detail
==================================================

新增：

/projects/:id/seasons/:seasonId/episodes/:episodeId

页面：

Episode Header

标题
集数
状态
预计时长

故事大纲

开场

中段

结尾

核心冲突

悬念

主要人物

主要地点

Story State

Unresolved Threads

Foreshadowing

提供：

AI生成大纲

Preview

Apply

==================================================
二十六、Web 导航
==================================================

Project Workspace：

项目概览
世界观
人物
季
剧集
剧本
分镜
图片
视频
配音
成片
素材
设置

本阶段：

季 / 剧集启用。

剧本 / 分镜 / 图片 / 视频：

仍然显示：

「将在后续阶段开放」

不要假功能。

==================================================
二十七、Mobile
==================================================

Mobile 不需要实现完整复杂编辑器。

增加：

项目
 ↓
季
 ↓
剧集

支持：

查看
创建
编辑基础字段
查看 Story Bible

AI 生成：

引导 Web 完成。

保持 Mobile 独立 UI。

不要把 Desktop Sidebar 压缩成 Mobile。

==================================================
二十八、Desktop
==================================================

Desktop：

增加：

Story Bible
Seasons
Episodes

可以提供基础查看 / 编辑。

复杂 AI 生成流程仍可以引导 Web。

==================================================
二十九、Shared Types
==================================================

packages/types：

新增：

StoryBible

StoryBibleInput

UpdateStoryBibleInput

Season

SeasonInput

UpdateSeasonInput

Episode

EpisodeInput

UpdateEpisodeInput

EpisodeStoryState

StoryContext

SeasonGenerationInput

SeasonGenerationResult

EpisodeGenerationInput

EpisodeGenerationResult

StoryBibleGenerationInput

StoryBibleGenerationResult

SeasonStatus

EpisodeStatus

GenerationTaskType：

STORY_BIBLE
SEASON_OUTLINE
EPISODE_OUTLINE

不要在 Web / Mobile / Desktop 重复定义。

==================================================
三十、packages/core
==================================================

新增：

story-bible.ts

season.ts

episode.ts

story-context.ts

continuity.ts

包含：

状态枚举

业务类型

Context 类型

纯函数

不要把 Prisma / NestJS 代码放进 core。

==================================================
三十一、API Client
==================================================

packages/api-client：

新增：

Story Bible：

getStoryBible()
createStoryBible()
updateStoryBible()
deleteStoryBible()

Season：

getSeasons()
getSeason()
createSeason()
updateSeason()
deleteSeason()

Episode：

getEpisodes()
getEpisode()
createEpisode()
updateEpisode()
deleteEpisode()

Generation：

createStoryBibleGeneration()
createSeasonOutlineGeneration()
createEpisodeOutlineGeneration()

applyGeneration()

保持与现有 API 风格一致。

页面禁止直接 fetch。

==================================================
三十二、Prompt Architecture
==================================================

新增：

apps/api/src/modules/generation/prompts/

story-bible-generation.prompt.ts

season-outline-generation.prompt.ts

episode-outline-generation.prompt.ts

不要把 Prompt 写死在 Service。

Prompt 必须接收 Context。

例如：

buildSeasonOutlinePrompt({
  storyBible,
  world,
  characters,
  season,
  instruction
})

==================================================
三十三、Schema Validation
==================================================

必须为：

StoryBible Generation

Season Outline Generation

Episode Outline Generation

分别创建 Schema。

禁止：

any

禁止：

直接相信 AI 返回的数据。

必须：

AI
→ JSON parse
→ Schema validation
→ GenerationTask.output
→ Preview
→ Apply

==================================================
三十四、Generation Executor
==================================================

继续复用：

GenerationExecutor

不要重新实现一个新的 executor。

流程：

create task
→ RUNNING
→ resolve provider
→ generate structured
→ validate
→ SUCCESS / FAILED

错误必须脱敏。

==================================================
三十五、成本架构
==================================================

本阶段不做计费。

但必须保证：

所有 AI GenerationTask 都记录：

provider
model
capability
usage
durationMs

未来可以计算：

token cost
image cost
video cost
tts cost

本阶段不实现：

Billing

Payment

Subscription

Balance

==================================================
三十六、数据一致性
==================================================

绝对禁止：

AI 自动修改 World
AI 自动修改 Character
AI 自动修改 Season
AI 自动修改 Episode

所有 AI 输出：

Preview

用户确认：

Apply

Apply 必须 Transaction。

==================================================
三十七、Migration
==================================================

新增 migration：

使用新的 timestamp。

例如：

20260817xxxxxx_story_bible_season_episode

不要修改：

20260814100000_init

20260814160000_project_workspace

20260814164000_world_system

20260817020000_generation_engine

20260817040000_ai_provider_management

20260817130000_ai_capability_architecture

20260817140000_character_generation

任何历史 migration。

不要 prisma migrate reset。

==================================================
三十八、Seed
==================================================

新增：

prisma/seed-story-demo.ts

使用现有：

「星河碰撞」

创建：

Story Bible

Season 1：

星河初遇

至少 3 个 Episode：

E01 星系碰撞

E02 第一次接触

E03 临时盟友

不要覆盖已有 World / Character。

Seed 必须显式执行：

pnpm db:seed:story-demo

不要自动执行。

==================================================
三十九、测试
==================================================

必须增加测试。

至少覆盖：

StoryBible：

- 创建
- 获取
- 更新
- 删除
- project 隔离
- 重复创建拒绝

Season：

- CRUD
- project 隔离
- number 唯一
- 删除有 Episode 时拒绝

Episode：

- CRUD
- season/project 隔离
- number 唯一
- previous episode state
- storyState

Context：

- Project Context
- Season Context
- Episode Context
- previous episode
- project isolation

AI：

- Provider Resolver 使用 STRUCTURED_OUTPUT
- Story Bible Schema
- Season Schema
- Episode Schema
- 非法 JSON → FAILED
- Schema failure → FAILED
- Preview 不创建 Episode
- Apply 创建 Episode
- Apply transaction rollback
- API Key 不泄漏

Generation：

- generation type 正确
- capability 正确
- provider/model 正确记录
- usage.durationMs

==================================================
四十、质量要求
==================================================

完成后必须运行：

pnpm prisma validate

pnpm lint

pnpm typecheck

pnpm test

全部通过才能结束。

如果数据库 migration 尚未执行：

pnpm exec prisma migrate deploy

不要使用：

pnpm prisma migrate reset

==================================================
四十一、必须保持现有数据
==================================================

执行前确认：

Project：

「星河碰撞」

World：

必须保留。

Character：

沈星河
太虚真人
艾尔

必须保留。

现有 Provider：

必须保留。

现有 GenerationTask：

必须保留。

Phase 4 / 4.5 / 4.6 / 5：

不能破坏。

==================================================
四十二、不要过度设计
==================================================

本阶段不要建立：

StoryGraph

KnowledgeGraph

Vector Database

RAG

Embedding

AI Agent

Redis

Queue

Worker

Event Bus

复杂权限

Billing

Subscription

复杂 Asset Pipeline

这些全部留给未来阶段。

当前只建立：

Story Bible
Season
Episode
Context
Continuity 基础能力。

==================================================
四十三、最终产品流程
==================================================

完成后用户应该能够：

创建项目

↓

填写 Story Bible

↓

创建 Season 1

↓

填写 Season 大纲

↓

点击：

「AI 拆分剧集」

↓

Preview：

E01
E02
E03
...

↓

用户确认

↓

创建 Episodes

↓

进入 Episode

↓

查看：

故事大纲
主要人物
主要地点
冲突
悬念
Story State

↓

点击：

「AI 生成本集大纲」

↓

Preview

↓

Apply

↓

Episode 完成。

未来 Phase 7：

Episode
↓
Script

未来 Phase 8：

Script
↓
Storyboard

未来 Phase 9：

Storyboard
↓
Image

未来 Phase 10：

Image
↓
Video

==================================================
四十四、非常重要的架构原则
==================================================

以后所有 AI 生成都必须遵守：

用户输入
+
结构化项目上下文
+
Provider Resolver
+
Capability
+
Model
+
AI
+
Schema Validation
+
Preview
+
Human Approval
+
Transaction Apply

不要：

User
→ AI
→ 直接写数据库

必须：

User
→ AI
→ GenerationTask
→ Preview
→ Apply
→ Database

==================================================
四十五、本阶段完成标准
==================================================

完成后告诉我：

1. 修改了哪些文件
2. 新增哪些 Prisma Model
3. 新增哪些字段
4. Migration 名称
5. 是否执行 migration
6. Story Bible API
7. Season API
8. Episode API
9. AI Generation API
10. Context Builder 结构
11. Continuity 结构
12. GenerationTask 新增类型
13. Provider Resolver 是否继续兼容
14. Web 完成情况
15. Mobile 完成情况
16. Desktop 完成情况
17. Seed 数据
18. 现有「星河碰撞」是否保留
19. 现有 World 是否保留
20. 现有 Character 是否保留
21. 现有 Provider 是否保留
22. 现有 GenerationTask 是否保留
23. pnpm prisma validate
24. pnpm lint
25. pnpm typecheck
26. pnpm test
27. 测试数量
28. 是否修改 Monorepo
29. 是否新增任何 AI Provider
30. 明确列出本阶段没有实现的功能

如果发现架构问题：

优先修正设计。

不要为了快速完成而绕过：

Provider Resolver
Schema Validation
Preview / Apply
Transaction
Context Builder
Project Isolation

不要修改技术栈。

不要重建 Monorepo。

不要 reset 数据库。

开始执行 Phase 6。