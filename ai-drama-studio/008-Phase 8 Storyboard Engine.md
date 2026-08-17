你现在开始执行 AI Drama Studio 的 Phase 8：

# Phase 8：Storyboard Engine —— 从 Script 到 Storyboard

目标：

在现有 AI Drama Studio 架构基础上，实现完整的「剧本 → 分镜」系统。

本阶段重点不是生成图片、视频，而是建立一个稳定、结构化、可编辑、可被后续 Image / Video / TTS / Editing Pipeline 消费的 Storyboard 数据层。

必须严格遵守现有架构，不重建 Monorepo，不更换技术栈，不 reset 数据库，不修改历史 migration。

==================================================
一、项目当前架构必须保持不变
==================================================

继续使用：

- pnpm
- Turborepo
- Nuxt 3 + Vue 3
- NestJS
- Ionic Vue + Capacitor
- Tauri 2
- Prisma
- PostgreSQL
- packages/types
- packages/core
- packages/api-client
- Provider Resolver
- ProjectAiConfig
- AiProvider
- AiModel
- AiCapability
- GenerationTask
- StoryContextBuilder
- StoryContinuityService

Web：
3010

API：
3011

不得更换端口。

不得重新创建项目。

不得删除现有功能。

不得重建 Monorepo。

不得 reset 数据库。

所有数据库变更必须使用新的增量 migration。

==================================================
二、核心产品目标
==================================================

用户现在已经可以：

Project
 ↓
World
 ↓
Characters
 ↓
Story Bible
 ↓
Season
 ↓
Episode
 ↓
Script
 ↓
Scene
 ↓
ScriptBlock

Phase 8 要增加：

Script
 ↓
Storyboard
 ↓
Shot
 ↓
未来：
Image
 ↓
Video
 ↓
Voice / TTS
 ↓
Editing
 ↓
Final Video

因此本阶段必须实现：

「剧本 → AI 分析 → 分镜 Preview → 用户确认 → Storyboard」

注意：

Storyboard 不是 Script 的简单复制。

Script 描述的是：

「故事发生了什么、人物说了什么」

Storyboard 描述的是：

「这一段故事应该如何被拍出来」

例如：

Script：

沈星河走进废弃城市。
他看见天空出现巨大的裂缝。
沈星河：
“这里……究竟发生了什么？”

Storyboard：

Shot 001
- Wide Shot
- 废弃城市
- 沈星河从远处进入画面
- 摄像机缓慢推进
- 天空裂缝逐渐出现
- 6 秒

Shot 002
- Medium Shot
- 沈星河抬头
- 摄像机从背后缓慢推近
- 4 秒

Shot 003
- Extreme Wide Shot
- 巨大宇宙裂缝占据天空
- 城市作为前景
- 5 秒

Shot 004
- Close Up
- 沈星河震惊的表情
- 2 秒

Shot 005
- Medium Close Up
- 沈星河说：
  “这里……究竟发生了什么？”
- 4 秒

因此 Storyboard 必须是独立领域模型。

==================================================
三、必须建立的数据层
==================================================

新增：

Storyboard

StoryboardShot

StoryboardAssetRef（如果合理，可以建立；如果你判断本阶段不需要，可以预留 metadata，不要过度设计）

关系：

Episode
  1
  |
  1
Storyboard
  |
  N
StoryboardShot

Storyboard 必须关联：

episodeId

并保证：

episodeId unique

即：

一个 Episode 对应一个当前 Storyboard。

--------------------------------------------------

Storyboard 字段建议：

id
episodeId
version
status
title
description
totalDurationSeconds
metadata
createdAt
updatedAt

StoryboardStatus：

DRAFT
GENERATING
READY
LOCKED

如果现有项目已有类似状态枚举，可以复用，而不是重复创建。

--------------------------------------------------

StoryboardShot：

