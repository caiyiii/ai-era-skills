你现在开始执行：

# Phase 7：Script Engine
## 剧本系统 + AI 剧本生成 + Scene / Dialogue / Action 基础架构

这是 AI Drama Studio 的第七阶段。

必须严格基于当前已经完成的 Phase 1 ~ Phase 6 继续开发。

不要重建项目。
不要更换技术栈。
不要重建 Monorepo。
不要修改历史 migration。
不要 reset 数据库。
不要删除已有数据。
不要绕过现有 API Client。
不要让 Web / Mobile / Desktop 直接调用 AI Provider。

如果发现现有代码存在兼容性问题，应做最小增量修复。

==================================================
一、当前系统架构必须保持
==================================================

Monorepo：

pnpm + Turborepo

Apps：

apps/web
Nuxt 3 + Vue 3 + Tailwind + Pinia + VueUse

apps/mobile
Ionic Vue + Capacitor

apps/desktop
Tauri 2 + Vue

apps/api
NestJS

Packages：

packages/types
packages/api-client
packages/core
packages/config
packages/utils

Database：

PostgreSQL + Prisma

当前端口：

Web：
3010

API：
3011

==================================================
二、已经完成的能力
==================================================

当前已经存在：

Project

StoryBible

World

Civilization

Faction

WorldLocation

PowerSystem

Character

CharacterRelationship

Season

Episode

GenerationTask

AiProvider

AiModel

AiProviderCapability

ProjectAiConfig

当前 AI 架构：

Web / Mobile / Desktop
        ↓
packages/api-client
        ↓
NestJS API
        ↓
Generation
        ↓
ProviderResolver
        ↓
AiService
        ↓
具体 Provider Adapter

Provider Resolver 当前优先级：

ProjectAiConfig
↓
Legacy Project.aiProviderId
↓
User Provider
↓
Platform Default
↓
System .env
↓
NO_AI_PROVIDER_CONFIGURED

AI 能力：

CHAT
STRUCTURED_OUTPUT
IMAGE
VIDEO
IMAGE_TO_VIDEO
TTS
VOICE_CLONE
MUSIC
EMBEDDING

当前真实实现：

CHAT / STRUCTURED_OUTPUT

当前真实 Provider：

OpenAI Compatible

可以连接 DeepSeek。

注意：

DeepSeek 只是当前可以使用的 Provider。

禁止把 DeepSeek 写死在任何业务逻辑、Prompt、Generation Service 或 UI 中。

用户未来必须可以自己承担 AI 成本，并自行配置不同阶段的 Provider / Model。

==================================================
三、Phase 7 的核心目标
==================================================

本阶段实现：

# Script Engine

让每一个 Episode 拥有真正结构化的剧本。

剧本不是一个 text 字段。

必须拆成：

Script
Scene
Dialogue
Action
Narration

为后面的：

Storyboard
Image
Video
TTS
Editing

提供稳定的数据基础。

==================================================
四、非常重要：不要把 Script 和 Episode 混在一起
==================================================

Episode：

负责：

第几集
标题
大纲
故事状态
连续性
时长
状态

Script：

负责：

这一集具体“怎么演”。

关系：

Episode
  ↓
Script
  ↓
Scene
  ↓
ScriptBlock
  ├── Dialogue
  ├── Action
  ├── Narration
  └── Direction

不要把大量剧本内容直接塞进 Episode。

==================================================
五、Prisma 数据模型设计
==================================================

新增：

Script

字段建议：

id
episodeId
title
version
status
logline
summary
estimatedDurationSeconds
createdAt
updatedAt

关系：

Episode 1 ---- 1 Script

唯一：

episodeId

==================================================

新增：

Scene

字段：

id
scriptId
number
title
location
timeOfDay
summary
purpose
conflict
estimatedDurationSeconds
metadata
createdAt
updatedAt

关系：

Script 1 ---- N Scene

唯一：

scriptId + number

Scene 必须拥有稳定顺序。

