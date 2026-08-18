你现在开始执行 AI Drama Studio：

# Phase 12 — Music / SFX Engine v1

这是一个已经连续完成 Phase 4.5 → Phase 4.6 → Phase 5 → Phase 6 → Phase 7 → Phase 8 → Phase 9 → Phase 10 → Phase 11 的成熟 Monorepo 项目。

本阶段目标：

建立统一的：

Music / SFX Generation Engine v1

用于生成：

1. Background Music / BGM
2. Sound Effect / SFX

本阶段必须继续遵循现有 AI Provider / Capability / ProjectAiConfig / GenerationTask / Asset 架构。

==================================================
一、最高优先级约束
==================================================

严格遵守以下规则：

1. 不重建 Monorepo
2. 不更换技术栈
3. 不删除已有业务模型
4. 不 reset 数据库
5. 不修改历史 migration
6. 只能新增增量 migration
7. 不重新实现 Provider Architecture
8. 不绕过 ProviderResolver
9. 不在业务代码读取：
   AI_API_KEY
   AI_BASE_URL
   AI_MODEL
10. 不写死 DeepSeek / OpenAI / 任意模型
11. 不把 CHAT Provider 当作 Music / SFX Provider
12. Music / SFX 必须使用独立 Capability
13. Generate 阶段绝对不能直接创建正式 Asset
14. 必须：
    GenerationTask
    → Preview
    → 用户确认
    → Apply
15. Apply 必须使用 Prisma transaction
16. API Key 只能服务端解密
17. 前端绝对不能拿到 API Key
18. 日志、异常、错误响应必须脱敏
19. 不伪造真实 AI 成功
20. 没有真实 Music / SFX Provider 时必须明确返回能力未配置 / 未支持
21. 不实现 Billing
22. 不实现 Login / Register / JWT / Subscription
23. 不实现 Redis / BullMQ / Worker
24. 不实现 Timeline
25. 不实现 FFmpeg
26. 不实现混音
27. 不实现 Episode Render
28. 不实现整集成片
29. 不实现发布平台
30. 不实现 Agent / RAG / Embedding
31. 不提前实现 Phase 13 以后功能

==================================================
二、首先检查现有项目
==================================================

开始编码前：

1. 阅读现有 Prisma schema
2. 阅读：
   AiCapability
   AiProvider
   AiModel
   AiProviderCapability
   ProjectAiConfig
3. 阅读 ProviderResolver
4. 阅读 GenerationTask
5. 阅读 Asset
6. 阅读 StoryboardShotAsset
7. 阅读 ScriptBlockAsset
8. 阅读现有：
   ImageGenerationService
   VideoGenerationService
   TtsGenerationService
9. 阅读：
   OpenAiCompatibleProvider
10. 阅读：
   AssetStorageService
11. 阅读：
   Generation Apply 总入口
12. 阅读：
   Web AI Settings
13. 阅读：
   Project Settings
14. 阅读：
   `/projects/:id/images`
   `/projects/:id/videos`
   `/projects/:id/voices`
15. 阅读现有测试结构

不要复制旧架构。

Music / SFX 必须复用已有基础设施。

如果现有实现与下面要求存在差异：

优先保持现有架构兼容，再增量扩展。

==================================================
三、本阶段最终架构
==================================================

目标架构：

Music:

User
 ↓
Project
 ↓
Music Generation Request
 ↓
AiCapability.MUSIC
 ↓
ProjectAiConfig
 ↓
ProviderResolver
 ↓
Music Provider Adapter
 ↓
GenerationTask(type=MUSIC)
 ↓
Preview
 ↓
Apply
 ↓
Asset(type=AUDIO)
 ↓
Music Asset Relation


SFX:

User
 ↓
Project
 ↓
SFX Generation Request
 ↓
AiCapability.SFX
 ↓
ProjectAiConfig
 ↓
ProviderResolver
 ↓
SFX Provider Adapter
 ↓
GenerationTask(type=SFX)
 ↓
Preview
 ↓
Apply
 ↓
Asset(type=AUDIO)
 ↓
SFX Asset Relation

==================================================
四、AiCapability
==================================================