id
storyboardId
sceneId
scriptBlockId
shotNumber
shotType
shotSize
cameraMovement
cameraAngle
composition
visualDescription
characterIds
location
action
dialogue
narration
direction
durationSeconds
transition
lighting
mood
visualStyle
imagePrompt
videoPrompt
negativePrompt
continuityNotes
metadata
createdAt
updatedAt

其中：

characterIds 可以使用 Json。

但是：

sceneId
scriptBlockId

必须使用真正的 Foreign Key。

不要把 Scene / ScriptBlock 名称作为唯一关联方式。

--------------------------------------------------

四、Storyboard Shot 的核心设计原则
==================================================

必须保证：

一个 Shot 尽量来源于一个 ScriptBlock。

但允许：

一个 ScriptBlock → 多个 Shot

例如：

ScriptBlock：

“艾尔抬头看向天空。”

可以生成：

Shot 01：
远景，艾尔站在城市废墟中。

Shot 02：
中景，艾尔抬头。

Shot 03：
特写，艾尔眼睛中的天空倒影。

因此：

ScriptBlock 1
   ↓
Shot 1
Shot 2
Shot 3

必须支持这种关系。

不要设计成：

一个 ScriptBlock 只能对应一个 Shot。

==================================================
五、Shot Type 必须结构化
==================================================

建立：

StoryboardShotType

至少支持：

ESTABLISHING
WIDE
FULL
MEDIUM
MEDIUM_CLOSE_UP
CLOSE_UP
EXTREME_CLOSE_UP
OVER_SHOULDER
POV
TWO_SHOT
INSERT
AERIAL
DYNAMIC

如果项目已有类似枚举，不要重复。

==================================================
六、Shot Size
==================================================

建立：

StoryboardShotSize

支持：

EXTREME_WIDE
WIDE
FULL
MEDIUM
MEDIUM_CLOSE_UP
CLOSE_UP
EXTREME_CLOSE_UP

==================================================
七、Camera Movement
==================================================

建立：

CameraMovement

支持：

STATIC
PAN
TILT
DOLLY_IN
DOLLY_OUT
TRUCK_LEFT
TRUCK_RIGHT
CRANE_UP
CRANE_DOWN
ZOOM_IN
ZOOM_OUT
HANDHELD
ORBIT
FOLLOW
TRACKING

不要只保存自由文本。

可以同时保留：

cameraMovement
cameraMovementParams

例如：

{
  "speed": "slow",
  "direction": "forward"
}

==================================================
八、Camera Angle
==================================================

支持：

EYE_LEVEL
LOW_ANGLE
HIGH_ANGLE
BIRDS_EYE
WORMS_EYE
DUTCH_ANGLE
OVERHEAD

==================================================
九、Transition
==================================================

支持：

CUT
FADE_IN
FADE_OUT
DISSOLVE
WIPE
MATCH_CUT
SMASH_CUT

==================================================
十、Storyboard 与未来 AI 图片 / 视频生成的关系
==================================================

这是非常重要的架构要求。

本阶段不要实现真实 Image Generation。

也不要实现真实 Video Generation。

但是 StoryboardShot 必须提前拥有未来生成所需要的 Prompt 字段：

imagePrompt
videoPrompt
negativePrompt

以及：

visualDescription
visualStyle
lighting
mood
composition
characterIds
location
durationSeconds
cameraMovement

未来：

StoryboardShot
 ↓
Image Generation
 ↓
Shot Image

以及：

StoryboardShot
 ↓
Video Generation
 ↓
Shot Video

因此不要让 Image Generation 将来重新解析 Script。

Storyboard 必须成为：

「视觉生产的唯一上游输入」。

==================================================
十一、GenerationTask
==================================================

新增：

GenerationTaskType.STORYBOARD

如果枚举中不存在，则增量添加。

capability：

STRUCTURED_OUTPUT

生成流程：

POST /projects/:projectId/generations/storyboard

↓

ProviderResolver

↓

resolveForCapability(
  projectId,
  STRUCTURED_OUTPUT
)