==================================================

新增：

ScriptBlock

不要为 Dialogue / Action / Narration 建三个完全独立的重复表。

使用：

ScriptBlock

字段：

id
sceneId
order
type
content
characterId?
metadata
createdAt
updatedAt

enum：

ScriptBlockType：

DIALOGUE
ACTION
NARRATION
DIRECTION

关系：

Scene 1 ---- N ScriptBlock

唯一：

sceneId + order

==================================================
六、Dialogue 相关结构
==================================================

Dialogue 使用：

ScriptBlock.type = DIALOGUE

characterId：

可为空。

如果是角色对白，必须尽量关联 Character。

禁止只把角色名字作为自由文本保存而不建立关系。

同时 metadata 可以预留：

emotion
delivery
subtext
voiceStyle

但不要把这些作为第一阶段强制字段。

==================================================
七、Action
==================================================

Action：

ScriptBlock.type = ACTION

描述：

角色行为
环境变化
镜头可见动作
剧情动作

注意：

Action 是后续 Storyboard 的重要输入。

所以不要设计成纯富文本。

必须保证：

Scene
→ Action
→ 后续 Storyboard

可以稳定转换。

==================================================
八、Narration
==================================================

Narration：

ScriptBlock.type = NARRATION

用于：

旁白
世界观解释
时间跳转
场景说明

后续 TTS 可以直接使用 Narration。

==================================================
九、Direction
==================================================

Direction：

ScriptBlock.type = DIRECTION

用于：

导演提示
镜头提示
表演提示
节奏提示

但注意：

Phase 7 不实现完整 Storyboard。

这里只保留结构。

后续 Phase 8 Storyboard 可以读取：

DIRECTION
ACTION
DIALOGUE
SCENE

==================================================
十、Script 状态
==================================================

新增：

ScriptStatus：

DRAFT
GENERATING
READY
LOCKED

说明：

DRAFT：

用户编辑中。

GENERATING：

AI 正在生成。

READY：

剧本已经生成并确认。

LOCKED：

剧本已经进入后续生产阶段。

Phase 7 暂时不要自动 LOCK。

==================================================
十一、Episode 与 Script 关系
==================================================

一个 Episode 最多一个当前 Script。

但是必须支持：

version

未来可以：

Script v1
Script v2
Script v3

因此不要设计成无法扩展版本。

本阶段可以：

Script 当前只有一个 active Script。

未来可以扩展 ScriptVersion。

如果实现成本低，可以直接设计：

ScriptVersion

但不要过度设计。

优先保证：

Episode
→ Script
→ Scene
→ ScriptBlock

清晰。

==================================================
十二、AI Script Generation
==================================================

新增：

POST

/projects/:projectId/generations/script

请求：

episodeId

用户可以提供：

prompt
tone
style
targetDurationSeconds
additionalInstructions

AI 必须读取：

Project
StoryBible
World
Characters
Season
Episode
Episode Outline

以及：

StoryContinuityService

==================================================
十三、Context Builder
==================================================

不要直接在 Script Generation Service 中手写大量 Prisma 查询。

继续使用：

StoryContextBuilder

新增：

buildScriptContext()

或者：

buildEpisodeScriptContext()

必须包含：

Project summary

StoryBible：

logline
premise
theme
tone
style
storyPromise
rules
timelineSummary
continuityNotes

World：

name
summary
cosmicBackground
coreConflict

Characters：

name
role
identity
personality
goal
conflict

Season：

number
title
synopsis
outline

Episode：

number
title
outline
storyState
continuityNotes

以及必要的上一集信息。

不要把：

API Key
GenerationTask 全量记录
imageProfile
voiceProfile
metadata 大对象

全部塞进 Prompt。

==================================================
十四、AI Script JSON Schema
==================================================

AI 必须返回结构化 JSON。

根对象：

{
  "script": {},
  "scenes": []
}

script：

title
logline
summary
estimatedDurationSeconds