如果当前 AiCapability 中不存在：

MUSIC
SFX

则增量添加：

MUSIC
SFX

不要修改已有：

CHAT
STRUCTURED_OUTPUT
IMAGE
VIDEO
IMAGE_TO_VIDEO
TTS
VOICE_CLONE
EMBEDDING

必须保持向后兼容。

注意：

MUSIC 和 SFX 是两个不同 Capability。

不要：

MUSIC → TTS fallback

不要：

SFX → TTS fallback

不要：

MUSIC → CHAT fallback

不要：

SFX → IMAGE fallback

任何情况下都不能跨 Capability fallback。

==================================================
五、GenerationTask
==================================================

新增：

GenerationTaskType.MUSIC
GenerationTaskType.SFX

如果已经存在占位 enum：

直接复用。

不要重复创建。

GenerationTask：

MUSIC:
capability = MUSIC

SFX:
capability = SFX

继续使用现有：

status
input
output
error
usage
provider
model
appliedAt
createdAt
updatedAt

不要创建：

MusicGenerationTask

SfxGenerationTask

避免重复模型。

==================================================
六、Asset
==================================================

继续复用：

Asset(type = AUDIO)

不要创建：

MusicAsset
SfxAsset

Asset 应记录：

type
status
mimeType
storageKey
thumbnailUrl
durationSeconds
sizeBytes
provider
model
version
generationTaskId
metadata

Music / SFX 都是：

AssetType.AUDIO

通过 metadata / relation 区分用途。

==================================================
七、Music / SFX Relation
==================================================

不要把 Music / SFX 直接塞进 Episode Json。

新增明确关系模型。

推荐：

EpisodeAudioAsset

字段：

id
episodeId
assetId
role
isPrimary
metadata
createdAt

唯一：

episodeId + assetId

并增加：

AudioAssetRole：

MUSIC
SFX
REFERENCE
FINAL

如果现有 Asset relation 体系更适合复用，则优先复用现有结构，但必须保证：

1. Episode 可以关联 Music
2. Episode 可以关联 SFX
3. 一个 Asset 可以被历史保留
4. 不删除历史生成
5. 可以标记 Primary
6. 可以设置 Final
7. 不影响 ScriptBlockAsset
8. 不影响 StoryboardShotAsset

不要破坏现有：

StoryboardShotAsset

ScriptBlockAsset

==================================================
八、Music 数据结构
==================================================

Music Generation Input 至少支持：

episodeId
prompt
style
mood
genre
durationSeconds
instrumentation
tempo
language
isInstrumental

可选：

title
negativePrompt
loopable
intensity

注意：

不要强制 AI 返回无法可靠获得的数据。

如果 Provider 只返回音频：

仍然允许成功。

metadata 可以记录：

{
  "type": "music",
  "style": "...",
  "mood": "...",
  "genre": "...",
  "durationSeconds": 30
}

==================================================
九、SFX 数据结构
==================================================

SFX Generation Input 至少支持：

episodeId
prompt
durationSeconds
category
intensity

例如：

category：

impact
weapon
magic
explosion
environment
mechanical
footstep
door
wind
fire
water
creature
technology
ui
other

不要把 category 限制得过死。

允许：

other

metadata：

{
  "type": "sfx",
  "category": "...",
  "intensity": "..."
}

==================================================
十、Music / SFX 的上下文
==================================================

Music 和 SFX 不应该从零生成。

必须允许 AI 获取当前故事上下文。

新增：

StoryContextBuilder.buildMusicContext()

StoryContextBuilder.buildSfxContext()

但必须遵守现有 Context Builder 原则：

只提供摘要。

不要：

JSON.stringify 全数据库

不要发送：

API Key
encryptedApiKey
GenerationTask 全量
imageProfile 全量
voiceProfile 全量
metadata 大对象

Music Context 可以包含：

Project
Story Bible
World summary
Season
Episode
Episode outline
Episode storyState
Episode continuityNotes
Script summary
Storyboard summary

SFX Context 可以额外包含：

相关 Scene
相关 StoryboardShot
shot visualDescription
shot action
shot environment
camera / transition 等必要信息