↓

StoryContextBuilder

↓

Storyboard Prompt

↓

AI

↓

JSON Parse

↓

Schema Validation

↓

GenerationTask.output

↓

Preview

↓

用户 Apply

↓

Transaction

↓

Storyboard + StoryboardShot

==================================================
十二、绝对不能直接写数据库
==================================================

AI 生成 Storyboard 时：

禁止：

AI → 直接创建 Storyboard

必须：

AI
 ↓
GenerationTask.output
 ↓
Preview
 ↓
用户确认
 ↓
Apply
 ↓
Transaction

Preview 阶段：

不得创建 StoryboardShot。

Apply 阶段：

使用：

prisma.$transaction

一次性：

1. 创建 / 更新 Storyboard
2. 删除旧 Shots
3. 创建新 Shots
4. 更新 version
5. 更新 GenerationTask.appliedAt

任何一步失败：

全部 rollback。

==================================================
十三、Storyboard Version
==================================================

必须支持版本。

例如：

第一次生成：

version = 1

第二次生成：

version = 2

第三次生成：

version = 3

但是不要删除 GenerationTask。

GenerationTask 本身就是生成历史。

Storyboard 保存当前生效版本。

未来可以扩展：

StoryboardVersion

但是 Phase 8 不需要建立复杂版本表。

当前：

Storyboard.version

即可。

==================================================
十四、AI Context
==================================================

必须继续使用：

StoryContextBuilder

增加：

buildStoryboardContext(projectId, episodeId)

必须包含：

Project
Story Bible
World
Characters
Season
Episode
Script
Scenes
ScriptBlocks
Previous Episode State

但是必须控制 Context 大小。

不要把整个数据库直接 JSON.stringify 给 AI。

必须进行摘要化。

==================================================
十五、Storyboard Context 必须包含
==================================================

Project：

name
genre
description

Story Bible：

logline
premise
theme
tone
style
storyPromise
rules
continuityNotes

World：

name
description
cosmicBackground
coreConflict

Characters：

id
name
role
identity
appearanceProfile
personalityProfile
abilities
goal
conflict
civilization
faction

Season：

number
title
synopsis

Episode：

number
title
outline
storyState
continuityNotes

Script：

version
status

Scenes：

number
title
location
time
description

ScriptBlocks：

type
character
content
direction

上一集：

storyState
continuityNotes

==================================================
十六、非常重要：人物视觉信息
==================================================

Storyboard 是未来 Image / Video Generation 的上游。

所以：

Character.imageProfile

应该允许进入 Storyboard Context。

但是：

不要把整个 Character.metadata 全部塞进去。

只提取视觉相关信息，例如：

appearance
hair
face
body
clothing
colors
visualTraits
age
species
etc.

如果 imageProfile 是 Json：

进行结构化摘要。

==================================================
十七、AI Prompt
==================================================

建立：

storyboard-generation.prompt.ts

要求 AI：

你是一名专业动画导演、分镜师、摄影指导。

根据：

Story Bible
World
Characters
Episode
Script

将完整剧本转换为可执行 Storyboard。

必须：

1. 不改变原故事
2. 不新增未经剧本支持的主要剧情
3. 不改变角色关系
4. 不改变角色身份
5. 不随意改变场景地点
6. 不随意修改对白
7. 不删除关键对白
8. 不改变事件顺序
9. 可以将一个 ScriptBlock 拆成多个 Shot
10. 每个 Shot 必须有明确视觉目的
11. 必须考虑镜头连续性
12. 必须考虑人物位置连续性
13. 必须考虑视线方向
14. 必须考虑镜头轴线
15. 必须考虑景别变化
16. 避免连续镜头全部使用同一种景别
17. 对关键剧情使用视觉强化
18. 对对白设计适合口型与镜头的构图
19. 为未来 Image Generation 生成 imagePrompt
20. 为未来 Video Generation 生成 videoPrompt