scenes：

[
  {
    "number": 1,
    "title": "",
    "location": "",
    "timeOfDay": "",
    "summary": "",
    "purpose": "",
    "conflict": "",
    "estimatedDurationSeconds": 0,
    "blocks": []
  }
]

blocks：

[
  {
    "order": 1,
    "type": "DIALOGUE",
    "characterName": "",
    "content": "",
    "metadata": {}
  }
]

type：

DIALOGUE
ACTION
NARRATION
DIRECTION

必须使用严格 JSON Schema。

禁止未知字段。

==================================================
十五、角色关联
==================================================

AI 返回：

characterName

不要让 AI 返回 Character UUID。

Apply 时：

characterName
↓
当前 Project Character
↓
characterId

如果找不到：

不要自动创建人物。

可以：

保留 characterName
并记录 warning

或者如果是 Dialogue：

必须明确标记为 unresolvedCharacter。

不要偷偷创建 Character。

==================================================
十六、Preview / Apply
==================================================

严格保持之前的：

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
Transaction

AI 生成成功：

只保存：

GenerationTask.output

绝对不能直接写：

Script
Scene
ScriptBlock

==================================================
十七、Apply Transaction
==================================================

用户点击：

“应用剧本”

执行：

prisma.$transaction

步骤：

1.

检查 Episode 属于 Project。

2.

检查 GenerationTask 属于 Project。

3.

检查 GenerationTask.type === SCRIPT。

4.

检查 GenerationTask.status === SUCCEEDED。

5.

创建或更新 Script。

6.

删除旧 Scene。

7.

删除旧 ScriptBlock。

8.

按 JSON 创建 Scene。

9.

按 order 创建 ScriptBlock。

10.

Dialogue characterName 尝试关联 Character。

11.

更新 Episode。

12.

GenerationTask.appliedAt = now。

任何一步失败：

全部 rollback。

==================================================
十八、不要覆盖用户正在编辑的剧本
==================================================

如果 Episode 已经存在 Script：

用户再次 AI 生成：

必须生成新的 GenerationTask。

不能直接覆盖 Script。

Preview 阶段：

显示：

“当前剧本已有内容，应用后将替换当前剧本。”

必须二次确认。

==================================================
十九、GenerationTask
==================================================

新增：

GenerationTaskType：

SCRIPT

如果已经存在其他类型：

全部保留。

Script Generation：

capability：

STRUCTURED_OUTPUT

记录：

provider
model
durationMs

继续不实现真实 billing。

==================================================
二十、AI Provider
==================================================

Script Generation 必须：

resolveForCapability(
  projectId,
  STRUCTURED_OUTPUT
)

绝对禁止：

AI_API_KEY
AI_MODEL
AI_BASE_URL

直接出现在 Script Generation Service。

禁止：

if deepseek
if openai

这样的业务判断。

==================================================
二十一、AI Prompt
==================================================

新增：

apps/api/src/modules/generation/prompts/script-generation.prompt.ts

Prompt 必须强调：

你是专业影视/动画剧本编剧。

必须严格遵守：

Story Bible
World
Character
Season
Episode

不能：

改变世界观核心规则
擅自改变人物核心设定
擅自增加重要人物
擅自改变人物关系
跳过 Episode 大纲
与上一集发生明显连续性冲突

如果信息不足：

优先基于已有上下文合理补全。

但不得破坏 Story Bible。

==================================================
二十二、剧本生成风格
==================================================

支持：

tone

style

additionalInstructions

例如：

电影感
轻小说
日漫
国漫
短剧
爽剧
严肃科幻
热血
悬疑
喜剧

但是：

这些只是生成参数。

不要写死 Prompt。

==================================================
二十三、剧本时长
==================================================

用户可以指定：

targetDurationSeconds

例如：

60 秒
90 秒
180 秒
300 秒

AI 生成时需要合理控制：

Scene 数量
Dialogue 数量
Narration
Action

不要只生成一篇几千字文章。