但是不要直接把所有数据库内容 dump 给模型。

==================================================
十一、Music Generation API
==================================================

新增：

POST

/projects/:projectId/generations/music

请求至少：

{
  episodeId,
  prompt,
  durationSeconds,
  style?,
  mood?,
  genre?,
  instrumentation?,
  tempo?,
  isInstrumental?,
  negativePrompt?
}

校验：

Project
→ Episode
→ Season
→ 所属关系

全部必须属于当前 Project。

禁止跨项目引用。

==================================================
十二、SFX Generation API
==================================================

新增：

POST

/projects/:projectId/generations/sfx

请求至少：

{
  episodeId,
  prompt,
  durationSeconds,
  category?,
  intensity?,
  negativePrompt?
}

同样执行：

Project
→ Episode

归属校验。

==================================================
十三、Provider Resolver
==================================================

Music：

resolveForCapability(
  projectId,
  AiCapability.MUSIC
)

SFX：

resolveForCapability(
  projectId,
  AiCapability.SFX
)

优先级必须保持现有架构：

ProjectAiConfig
→ Legacy（仅适用于兼容的文本能力）
→ User Provider
→ Platform Default
→ System Provider
→ NO_AI_PROVIDER_CONFIGURED

但是：

MUSIC 不能 fallback 到 CHAT。

SFX 不能 fallback 到 CHAT。

TTS 不能 fallback 到 MUSIC。

IMAGE 不能 fallback 到 MUSIC。

任何 Capability 都必须严格隔离。

如果没有 MUSIC：

MUSIC_PROVIDER_NOT_CONFIGURED

如果没有 SFX：

SFX_PROVIDER_NOT_CONFIGURED

如果 Provider 存在但不支持：

CAPABILITY_NOT_SUPPORTED

==================================================
十四、Provider Adapter
==================================================

新增抽象：

generateMusic()

generateSfx()

如果现有：

AiProvider

interface

允许增加：

generateMusic(input)
generateSfx(input)

不要破坏已有：

generateImage
generateVideo
generateSpeech
generateWith

==================================================
十五、OpenAI Compatible Music / SFX
==================================================

不要假设所有 OpenAI Compatible Provider 都支持：

Music
SFX

因此：

只有 Provider 明确声明支持 MUSIC / SFX：

才允许调用。

不要因为：

baseUrl + apiKey

存在就认为支持。

如果 Provider 没有真实统一协议：

必须返回：

CAPABILITY_NOT_SUPPORTED

而不是伪造 URL。

不要凭空创建：

/music/generations

/sfx/generations

如果当前没有可靠统一协议。

可以建立：

MusicProviderAdapter

SfxProviderAdapter

并定义：

协议：

openai-compatible-music-v1

openai-compatible-sfx-v1

但必须要求 Provider 明确配置 adapter / protocol。

==================================================
十六、Provider UI
==================================================

更新：

/ai-providers

Provider Form

能力选择：

Chat
Structured Output
Image
Video
Image to Video
TTS
Music
SFX
...

Music / SFX 必须单独显示。

例如：

☑ Chat
☑ Structured Output
☑ Image
☐ Video
☐ Image to Video
☐ TTS
☐ Music
☐ SFX

不要自动勾选。

Provider 没有真实支持时：

不能仅因为用户勾选就假装支持。

保存时：

至少验证 capability declaration。

真实调用失败时：

返回明确：

CAPABILITY_NOT_SUPPORTED

==================================================
十七、Project AI Settings
==================================================

更新：

/projects/:id/settings

增加：

Music
SFX

配置卡片。

例如：

Music
Provider: 未配置
Model: —
Status: Not configured

[配置]

SFX
Provider: 未配置
Model: —
Status: Not configured

[配置]

必须继续强调：

AI 费用由用户自己的 Provider 账户承担。

平台默认 Provider 只是：

Demo / Development fallback

不是长期免费代付。

==================================================
十八、ProjectAiConfig
==================================================

使用现有：

ProjectAiConfig

capability：

MUSIC

SFX

设置：

PATCH

/projects/:projectId/ai-config/MUSIC

PATCH

/projects/:projectId/ai-config/SFX