==================================================
十八、AI 必须输出严格 JSON
==================================================

根结构：

{
  "storyboard": {
    "title": "...",
    "description": "...",
    "totalDurationSeconds": 0
  },
  "shots": []
}

shots：

{
  "shotNumber": 1,
  "sceneNumber": 1,
  "scriptBlockIds": [],
  "shotType": "WIDE",
  "shotSize": "WIDE",
  "cameraMovement": "DOLLY_IN",
  "cameraAngle": "EYE_LEVEL",
  "composition": "...",
  "visualDescription": "...",
  "characterIds": [],
  "location": "...",
  "action": "...",
  "dialogue": "...",
  "narration": "...",
  "direction": "...",
  "durationSeconds": 5,
  "transition": "CUT",
  "lighting": "...",
  "mood": "...",
  "visualStyle": "...",
  "imagePrompt": "...",
  "videoPrompt": "...",
  "negativePrompt": "...",
  "continuityNotes": "..."
}

==================================================
十九、Schema Validation
==================================================

必须建立严格 Schema。

禁止：

unknown fields

必须检查：

shotNumber
sceneNumber
scriptBlockIds
shotType
shotSize
cameraMovement
cameraAngle
visualDescription
durationSeconds

durationSeconds：

必须 > 0

shotNumber：

必须连续。

例如：

1
2
3
4
5

不能：

1
3
7

==================================================
二十、生成前校验
==================================================

调用：

StoryContinuityService.validateEpisodeContinuity()

同时检查：

Episode 存在

Episode 属于 Project

Script 存在

Script 属于 Episode

Scene 属于 Script

ScriptBlock 属于 Scene

Character 属于 Project

禁止跨项目引用。

如果 Episode 没有 Script：

返回明确错误：

SCRIPT_REQUIRED_FOR_STORYBOARD

不要调用 AI。

==================================================
二十一、Apply 时必须验证引用
==================================================

AI 输出中的：

sceneNumber
scriptBlockIds
characterIds

全部必须重新验证。

绝不能相信 AI 返回的数据。

例如 AI 返回：

characterId = xxx

必须确认：

xxx 属于当前 Project。

否则：

STORYBOARD_INVALID_CHARACTER_REFERENCE

并 rollback。

==================================================
二十二、Storyboard API
==================================================

实现：

GET
/projects/:projectId/episodes/:episodeId/storyboard

POST
/projects/:projectId/episodes/:episodeId/storyboard

PATCH
/projects/:projectId/episodes/:episodeId/storyboard

DELETE
/projects/:projectId/episodes/:episodeId/storyboard

Storyboard Shot：

GET
/projects/:projectId/episodes/:episodeId/storyboard/shots

POST
/projects/:projectId/episodes/:episodeId/storyboard/shots

PATCH
/projects/:projectId/episodes/:episodeId/storyboard/shots/:shotId

DELETE
/projects/:projectId/episodes/:episodeId/storyboard/shots/:shotId

POST
/projects/:projectId/episodes/:episodeId/storyboard/shots/reorder

==================================================
二十三、Generation API
==================================================

新增：

POST
/projects/:projectId/generations/storyboard

复用：

GET
/projects/:projectId/generations

GET
/projects/:projectId/generations/:id

POST
/projects/:projectId/generations/:id/apply

apply 根据：

GenerationTask.type

分支：

WORLD
CHARACTER
STORY_BIBLE
SEASON_OUTLINE
EPISODE_OUTLINE
SCRIPT
STORYBOARD

不要破坏已有 Apply。

==================================================
二十四、GenerationTask metadata
==================================================

Storyboard GenerationTask：

type = STORYBOARD

capability = STRUCTURED_OUTPUT

usage：

{
  "durationMs": 1234,
  "shotCount": 18,
  "sceneCount": 5
}

不要伪造 token usage。

如果 Provider 当前没有返回 token usage：

不要假装有。