==================================================
二十四、漫剧模式
==================================================

项目未来主要面向：

AI 漫剧

因此 Script Engine 必须考虑：

“可视化表达”。

例如：

不要只写：

“沈星河很震惊。”

应该尽量：

Action：

沈星河抬头望向天空，瞳孔收缩。

Direction：

镜头快速推近他的眼睛。

Dialogue：

沈星河：
“那是什么？”

这样后面才能转换：

Script
→ Storyboard
→ Image Prompt
→ Video Prompt

==================================================
二十五、不要在 Phase 7 实现 Storyboard
==================================================

禁止本阶段实现：

Storyboard

Image Generation

Video Generation

TTS

Voice Clone

Music

Editing

FFmpeg

Timeline Editor

这些留到后续阶段。

但是：

Script 数据结构必须为它们做好输入准备。

==================================================
二十六、Web 页面
==================================================

新增：

/projects/:id/episodes/:episodeId/script

页面结构：

顶部：

Episode 信息

剧本标题

状态

AI 生成

保存

锁定

--------------------------------------------------

左侧：

Scene 列表

Scene 1
Scene 2
Scene 3
...

支持：

新增 Scene
删除 Scene
排序

--------------------------------------------------

中间：

当前 Scene

Scene 信息：

标题
地点
时间
场景目的
冲突
预计时长

下面：

Script Blocks

例如：

[动作]
沈星河抬头望向天空。

[对白]
沈星河：
“那是什么？”

[旁白]
天穹之上，一道裂缝缓缓出现。

支持：

新增 Block
删除 Block
排序
编辑

--------------------------------------------------

右侧：

AI Assistant / AI Generate

显示：

当前 Episode
Story Bible
Characters
Continuity

提供：

生成剧本

重新生成

生成当前 Scene

未来预留：

润色
扩写
缩写
改对白
改变风格

Phase 7 暂时只实现：

生成整集剧本

==================================================
二十七、AI Preview Modal
==================================================

AI 生成后：

不要直接写入页面。

显示：

Script Preview

包括：

剧本标题

概要

预计时长

Scene 数

每 Scene：

标题
概要
Blocks 数量

用户可以：

取消

重新生成

应用剧本

如果已有 Script：

显示警告：

“应用 AI 剧本后将替换当前剧本内容。”

==================================================
二十八、AI Generation History
==================================================

继续使用现有：

GenerationTask

显示：

时间
状态
Provider
Model
耗时

不要显示：

API Key

encryptedApiKey

==================================================
二十九、Mobile
==================================================

Mobile：

项目
→ Season
→ Episode
→ Script

支持：

查看剧本

查看 Scene

查看 Dialogue / Action / Narration

基础编辑。

AI 生成：

可以引导 Web。

不要为了 Mobile 重复实现复杂 Script Editor。

==================================================
三十、Desktop
==================================================

Desktop：

可以查看 Script。

可以基础编辑。

复杂 AI Script Generation 可以继续引导 Web。

保持现有 Desktop App Shell。

==================================================
三十一、API
==================================================

新增：

GET

/projects/:projectId/episodes/:episodeId/script

POST

/projects/:projectId/episodes/:episodeId/script

PATCH

/projects/:projectId/episodes/:episodeId/script

DELETE

/projects/:projectId/episodes/:episodeId/script

Scene：

GET
POST
PATCH
DELETE
reorder

ScriptBlock：

GET
POST
PATCH
DELETE
reorder

AI：

POST

/projects/:projectId/generations/script

继续复用：

GET /projects/:projectId/generations

GET /projects/:projectId/generations/:id

POST /projects/:projectId/generations/:id/apply

==================================================
三十二、权限边界
==================================================

虽然现在还没有：

Login
JWT
User

但是代码结构必须预留：

project ownership

未来：

User
→ Project
→ Script

禁止把权限逻辑写死为：

“所有人都可以访问所有项目”。

至少 Service 层保持：