继续复用现有：

providerId
modelId

校验：

Provider 存在
Provider enabled
Provider 有 Key
Provider 支持 MUSIC / SFX
Model 属于 Provider
Model enabled
Model 支持对应 capability

不要新增：

ProjectMusicConfig

ProjectSfxConfig

==================================================
十九、Preview
==================================================

Generate：

只创建：

GenerationTask

不要创建正式 Asset。

GenerationTask.output 可以保存：

{
  assetType: "AUDIO",
  audioType: "MUSIC",
  durationSeconds,
  mimeType,
  previewUrl,
  provider,
  model,
  metadata
}

注意：

如果 Provider 返回的是临时 URL：

不能假设永久有效。

Preview URL 只能用于 Preview。

不要在 GenerationTask 中长期保存大 Base64。

==================================================
二十、Apply
==================================================

用户：

Preview
→ Apply

才正式：

1. 下载 / 接收音频
2. 写入 Asset Storage
3. 创建 Asset
4. 创建 EpisodeAudioAsset
5. 设置 FINAL
6. 设置 Primary
7. generationTask.appliedAt

整个过程必须：

prisma.$transaction

如果文件已经写入但 DB transaction 失败：

执行补偿删除。

不能留下孤儿文件。

==================================================
二十一、Asset Storage
==================================================

继续：

AssetStorageService

不要业务代码直接：

fs.writeFile

路径：

storage/assets/{projectId}/{assetId}/

例如：

music.mp3

sfx.wav

不要把音频 Base64 长期存 PostgreSQL。

支持：

audio/mpeg
audio/wav
audio/ogg
audio/aac
audio/mp4

如果 Provider 返回其他格式：

根据真实 Content-Type 处理。

不要硬编码 mp3。

==================================================
二十二、Music History
==================================================

Web：

/projects/:id/music

展示：

标题
Preview
Duration
Style
Mood
Provider
Model
Version
Status
CreatedAt

支持：

播放
查看详情
设为最终
删除 / 软删除（如果现有 Asset 体系支持）

重新生成：

创建新的 GenerationTask

创建新的 Asset

不能覆盖历史。

==================================================
二十三、SFX History
==================================================

Web：

/projects/:id/sfx

展示：

名称
Preview
Category
Duration
Provider
Model
Version
Status
CreatedAt

支持：

播放
设为最终
历史版本

同样：

重新生成 ≠ 覆盖

==================================================
二十四、Episode 工作台
==================================================

在：

/projects/:id/episodes/:episodeId

增加：

Music
SFX

入口。

例如：

🎵 Music
🔊 SFX

允许：

添加
生成
播放
设置 Final
查看历史

但是：

本阶段不要做：

Timeline

轨道

剪辑

拖动音频

Mix

Volume automation

Fade in/out

BGM ducking

FFmpeg

==================================================
二十五、与 Storyboard 的关系
==================================================

本阶段：

Music / SFX 不直接成为 StoryboardShotAsset。

原因：

Image / Video 属于 Shot Visual Asset。

Music / SFX 属于 Episode Audio Asset。

所以：

StoryboardShot
 ├─ IMAGE
 └─ VIDEO

Episode
 ├─ MUSIC
 └─ SFX

ScriptBlock
 └─ TTS

保持三个音频来源层次：

Episode Music
Episode SFX
ScriptBlock Dialogue TTS

不要混成一个模型。

==================================================
二十六、SFX 上下文增强
==================================================

虽然 SFX 主要挂 Episode：

但 API 允许：

sceneId?
shotId?

如果提供：

必须验证：

Scene 属于 Episode
Shot 属于 Episode
Shot 属于当前 Storyboard

这样可以生成：

「艾尔拔剑」

「灵械核心爆炸」

「飞船引擎启动」

等上下文音效。

但第一版仍然：

Asset 归属 Episode。

metadata 保存：

{
  "sceneId": "...",
  "shotId": "...",
  "source": "storyboard"
}

不要建立复杂 Timeline Relation。

==================================================
二十七、Music Context
==================================================

Music Prompt 不只是：

"Generate epic music"

应该自动结合：