==================================================
二十五、Web UI
==================================================

新增：

/projects/:id/episodes/:episodeId/storyboard

页面结构：

顶部：

Episode Title
Storyboard Status
Version
Total Duration
AI Generate
Lock / Unlock

左侧：

Scene List

Scene 01
Scene 02
Scene 03

中间：

Shot Timeline / Shot List

Shot 001
Shot 002
Shot 003
...

右侧：

Shot Inspector

显示：

镜头号
场景
Script Block
景别
镜头类型
机位
运镜
构图
视觉描述
人物
地点
动作
对白
旁白
时长
转场
灯光
情绪
视觉风格
Image Prompt
Video Prompt
Negative Prompt
Continuity Notes

==================================================
二十六、Storyboard UI 不要做成普通 CRUD 表格
==================================================

这是一个创作工具。

优先使用：

Card
Timeline
Shot List
Inspector
Preview

而不是：

巨大 Table

目标：

以后用户可以像使用剪辑软件 / 分镜软件一样管理 Storyboard。

但是 Phase 8 不实现真正的视频时间线编辑器。

==================================================
二十七、AI Generate UI
==================================================

点击：

AI 生成分镜

Modal：

生成范围：

整集
当前 Scene

注意：

如果当前 Phase 8 实现「当前 Scene」需要明显增加复杂度，可以只实现整集。

建议：

Phase 8 v1 只实现：

整集生成。

保留未来：

单 Scene
选中 Scene
重新生成选中 Shot
润色 Shot
扩展 Shot
缩短 Shot
重新设计镜头

==================================================
二十八、Preview
==================================================

AI 生成后：

不要直接 Apply。

显示：

Storyboard Preview

例如：

Scene 01

Shot 001
WIDE
5s

Shot 002
MEDIUM
4s

Shot 003
CLOSE UP
3s

...

用户可以：

Apply
Regenerate
Cancel

如果已有 Storyboard：

明确提示：

「应用新的 AI 分镜将替换当前分镜内容，并生成新版本，是否继续？」

==================================================
二十九、Manual Editing
==================================================

必须支持基础手工编辑：

修改 Shot：

shotType
shotSize
cameraMovement
cameraAngle
composition
visualDescription
durationSeconds
transition
lighting
mood
visualStyle
imagePrompt
videoPrompt
negativePrompt
continuityNotes

支持：

新增 Shot
删除 Shot
修改 Shot
调整 Shot 顺序

禁止修改：

shot.id

sceneId

scriptBlockId

characterId 的归属关系必须经过校验。

==================================================
三十、Script → Storyboard 映射
==================================================

必须在 UI 中让用户看到：

这个 Shot 来源于哪个：

Scene

ScriptBlock

例如：

Shot 03
来源：
Scene 01
ScriptBlock #4

并允许点击跳回 Script。

这样以后：

Storyboard
→ Script

可以互相跳转。

==================================================
三十一、Storyboard 不允许修改 Script
==================================================

非常重要。

Storyboard 是 Script 的下游。

修改 Storyboard：

不能自动修改 Script。

如果用户想修改故事：

回到 Script。

然后：

Script 修改
 ↓
Storyboard 标记 STALE

为未来预留：

StoryboardStatus.STALE

如果现在增加 STALE 状态不会造成复杂度，可以加入。

否则使用 metadata：

{
  "sourceScriptVersion": 1
}

Storyboard：

sourceScriptVersion

当 Script version 改变：

Storyboard 可以检测：

current Script version != sourceScriptVersion

然后 UI 显示：

「剧本已更新，当前分镜可能已过期」

建议实现。

==================================================
三十二、非常重要：未来图片 / 视频生产必须考虑
==================================================

本阶段不要实现：

IMAGE

VIDEO

TTS

VOICE CLONE

MUSIC

EDITING

但是数据结构必须保证未来可以做到：

StoryboardShot
 ↓
Image Generation Task
 ↓