projectId scope。

==================================================
三十三、错误处理
==================================================

新增错误：

SCRIPT_NOT_FOUND

SCRIPT_ALREADY_EXISTS

EPISODE_NOT_FOUND

GENERATION_NOT_FOUND

GENERATION_NOT_SUCCEEDED

SCRIPT_GENERATION_FAILED

SCRIPT_GENERATION_SCHEMA_INVALID

CHARACTER_NOT_FOUND

PROJECT_EPISODE_MISMATCH

不要把内部异常直接返回。

API Key 不得出现在：

错误
日志
GenerationTask error
API response

==================================================
三十四、Continuity
==================================================

Script Generation 必须调用：

StoryContinuityService

至少检查：

Episode 编号

上一集 storyState

当前 Episode storyState

Character 是否存在

Character 是否属于当前 Project

禁止：

跨项目 Character

不存在的人物作为已知角色

明显违反 Story Bible rules

Phase 7 不需要实现复杂 AI Continuity Checker。

==================================================
三十五、未来成本模型
==================================================

这是非常重要的架构要求。

本项目最终不是由平台永久承担 AI 成本。

最终商业模式：

平台提供软件。

用户承担 AI API 成本。

因此：

所有 AI Generation 都必须经过：

Capability
↓
ProjectAiConfig
↓
Provider
↓
Model

未来：

用户可以配置：

文本模型：

DeepSeek
OpenAI
Claude
Gemini
Qwen

图片模型：

Flux
SDXL
Midjourney API 等

视频模型：

Veo
Kling
Runway
可接入的其他 Provider

TTS：

ElevenLabs
OpenAI
Azure
其他 Provider

本阶段不要实现这些 Provider。

但是：

Script Generation 不得写死任何模型。

==================================================
三十六、未来流水线必须兼容
==================================================

最终目标：

Story Outline
↓
Season Outline
↓
Episode Outline
↓
Script
↓
Scene
↓
Storyboard
↓
Image Prompt
↓
Image
↓
Video Prompt
↓
Video
↓
Voice
↓
Music
↓
Editing
↓
Final Episode

因此 Script Engine 必须成为：

Storyboard Engine 的稳定输入。

尤其要保证：

Scene ID

ScriptBlock ID

Character ID

后续可以被 Storyboard 引用。

不要使用只有文本名称才能关联的设计。

==================================================
三十七、未来 AI 自动化
==================================================

本阶段不要实现 Agent。

但架构必须允许未来：

AI Pipeline Orchestrator

例如：

generateEpisode()

内部：

generateScript()
↓
generateStoryboard()
↓
generateImages()
↓
generateVideos()
↓
generateVoices()
↓
generateMusic()
↓
renderVideo()

现在只实现：

generateScript()

不要提前实现 Worker。

==================================================
三十八、不要实现假的功能
==================================================

如果：

IMAGE
VIDEO
TTS
MUSIC

没有真实 Provider：

显示：

“将在后续阶段开放”。

不要生成假的成功结果。

==================================================
三十九、测试要求
==================================================

必须增加测试。

至少覆盖：

1.

Script CRUD

2.

Project isolation

3.

Episode isolation

4.

Script unique episode

5.

Scene CRUD

6.

Scene ordering

7.

ScriptBlock CRUD

8.

ScriptBlock ordering

9.

Dialogue Character association

10.

Cross-project Character rejection

11.

Script Generation Resolver

12.

STRUCTURED_OUTPUT capability

13.

Invalid JSON → FAILED

14.

Schema invalid → FAILED

15.

Preview 不创建 Script

16.

Apply 创建 Script

17.

Apply 创建 Scene

18.

Apply 创建 ScriptBlock

19.

Apply Transaction rollback

20.

Existing Script replacement confirmation flow

21.

API Key 不泄漏

22.

GenerationTask.appliedAt

23.

Continuity validation

24.

Story Bible context injection

25.

Character context injection

测试不能只测试 happy path。