Story Bible
Episode outline
Episode tone
Episode emotional state

例如：

Episode：

第一次接触

Story tone：

神秘、紧张、希望

Music AI context：

- mysterious first contact
- gradual tension
- sci-fi atmosphere
- cultivation + cyberpunk fusion

但不要替用户强行生成 Prompt。

应该：

用户 Prompt
+
Story Context

组成最终 Prompt。

==================================================
二十八、SFX Context
==================================================

SFX：

用户输入：

"飞船撞击空间站"

Context Builder 自动补充：

Episode
Scene
Shot
Action
Visual Description

最终 AI 输入更接近：

"Spaceship collision with a massive space station,
metal deformation,
energy burst,
low-frequency impact,
sci-fi industrial environment..."

具体内容由 AI 根据 Context 生成。

==================================================
二十九、GenerationTask usage
==================================================

继续使用：

usage

至少记录：

durationMs

如果 Provider 返回：

audioDurationSeconds

则记录：

audioDurationSeconds

如果 Provider 返回：

sizeBytes

记录：

sizeBytes

如果 Provider 返回：

providerRequestId

可以记录：

providerRequestId

但不要伪造：

tokenUsage

cost

estimatedCost

除非 Provider 真实返回。

==================================================
三十、费用
==================================================

绝对不要做：

fake cost

fake billing

fake token price

不要计算：

$0.03

$0.1

等伪造数字。

UI 只显示：

"费用由当前配置的 Provider 账户承担。"

如果未来 Provider API 返回真实 usage：

只保存 usage。

Billing 放到未来阶段。

==================================================
三十一、错误处理
==================================================

新增：

MUSIC_PROVIDER_NOT_CONFIGURED
SFX_PROVIDER_NOT_CONFIGURED
MUSIC_CAPABILITY_NOT_SUPPORTED
SFX_CAPABILITY_NOT_SUPPORTED

以及必要的：

MUSIC_GENERATION_FAILED
SFX_GENERATION_FAILED
AUDIO_ASSET_APPLY_FAILED

错误信息：

不能包含 API Key。

不能包含 encryptedApiKey。

不能包含 Authorization header。

==================================================
三十二、Generation History
==================================================

现有：

/projects/:id/generations

继续支持：

MUSIC
SFX

显示：

Type
Capability
Status
Provider
Model
Duration
CreatedAt

不要显示：

API Key

==================================================
三十三、API Client
==================================================

新增：

generateMusic()
generateSfx()

以及：

getMusicAssets()
getSfxAssets()

以及必要的：

getEpisodeAudioAssets()

setPrimaryMusicAsset()

setPrimarySfxAsset()

但优先复用现有 Asset Client。

不要 Web 页面自己 fetch。

==================================================
三十四、Shared Types
==================================================

增加：

MusicGenerationInput
SfxGenerationInput

MusicGenerationResult
SfxGenerationResult

EpisodeAudioAsset

MusicMetadata

SfxMetadata

AudioAssetRole

如果已经存在同类类型：

复用，不重复创建。

==================================================
三十五、Core
==================================================

packages/core 增加：

music.ts
sfx.ts

包含：

normalizeMusicInput()
normalizeSfxInput()

validateMusicDuration()
validateSfxDuration()

以及必要的纯函数。

Core：

不能依赖 Prisma。

不能依赖 NestJS。

==================================================
三十六、Duration 安全限制
==================================================

必须增加合理限制。

例如：

Music：

1–600 秒

SFX：

0.1–60 秒

具体限制如果项目已有 config 体系：

放入 config。

不要散落 magic number。

如果用户输入超过限制：

返回：

INVALID_DURATION

==================================================
三十七、Web UI
==================================================

新增：

Music Generation Modal

字段：

Prompt
Style
Mood
Genre
Duration
Instrumentation
Tempo
Instrumental

SFX Generation Modal

字段：

Prompt
Category
Intensity
Duration

UI 流程：

填写
↓
生成
↓
Loading
↓
Preview
↓
播放
↓
重新生成
↓
应用为最终

不要：

点击生成后直接正式入库。

==================================================
三十八、Loading / Error UX
==================================================

生成过程中：