Image Asset
 ↓
Video Generation Task
 ↓
Video Asset

因此：

StoryboardShot 必须是未来视觉生成任务的稳定输入。

不要让未来 Image Generation 再重新读取 Script。

==================================================
三十三、Asset 关系预留
==================================================

如果当前 Asset 模型可以安全扩展：

增加：

StoryboardShotAsset

或者使用已有 Asset.metadata 预留：

sourceType
sourceId
generationTaskId

不要现在实现图片生成。

目标是以后能够表达：

Shot
 ├── Reference Image
 ├── Generated Image
 ├── Video
 ├── Voice
 └── Music

但是本阶段只做数据层兼容。

==================================================
三十四、未来 Image Generation 的 Capability
==================================================

不要现在实现真实 Image Provider。

但是必须确保：

AiCapability.IMAGE

继续存在。

未来：

StoryboardShot
 ↓
resolveForCapability(
  projectId,
  IMAGE
)

不要出现：

Storyboard → DeepSeek

这种错误。

==================================================
三十五、未来 Video Generation
==================================================

继续保持：

AiCapability.VIDEO
AiCapability.IMAGE_TO_VIDEO

未来：

IMAGE_TO_VIDEO：

StoryboardShot
+
Reference Image
+
Video Prompt

↓

Video Provider

VIDEO：

纯文本 / Image / Reference

↓

Video Provider

本阶段只做架构兼容。

==================================================
三十六、成本模型
==================================================

必须继续遵守 BYOK / User Pays 原则。

平台默认 Provider：

只用于：

开发
Demo
首次体验

不是长期平台代付。

未来：

User
 ↓
自己的 Provider
 ↓
自己的 API Key
 ↓
自己的 AI 成本

Storyboard Generation 必须通过：

ProjectAiConfig

Resolver。

不能：

写死 DeepSeek

不能：

写死 OpenAI

不能：

写死任何模型。

==================================================
三十七、不要在本阶段实现 Billing
==================================================

不要实现：

充值
余额
积分
订阅
Stripe
支付

但是 GenerationTask 必须继续记录：

provider
model
capability
usage
createdAt
durationMs

为未来成本统计做准备。

==================================================
三十八、测试
==================================================

必须增加测试。

至少覆盖：

1. Storyboard CRUD

2. Project isolation

3. Episode isolation

4. Script 不存在时生成失败

5. Cross-project Scene rejected

6. Cross-project ScriptBlock rejected

7. Cross-project Character rejected

8. Provider Resolver 使用 STRUCTURED_OUTPUT

9. 非法 JSON → FAILED

10. Schema validation failure → FAILED

11. Preview 不创建 Storyboard

12. Apply 创建 Storyboard

13. Apply 创建 Shots

14. Apply transaction rollback

15. Existing Storyboard version + 1

16. Existing Storyboard replacement confirmation

17. Shot reorder

18. Shot CRUD

19. ScriptBlock → Shot 一对多

20. sourceScriptVersion

21. Script version 变化后 Storyboard stale detection

22. API Key 不泄漏

23. GenerationTask capability = STORYBOARD / STRUCTURED_OUTPUT

24. AI 返回不存在的 Character → rejected

25. AI 返回不存在的 ScriptBlock → rejected

26. shotNumber 非连续 → rejected

27. durationSeconds 非法 → rejected

28. Project Provider 配置正确使用

29. Legacy Provider Resolver 兼容

30. System Provider fallback 兼容

==================================================
三十九、Seed
==================================================

新增：

prisma/seed-storyboard-demo.ts

命令：

pnpm db:seed:storyboard-demo

不要自动执行。

使用：

星河碰撞
Season 1
Episode 1
已有 Script

创建一套 Demo Storyboard：

至少：

3 Scene

每 Scene 3–6 Shots

总计至少 12 Shots

包含：

WIDE
MEDIUM
CLOSE_UP
OVER_SHOULDER