==================================================
四十、Migration
==================================================

创建新的 migration。

格式：

YYYYMMDDHHMMSS_script_engine

不要修改旧 migration。

不要 reset。

执行：

pnpm exec prisma migrate deploy

确保现有：

星河碰撞

World

Characters

Seasons

Episodes

GenerationTasks

AiProviders

全部保留。

==================================================
四十一、Seed
==================================================

增加：

prisma/seed-script-demo.ts

不要自动执行。

命令：

pnpm db:seed:script-demo

Demo：

星河碰撞

Season 1：

Episode 1：

生成一个完整 Demo Script：

至少：

3 Scene

每 Scene：

至少 3~5 ScriptBlock

包含：

Dialogue

Action

Narration

Direction

并正确关联：

沈星河
太虚真人
艾尔

==================================================
四十二、质量要求
==================================================

完成后必须运行：

pnpm prisma validate

pnpm lint

pnpm typecheck

pnpm test

必须全部通过。

如果测试失败：

自动修复。

不要停下来要求我手动修复。

==================================================
四十三、API 启动
==================================================

不要随意启动新的 NestJS 实例。

当前：

API：

3011

Web：

3010

如果端口已经被占用：

先确认现有服务是否正常。

不要因为端口占用就杀掉正常服务。

==================================================
四十四、完成后的最终检查
==================================================

完成后请检查：

1.
星河碰撞是否仍存在。

2.
World 是否仍存在。

3.
Characters 是否仍存在。

4.
Season / Episode 是否仍存在。

5.
AiProvider 是否仍存在。

6.
GenerationTask 是否仍存在。

7.
已有 World Generation 是否还能工作。

8.
已有 Character Generation 是否还能工作。

9.
Story Bible Generation 是否还能工作。

10.
Season Generation 是否还能工作。

11.
Episode Generation 是否还能工作。

12.
Script Generation 是否能使用：

resolveForCapability(
  projectId,
  STRUCTURED_OUTPUT
)

13.
DeepSeek 是否只是 Provider，而不是硬编码。

==================================================
四十五、自动处理规则
==================================================

你现在处于自动执行模式。

不要在中途停下来让我选择：

A / B / C。

如果存在多个合理实现：

优先选择：

最简单
最稳定
最容易扩展
最符合当前架构
最少破坏已有代码

的方案。

如果发现：

Schema 问题：

自动修复。

如果发现：

TypeScript 问题：

自动修复。

如果发现：

Prisma Migration 问题：

自动修复 migration。

如果发现：

API Client 与 API 不一致：

自动同步。

如果发现：

Web 类型错误：

自动修复。

如果发现：

测试失败：

自动修复并重新测试。

不要修改历史 migration。

不要 reset 数据库。

不要删除用户已有数据。

==================================================
四十六、本阶段明确禁止
==================================================

不要实现：

Login
Register
JWT
Subscription
Billing
Payment

不要实现：

Redis
BullMQ
Worker

不要实现：

Storyboard

Image Generation

Video Generation

TTS

Voice Clone

Music

FFmpeg

Timeline Editor

Auto Publish

YouTube API

TikTok API

Bilibili API

不要新增真实 AI Provider。

不要把 DeepSeek 写死。

不要重建 Monorepo。

不要更换技术栈。

==================================================
四十七、最终报告
==================================================

完成后输出：

1. 修改文件

2. 新增 Prisma Model

3. 新增 Enum

4. Migration 名称

5. API

6. Script 数据结构

7. AI Generation 流程

8. Context Builder

9. Continuity

10. Preview / Apply

11. Web

12. Mobile

13. Desktop

14. Seed

15. 测试数量

16. lint

17. typecheck

18. prisma validate

19. 现有数据是否保留

20. Provider 是否保留

21. GenerationTask 是否保留

22. 当前 API 是否正常

23. 当前 Web 是否正常

24. 是否存在任何遗留问题

25. 下一阶段建议

执行 Phase 7。