"正在生成音乐..."

"正在生成音效..."

如果没有配置：

"尚未配置音乐生成 AI"

"尚未配置音效生成 AI"

按钮：

[前往 AI 配置]

跳转：

/projects/:id/settings#MUSIC

/projects/:id/settings#SFX

如果 Provider 不支持：

"当前 Provider 不支持音乐生成"

不要显示：

"生成失败，请重试"

这种没有意义的通用错误。

==================================================
三十九、Mobile / Desktop
==================================================

保持现有策略。

Mobile：

可以：

查看
播放
基础编辑

复杂 AI 生成：

引导 Web。

Desktop：

可以：

查看
播放
基础编辑

复杂生成：

引导 Web。

不要为了 Phase 12 重建 Desktop 音频工作站。

==================================================
四十、Seed
==================================================

新增：

prisma/seed-music-sfx-demo.ts

命令：

pnpm db:seed:music-sfx-demo

必须：

不调用真实 AI。

创建本地 fixture。

至少：

2 个 Music Asset

4 个 SFX Asset

例如：

Music：

- First Contact Theme
- Cyber Cultivation Theme

SFX：

- Spaceship Impact
- Energy Explosion
- Mechanical Core
- Sword Draw

全部：

Asset(type=AUDIO)

metadata：

{
  demo: true,
  source: "seed"
}

必须关联：

EpisodeAudioAsset

不要自动执行 seed。

==================================================
四十一、测试
==================================================

至少覆盖：

1. MUSIC capability
2. SFX capability
3. Resolver
4. ProjectAiConfig
5. Provider enabled
6. Provider disabled
7. Provider no API key
8. Capability unsupported
9. Cross project protection
10. Invalid episode
11. Invalid duration
12. Preview 不创建 Asset
13. Apply 创建 Asset
14. Apply 创建 EpisodeAudioAsset
15. Apply transaction
16. 文件失败补偿
17. GenerationTask.appliedAt
18. duplicate Apply
19. history 保留
20. setPrimary 不调用 AI
21. API Key 不泄漏
22. error sanitize
23. MUSIC 不 fallback TTS
24. SFX 不 fallback TTS
25. MUSIC 不 fallback CHAT
26. SFX 不 fallback CHAT
27. Context Builder 不泄漏敏感字段
28. SFX sceneId 校验
29. SFX shotId 校验
30. duration limit
31. API Client
32. Core helpers

目标：

所有既有测试继续通过。

不要为了通过测试删除已有测试。

==================================================
四十二、Migration
==================================================

migration 名称：

20260818100000_music_sfx_engine

如果时间戳已存在或命名冲突：

使用当前合理 timestamp。

原则：

增量 migration。

不要：

migrate reset

DROP TABLE

删除历史字段。

==================================================
四十三、数据库完整性
==================================================

执行：

pnpm exec prisma migrate deploy

pnpm exec prisma validate

如果需要：

pnpm exec prisma generate

如果 Windows 下出现：

EPERM
query_engine-windows.dll.node

不要删除数据库。

检查是否 API 进程占用。

如果无法安全重启：

记录原因。

不要杀掉用户正在运行的 API 进程。

==================================================
四十四、质量检查
==================================================

完成后必须执行：

pnpm prisma validate

pnpm lint

pnpm typecheck

pnpm test

如果项目已有：

pnpm build

也执行。

如果失败：

优先修复本阶段引入的问题。

不要为了通过：

修改测试预期

删除功能

绕过类型检查

使用 any 大面积压制错误

==================================================
四十五、API 健康检查
==================================================

确认：

http://localhost:3011/health

正常。

Web：

http://localhost:3010

正常。

如果端口被占用：

不要擅自杀进程。

报告：

哪个 PID 占用。

确认现有服务是否正常。

==================================================
四十六、架构文档
==================================================

新增：

docs/architecture/music-sfx-generation.md

必须解释：

Music / SFX Architecture

包括：

Capability
ProviderResolver
ProjectAiConfig
GenerationTask
Preview
Apply
Asset
EpisodeAudioAsset
Storage

并明确：

Music / SFX 当前没有 Timeline。