包含：

STATIC
DOLLY_IN
PAN
TRACKING

包含：

不同 duration

包含：

不同 transition

Shot 必须正确关联：

Scene
ScriptBlock
Character

==================================================
四十、Mobile
==================================================

Mobile：

项目
→ Season
→ Episode
→ Storyboard

提供：

Shot List
Shot Detail
基础编辑

不要复制 Desktop / Web 的复杂工作台。

AI Generate：

引导 Web。

==================================================
四十一、Desktop
==================================================

Desktop：

显示 Storyboard。

提供：

Scene
Shot List
Shot Inspector

支持基础编辑。

复杂 AI Generate 可以引导 Web。

不要实现复杂原生视频编辑器。

==================================================
四十二、错误处理
==================================================

必须使用统一 AppError。

例如：

SCRIPT_REQUIRED_FOR_STORYBOARD

STORYBOARD_NOT_FOUND

STORYBOARD_SHOT_NOT_FOUND

STORYBOARD_INVALID_SCENE

STORYBOARD_INVALID_SCRIPT_BLOCK

STORYBOARD_INVALID_CHARACTER

STORYBOARD_SCHEMA_INVALID

STORYBOARD_GENERATION_FAILED

STORYBOARD_APPLY_FAILED

STORYBOARD_STALE

不得返回：

数据库原始错误

API Key

Provider Secret

内部 Stack Trace

==================================================
四十三、性能
==================================================

Storyboard 生成可能产生很多 Shot。

因此：

不要把所有 GenerationTask 全部加载进页面。

Shot List 支持合理分页 / lazy loading。

Context Builder 必须控制 Prompt 大小。

数据库查询使用：

select

include

必要索引。

不要无脑：

include 所有关系。

==================================================
四十四、不要过度设计
==================================================

本阶段明确不要实现：

❌ Image Generation

❌ Video Generation

❌ TTS

❌ Voice Clone

❌ Music

❌ Editing

❌ Timeline Video Editor

❌ Redis

❌ BullMQ

❌ Worker

❌ Agent

❌ RAG

❌ Embedding

❌ Billing

❌ Payment

❌ Login

❌ Register

❌ JWT

❌ Subscription

❌ 自动发布

❌ YouTube API

❌ TikTok API

❌ Bilibili API

❌ 新 AI Provider

不要因为发现未来需求而提前实现这些功能。

只需要保证架构兼容。

==================================================
四十五、自动发现现有代码
==================================================

在修改之前：

1. 检查当前 Prisma schema
2. 检查 GenerationTask
3. 检查 AiCapability
4. 检查 ProviderResolver
5. 检查 StoryContextBuilder
6. 检查 StoryContinuityService
7. 检查 Script / Scene / ScriptBlock
8. 检查 GenerationController
9. 检查 Generation Apply
10. 检查现有 Web Episode / Script 页面

不要重复创建已有功能。

如果现有实现与本 Prompt 略有不同：

优先复用现有架构。

不要为了严格匹配 Prompt 而重构已经稳定的模块。

==================================================
四十六、Migration
==================================================

必须新增：

prisma/migrations/<timestamp>_storyboard_engine/migration.sql

不得修改：

历史 migration。

执行：

pnpm prisma validate

pnpm exec prisma migrate deploy

==================================================
四十七、Shared Packages
==================================================

新增类型必须进入：

packages/types

业务纯逻辑进入：

packages/core

配置进入：

packages/config

API 请求进入：

packages/api-client

禁止 Web 自己重新定义：

Storyboard
StoryboardShot
ShotType
CameraMovement
CameraAngle

==================================================
四十八、API Client
==================================================

必须增加：

getEpisodeStoryboard()
createStoryboard()
updateStoryboard()
deleteStoryboard()

getStoryboardShots()
createStoryboardShot()
updateStoryboardShot()
deleteStoryboardShot()
reorderStoryboardShots()

createStoryboardGeneration()

getGeneration()
applyGeneration()

Web 禁止直接 fetch API。

==================================================
四十九、质量要求
==================================================

完成后必须运行：

pnpm prisma validate

pnpm lint

pnpm typecheck

pnpm test

如果存在测试失败：

必须自己修复。

如果存在 TypeScript 错误：

必须自己修复。

如果存在 ESLint 错误：

必须自己修复。

不要停在：

“功能基本完成”。

==================================================
五十、最终验收标准
==================================================

Phase 8 完成后，我应该能够：

1. 打开：

星河碰撞

2. 进入：

Season 1

3. 打开：

Episode 1

4. 打开：

Script

5. 确认 Script 存在

6. 进入：

Storyboard

7. 点击：

AI 生成分镜

8. AI 使用：

ProjectAiConfig

9. Provider Resolver：

STRUCTURED_OUTPUT

10. AI 读取：

Story Bible
World
Characters
Season
Episode
Script
Previous Episode

11. AI 输出结构化 Storyboard

12. Schema Validation

13. Preview

14. 我可以查看：

Scene
Shot
景别
镜头
运镜
人物
动作
对白
时长
视觉描述
Image Prompt
Video Prompt

15. 点击：

应用

16. 数据进入：

Storyboard

StoryboardShot

17. 刷新页面：

数据仍然存在。

18. 修改 Shot：

修改成功。

19. 拖动排序：

Shot 顺序正确。

20. 修改 Script version：

Storyboard 显示：

「剧本已更新，当前分镜可能已过期」

21. 再次 AI 生成：

生成新的 GenerationTask

22. Preview：

旧 Storyboard 不受影响。

23. Apply：

Storyboard version + 1

24. 旧 GenerationTask：

仍然保留。

25. API Key：

任何 API 返回、日志、错误都不能泄漏。

==================================================
五十一、架构原则
==================================================

最终整个 AI Drama Studio 的生产链必须逐渐形成：

用户故事大纲
        ↓
Story Bible
        ↓
World
        ↓
Characters
        ↓
Season
        ↓
Episodes
        ↓
Script
        ↓
Storyboard
        ↓
Image
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
        ↓
Final Season
        ↓
Publish

其中：

Story Bible
= 创作规则

World
= 世界事实

Character
= 人物事实

Episode
= 故事结构

Script
= 文本剧本

Storyboard
= 视觉导演方案

Image
= 视觉资产

Video
= 动态资产

Voice
= 声音资产

Editing
= 最终时间线

每一层都必须有明确职责。

不要让某一层承担下一层职责。

尤其：

Script ≠ Storyboard

Storyboard ≠ Image

Image ≠ Video

Video ≠ Editing

==================================================
五十二、最终输出
==================================================

完成后请汇报：

1. 修改了哪些文件
2. 新增哪些 Prisma Model
3. 新增哪些 Enum
4. Migration 名称
5. 是否执行 Migration
6. Storyboard 数据结构
7. Shot 数据结构
8. Script → Shot 映射方式
9. AI Generation 流程
10. Context Builder
11. Continuity 校验
12. Provider Resolver
13. API
14. Web
15. Mobile
16. Desktop
17. Seed
18. 测试数量
19. lint
20. typecheck
21. prisma validate
22. 现有数据是否保留
23. GenerationTask 是否保留
24. Provider 是否保留
25. 是否修改 Monorepo
26. 本阶段明确没有实现什么
27. 当前 API 是否正常
28. 当前 Web 是否正常
29. 是否存在遗留问题
30. 下一阶段建议

执行 Phase 8。

不要等待我的确认。

如果发现明显的小问题，在不改变架构的情况下直接修复。

如果发现未来阶段需要的架构兼容点，可以增加最小必要字段或接口，但不要提前实现未来功能。

Phase 8 的目标是：

建立一个真正可用于后续 Image / Video / TTS / Editing 的稳定 Storyboard Engine。