==================================================
四十七、不要实现的东西
==================================================

本阶段严禁实现：

Timeline

Audio Timeline

FFmpeg

Mixing

Mastering

Volume Automation

Fade

Crossfade

Ducking

BGM + Dialogue Mix

SFX Layer

Episode Render

Video Render

Subtitle

Caption

Export

Publish

YouTube

TikTok

Bilibili

Instagram

Redis

BullMQ

Worker

Agent

RAG

Embedding

Billing

Payment

Login

Register

JWT

Subscription

Voice Clone

Character Voice Clone

Music Remix

Music Extend

Lip Sync

Talking Avatar

不要顺手实现。

==================================================
四十八、重要产品设计原则
==================================================

牢记：

AI Drama Studio 最终不是：

"一个 AI Provider 生成所有东西"

而是：

Project
 ↓
AI Capability
 ↓
Provider
 ↓
Model
 ↓
Generation
 ↓
Asset

不同能力允许不同 Provider：

STRUCTURED_OUTPUT → DeepSeek
IMAGE → 图片模型
IMAGE_TO_VIDEO → 视频模型
TTS → 语音模型
MUSIC → 音乐模型
SFX → 音效模型

未来用户自己承担 AI 成本。

平台可以提供默认 Provider 作为：

Demo / Development

但最终：

用户应该配置自己的 Provider / API Key。

==================================================
四十九、最终验收标准
==================================================

Phase 12 完成必须满足：

[ ] MUSIC Capability
[ ] SFX Capability
[ ] ProjectAiConfig 支持 MUSIC
[ ] ProjectAiConfig 支持 SFX
[ ] Provider Resolver 支持 MUSIC
[ ] Provider Resolver 支持 SFX
[ ] Music Generation
[ ] SFX Generation
[ ] GenerationTask
[ ] Preview
[ ] Apply
[ ] Asset AUDIO
[ ] Episode Audio Relation
[ ] Music History
[ ] SFX History
[ ] Project Settings
[ ] Provider UI
[ ] Audio Storage
[ ] Primary Music
[ ] Primary SFX
[ ] Seed
[ ] Tests
[ ] lint
[ ] typecheck
[ ] prisma validate
[ ] build
[ ] API health
[ ] Web health
[ ] Architecture documentation

并且：

[ ] 不修改历史 migration
[ ] 不 reset DB
[ ] 不删除现有数据
[ ] 不泄漏 API Key
[ ] 不伪造 AI 成功
[ ] 不伪造费用
[ ] 不跨 Capability fallback
[ ] 不实现 Timeline
[ ] 不实现 FFmpeg
[ ] 不实现整集渲染

==================================================
五十、Phase 12 完成后的产品能力
==================================================

完成后，AI Drama Studio 应该拥有：

世界观
↓
人物
↓
Season
↓
Episode
↓
Script
↓
Storyboard
↓
┌───────────────┐
│               │
Image           Video
│               │
Asset           Asset
│
└───────┐
        │
        ↓
     Shot Visual


Script
↓
Dialogue
↓
TTS
↓
Audio Asset


Episode
├── Music
│    ↓
│   Audio Asset
│
└── SFX
     ↓
    Audio Asset

此时：

视觉资产
图片
视频

声音资产
Dialogue
Music
SFX

基本全部形成。

但是：

这些 Asset 仍然只是“生产资产”。

不要在 Phase 12 把它们拼成最终视频。

==================================================
五十一、最终报告
==================================================

完成后输出：

1. 修改文件
2. 新增 Prisma Model / Enum
3. Migration 名称
4. Migration 是否执行
5. Music API
6. SFX API
7. Provider Resolver
8. Capability
9. Adapter
10. Asset Relation
11. Storage
12. Preview / Apply
13. Web
14. Mobile
15. Desktop
16. Seed
17. Tests 数量
18. lint
19. typecheck
20. prisma validate
21. build
22. API health
23. Web health
24. 数据是否保留
25. Provider 是否保留
26. GenerationTask 是否保留
27. 遗留问题
28. 本阶段明确未实现内容
29. 下一阶段建议

最后不要自行开始 Phase 13。

Phase 12 到此停止。