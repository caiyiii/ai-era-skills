你现在开始执行 AI Drama Studio Phase 9：

# Phase 9：Image Generation Engine v1
## Storyboard → Image → Asset

这是一个生产级增量阶段。

必须严格遵守现有项目架构，不允许重建 Monorepo，不允许更换技术栈，不允许 reset 数据库，不允许修改历史 migration，不允许破坏 Phase 1～8 已有功能。

==================================================
一、当前项目架构
==================================================

项目：
AI Drama Studio

当前技术栈：

- pnpm
- Turborepo
- Nuxt 3 + Vue 3
- NestJS
- Ionic Vue + Capacitor
- Tauri 2 + Vue
- Prisma
- PostgreSQL
- Docker
- packages/types
- packages/api-client
- packages/core
- packages/config

当前端口：

Web：
3010

API：
3011

当前完整内容生产链：

Project
  ↓
Story Bible
  ↓
World
  ↓
Characters
  ↓
Season
  ↓
Episode
  ↓
Script
  ↓
Storyboard
  ↓
【Phase 9 Image】
  ↓
Asset
  ↓
未来 Phase 10+ Video
  ↓
未来 Editing
  ↓
未来 Publishing

目前已经完成：

Phase 4：
AI Generation Engine

Phase 4.5：
AI Provider Management

Phase 4.6：
AI Capability / Provider Architecture

Phase 5：
Character + AI Character Generation

Phase 6：
Story Bible + Season + Episode

Phase 7：
Script Engine

Phase 8：
Storyboard Engine

本阶段只实现：

StoryboardShot → Image Generation → Asset

==================================================
二、最重要的架构原则
==================================================

必须严格遵守以下原则。

1.
Image Generation 的唯一业务上游是：

StoryboardShot

不能重新解析：

Script
Episode
Story Bible

来决定图片内容。

正确：

StoryboardShot
    ↓
Image Generation

错误：

Script
    ↓
Image Generation

2.
StoryboardShot 是画面的唯一 Source of Truth。

3.
图片生成结果必须进入 Asset。

4.
AI 生成 Preview 阶段不能直接修改业务资产。

5.
用户确认 Apply 后才能正式创建/更新 Asset。

6.
所有 AI 调用必须经过：

ProjectAiConfig
→ Provider Resolver
→ AiCapability.IMAGE
→ AiModel
→ Provider Adapter

绝对禁止：

- 写死 DeepSeek
- 写死 OpenAI
- 写死某个图片模型
- 业务代码读取 AI_API_KEY
- 前端直接调用 AI Provider

7.
平台默认 Provider 只是开发 / Demo / 冷启动用途。

最终产品必须让用户：

自己配置 Provider
自己配置 Model
自己承担 AI 生成成本

平台不能设计成长期替用户承担图片生成成本。

8.
未来不同 AI 能力可以使用不同 Provider：

CHAT
STRUCTURED_OUTPUT
IMAGE
VIDEO
IMAGE_TO_VIDEO
TTS
VOICE_CLONE
MUSIC
EMBEDDING

例如：

文本：
DeepSeek

图片：
用户自己的 Flux / SDXL / GPT Image / Gemini Image / 其他 Provider

视频：
用户自己的 Kling / Runway / Veo / Seedance / 其他 Provider

语音：
用户自己的 ElevenLabs / MiniMax / OpenAI / 其他 Provider

本阶段只真正实现：

IMAGE

其他 Capability 继续保持未实现状态。

==================================================
三、本阶段目标
==================================================

实现完整的：

StoryboardShot
    ↓
Image Generation Task
    ↓
Provider Resolver
    ↓
IMAGE Provider
    ↓
生成图片
    ↓
Preview
    ↓
用户确认
    ↓
Asset
    ↓
StoryboardShot 关联 Asset
    ↓
前端显示图片

必须实现：

- 单 Shot 图片生成
- 图片重新生成
- 图片 Preview
- 图片 Apply
- 图片生成历史
- Asset 持久化
- Shot 与 Asset 关联
- Provider / Model / Capability 正确解析
- GenerationTask 记录
- 失败状态
- 错误脱敏
- 基础图片元数据
- 图片版本
- 未来 Video 使用 Asset 的架构预留

==================================================
四、不要实现的东西
==================================================

本阶段明确禁止继续实现：

- Video Generation
- Image-to-Video
- TTS
- Voice Clone
- Music
- Editing
- Timeline Editor
- FFmpeg
- 视频合成
- 字幕系统
- 自动发布
- YouTube API
- TikTok API
- Bilibili API
- Login
- Register
- JWT
- Subscription
- Billing
- Payment
- Redis
- BullMQ
- Worker
- Agent
- RAG
- Embedding
- Character Image Generation
- Character Voice Generation
- 自动批量整集生成
- 自动发布

但是数据库 / Asset 架构必须为这些未来能力预留合理扩展空间。

==================================================
五、Asset 架构
==================================================

检查当前 Prisma 中已有 Asset 模型。

不要删除已有 Asset。

如果现有 Asset 字段不足，可以通过增量 migration 扩展。

Asset 必须能够表达：

- 图片
- 视频
- 音频
- 未来其他媒体

不要为 Image 单独建立 ImageAsset 表。

统一使用：

Asset

作为未来所有媒体资产的统一抽象。

建议结构至少能够支持：

Asset：

id
projectId
type
status
name
mimeType
storageKey
url
thumbnailUrl
width
height
durationSeconds
sizeBytes
provider
model
metadata
createdAt
updatedAt

如果现有字段已经存在，则复用，不重复创建。

如果字段命名不同，优先兼容现有结构，而不是重建。

==================================================
六、AssetType
==================================================

检查现有 AssetType。

如果已经包含 IMAGE，则直接复用。

如果没有 IMAGE，则增量增加：

IMAGE

未来已经预留：

VIDEO
AUDIO

不要在本阶段强行实现这些能力。

==================================================
七、Asset 状态
==================================================

如果当前 Asset 没有合理状态，可以增加：

PENDING
READY
FAILED
DELETED

如果现有枚举已经能够表达这些状态，优先复用。

不要为了本阶段过度设计。

==================================================
八、StoryboardShot 与 Asset 关系
==================================================

这是本阶段非常重要的架构。

不要简单把：

assetId

作为唯一字段硬编码在 StoryboardShot 上。

因为未来一个 Shot 可能拥有：

- 多张生成图片
- 多个版本
- 多个候选图
- 最终选中的图片
- 视频
- 视频封面
- 动态参考图

因此建议建立：

StoryboardShotAsset

关系表。

关系：

StoryboardShot
      │
      ├── Asset 1
      ├── Asset 2
      ├── Asset 3
      └── Asset 4

StoryboardShotAsset：

id
shotId
assetId
role
isPrimary
sortOrder
metadata
createdAt

role 至少支持：

REFERENCE
GENERATED
FINAL
THUMBNAIL

如果认为当前阶段不需要全部 role，可以最少实现：

GENERATED
FINAL

但必须为未来扩展保留 role。

唯一约束建议：

shotId + assetId

避免重复关联。

如果当前 Asset 模型已经存在合理关系结构，可以复用，而不是重复建表。

==================================================
九、图片生成任务
==================================================

复用：

GenerationTask

不要新建 ImageGenerationTask。

GenerationTaskType 增加：

IMAGE

如果 IMAGE 已存在，则复用。

GenerationTask 必须记录：

type = IMAGE

capability = IMAGE

status

input

output

provider

model

usage

error

createdAt

updatedAt

==================================================
十、Image Generation Input
==================================================

创建：

CreateImageGenerationDto

至少支持：

shotId

promptOverride?

negativePromptOverride?

aspectRatio?

width?

height?

count?

seed?

style?

referenceAssetIds?

用户可以选择：

是否使用 StoryboardShot 自带 imagePrompt

是否覆盖 Prompt

是否指定图片尺寸

是否生成多张候选图

默认：

count = 1

建议允许：

1～4

禁止无限生成。

==================================================
十一、Prompt 来源
==================================================

默认图片 Prompt 必须来自：

StoryboardShot.imagePrompt

而不是重新调用 LLM 生成 Prompt。

因为：

StoryboardShot 已经是上游视觉规划结果。

默认：

shot.imagePrompt

negative：

shot.negativePrompt

同时可以组合：

- shot.visualDescription
- shot.composition
- shot.lighting
- shot.mood
- shot.style
- shot.characters
- shot.cameraAngle
- shot.shotSize
- shot.cameraMovement

但是不要把整个数据库对象 JSON.stringify 给 Provider。

只构造结构化 ImageGenerationInput。

==================================================
十二、Reference Image
==================================================

需要为未来角色一致性保留：

referenceAssetIds

例如：

Shot：

沈星河站在废墟中

可以指定：

Character 沈星河的参考图片 Asset

未来：

Character
→ imageProfile
→ Character Reference Asset
→ StoryboardShot
→ Image Generation

但是本阶段不要实现完整 Character Image Generation。

只允许：

referenceAssetIds

作为可选输入。

必须验证：

Asset 属于当前 Project。

禁止跨项目 Asset。

==================================================
十三、Provider Architecture
==================================================

必须使用：

resolveForCapability(
    projectId,
    AiCapability.IMAGE
)

绝对不能：

resolveForCapability(
    projectId,
    STRUCTURED_OUTPUT
)

图片必须是：

IMAGE

完整流程：

ImageGenerationService
        ↓
ProviderResolver
        ↓
AiCapability.IMAGE
        ↓
AiModel
        ↓
AiProvider
        ↓
Image Provider Adapter
        ↓
generateImage()

==================================================
十四、AI Provider 接口扩展
==================================================

检查现有：

AiProvider

如果当前只有：

generateText()
generateStructured()

则增量增加：

generateImage()

建议接口：

generateImage(input: ImageGenerationInput): Promise<ImageGenerationResult>

不要破坏现有：

generateText
generateStructured

==================================================
十五、ImageGenerationResult
==================================================

不要强制 Provider 必须返回公网 URL。

统一抽象：

ImageGenerationResult：

images: [
  {
    url?
    base64?
    mimeType
    width?
    height?
    seed?
    revisedPrompt?
    providerAssetId?
    metadata?
  }
]

原因：

不同 Provider 返回方式不同：

- URL
- Base64
- Object Storage
- Provider Asset ID

Adapter 层负责转换。

业务层不关心 Provider API 的具体响应格式。

==================================================
十六、OpenAI Compatible Image Provider
==================================================

当前系统已有：

OpenAiCompatibleProvider

检查当前 Provider 是否支持图片。

如果支持：

实现：

generateImage()

但必须考虑：

OpenAI-compatible 不代表所有 Provider 的 Image API 都完全一致。

因此：

不要假设所有 OpenAI Compatible Provider 都支持：

POST /images/generations

必须：

1.
先通过 capability 配置判断 Provider 支持 IMAGE。

2.
Adapter 尝试调用标准 Image API。

3.
对不支持的 Provider 返回：

CAPABILITY_NOT_SUPPORTED

而不是：

500 MODULE ERROR

==================================================
十七、Provider Capability
==================================================

现有：

AiProviderCapability

必须支持：

IMAGE

Provider 管理 UI 可以显示：

Chat
Structured Output
Image
Video
Image to Video
TTS
...

但本阶段：

真正可执行：

IMAGE

如果 Provider 没有 IMAGE：

禁止生成。

错误：

AI_CAPABILITY_NOT_SUPPORTED

不要 fallback 到：

DeepSeek Chat

特别注意：

IMAGE 不允许 fallback 到：

STRUCTURED_OUTPUT
CHAT

这条规则非常重要。

==================================================
十八、Project AI Config
==================================================

图片生成必须读取：

ProjectAiConfig

capability：

IMAGE

例如：

ProjectAiConfig：

projectId = demo-xinghe
capability = IMAGE
providerId = xxx
modelId = xxx

如果没有：

IMAGE

配置：

按照现有 Resolver 规则 fallback。

但是：

只有支持 IMAGE 的 Provider 才允许。

Resolver：

ProjectAiConfig IMAGE
→ User Provider IMAGE
→ Platform Default IMAGE
→ System IMAGE Provider（如果未来支持）
→ NO_AI_PROVIDER_CONFIGURED

不能：

ProjectAiConfig IMAGE
→ DeepSeek TEXT

==================================================
十九、用户承担 AI 成本
==================================================

必须在架构上明确：

Platform Provider：

仅作为：

开发
Demo
冷启动

不是最终商业模式。

未来正式产品：

User
  ↓
自己的 Provider
  ↓
自己的 API Key
  ↓
自己的 AI 成本

本阶段 UI 中可以增加说明：

「图片生成将使用当前项目配置的 AI Provider，相关 API 使用费用由对应 Provider 账户承担。」

不要实现 Billing。

==================================================
二十、API
==================================================

新增：

POST

/projects/:projectId/episodes/:episodeId/storyboard/shots/:shotId/generate-image

用于创建图片生成任务。

推荐也支持：

POST

/projects/:projectId/generations/image

Body：

{
  shotId,
  promptOverride?,
  negativePromptOverride?,
  width?,
  height?,
  aspectRatio?,
  count?,
  seed?,
  referenceAssetIds?
}

如果架构上更统一，优先使用：

/generations/image

因为未来：

/generations/video
/generations/tts

可以保持一致。

推荐：

POST /projects/:projectId/generations/image

==================================================
二十一、Generation API
==================================================

复用：

GET
/projects/:projectId/generations

GET
/projects/:projectId/generations/:id

POST
/projects/:projectId/generations/:id/apply

Apply 必须根据：

GenerationTask.type

分支：

IMAGE

不能破坏：

WORLD
CHARACTER
STORY_BIBLE
SEASON_OUTLINE
EPISODE_OUTLINE
SCRIPT
STORYBOARD

==================================================
二十二、Image Generation Task 生命周期
==================================================

流程：

create
  ↓
PENDING
  ↓
RUNNING
  ↓
Provider
  ↓
Image Result
  ↓
Asset Preview
  ↓
SUCCEEDED

失败：

FAILED

不能把：

Provider Error

写入 Asset。

失败只写：

GenerationTask.error

==================================================
二十三、Preview
==================================================

这是本阶段最重要的 UX。

用户：

Storyboard
→ Shot
→ AI生成图片

AI 返回：

候选图片

Preview 页面显示：

- 图片
- Provider
- Model
- Prompt
- Negative Prompt
- 尺寸
- 生成耗时
- Seed（如果有）
- Reference Image
- 生成数量

此时：

不能创建最终 Asset。

可以将 Provider 返回的临时 URL / Base64 保存到：

GenerationTask.output

但必须明确：

output = Preview Result

不是正式 Asset。

==================================================
二十四、Apply
==================================================

用户点击：

「应用」

才执行：

prisma.$transaction

流程：

1.
创建 Asset

2.
写入：

type = IMAGE

3.
写：

provider
model
mimeType
width
height
metadata

4.
创建：

StoryboardShotAsset

5.
设置：

isPrimary = true

如果已有 FINAL：

旧 Asset 关系不要删除。

而是：

旧 FINAL → isPrimary = false

新 Asset → isPrimary = true

这样可以保留历史版本。

==================================================
二十五、重新生成
==================================================

用户点击：

「重新生成」

不能覆盖旧 Asset。

必须：

创建新的 GenerationTask。

生成新的 Preview。

Apply 后：

创建新的 Asset。

旧 Asset 保留。

这样形成：

Shot
 ├── Image v1
 ├── Image v2
 ├── Image v3
 └── Image v4 FINAL

这对于未来：

选择最佳图片

非常重要。

==================================================
二十六、Asset Version
==================================================

如果 Asset 当前没有 version：

增加：

version

或者通过：

metadata.version

实现。

推荐：

version

并且：

Shot + role + version

保持可追踪。

不要删除历史图片。

==================================================
二十七、Asset Storage
==================================================

这是本阶段必须认真设计的地方。

不要把：

Base64

长期保存到 PostgreSQL。

不要把：

巨大图片二进制

直接存数据库。

短期开发环境可以：

Provider URL

作为 preview。

Apply 时应该抽象：

AssetStorageService

例如：

saveFromUrl()
saveFromBase64()

统一返回：

storageKey
url
mimeType
size
width
height

本阶段可以先实现：

LocalStorageAdapter

开发环境：

./storage

或者：

storage/assets/{projectId}/{assetId}/...

但必须让未来可以替换：

S3
Cloudflare R2
MinIO
OSS
COS
其他 Object Storage

不要让业务代码直接操作 fs。

==================================================
二十八、Storage Adapter
==================================================

新增：

AssetStorageService

接口：

saveFromUrl()
saveFromBase64()
delete()
getUrl()

实现：

LocalAssetStorageProvider

未来预留：

S3AssetStorageProvider
R2AssetStorageProvider
OSSAssetStorageProvider

本阶段不要实现云存储。

==================================================
二十九、安全
==================================================

绝对禁止：

- API Key 返回前端
- encryptedApiKey 返回前端
- API Key 写日志
- Provider response 中包含 Key
- GenerationTask.input 保存 API Key
- GenerationTask.output 保存 API Key

所有错误：

sanitizeSecret()

==================================================
三十、权限边界
==================================================

虽然当前没有登录系统，但所有 API 必须继续检查：

Project
→ Episode
→ Storyboard
→ Shot

归属关系。

不能：

Project A
引用
Project B Shot

不能：

Project A
引用
Project B Asset

不能：

Project A
使用
Project B Character Reference Asset

==================================================
三十一、图片尺寸
==================================================

不要把尺寸写死。

支持：

aspectRatio：

1:1
4:3
3:4
16:9
9:16
21:9

但必须允许 Provider Adapter 自己决定最终尺寸。

数据库记录：

requestedWidth
requestedHeight
actualWidth
actualHeight

如果现有结构不需要全部列，可以：

metadata

保存。

==================================================
三十二、生成数量
==================================================

默认：

count = 1

最大：

4

服务端强制限制。

不能相信前端。

例如：

count = 100

必须拒绝。

==================================================
三十三、Seed
==================================================

新增：

prisma/seed-image-demo.ts

命令：

pnpm db:seed:image-demo

不要自动执行。

如果当前存在：

星河碰撞
E01
Storyboard

则可以：

创建模拟 Asset

但是：

不要调用真实 AI Provider。

Demo Asset 可以引用本地 placeholder。

同时创建：

StoryboardShotAsset

用于测试 UI。

==================================================
三十四、Web UI
==================================================

新增：

/projects/:id/episodes/:episodeId/images

或者：

/projects/:id/images

推荐：

/projects/:id/episodes/:episodeId/storyboard

直接在 Storyboard Shot Inspector 中集成图片。

核心体验：

Storyboard
  ↓
选择 Shot
  ↓
右侧 Inspector
  ↓
「生成图片」

生成后：

Preview

显示：

┌──────────────────────────────┐
│                              │
│          图片                │
│                              │
└──────────────────────────────┘

Provider：
DeepSeek / xxx

Model：
xxx

Size：
16:9

Prompt：
......

[重新生成]

[应用为最终图片]

==================================================
三十五、Shot 图片状态
==================================================

Shot Card 可以显示：

无图片：

「未生成」

生成中：

「生成中」

已有候选：

「有候选」

已有最终图片：

「已完成」

如果 Script / Storyboard 已变：

可以显示：

「分镜已更新」

但不要自动删除图片。

==================================================
三十六、Image Gallery
==================================================

Shot Inspector 中增加：

「图片历史」

显示：

v1
v2
v3
v4

每张：

- 图片
- Provider
- Model
- 时间
- Seed
- 状态
- 是否 Final

操作：

「设为最终」

不要删除历史。

==================================================
三十七、Generation History
==================================================

继续复用：

GenerationTask

图片生成历史显示：

时间
类型
状态
Provider
Model
耗时
Shot
结果数量

不要显示：

API Key

==================================================
三十八、Apply 事务
==================================================

Apply Image Generation：

必须使用：

prisma.$transaction

确保：

Asset 创建成功
+
StoryboardShotAsset 创建成功
+
GenerationTask.appliedAt

全部成功。

任何一步失败：

全部 rollback。

==================================================
三十九、幂等性
==================================================

Apply 不能因为用户重复点击而创建两个完全一样的最终 Asset。

必须考虑：

GenerationTask.appliedAt

如果已经 Apply：

返回：

GENERATION_ALREADY_APPLIED

不要重复执行。

==================================================
四十、GenerationTask output
==================================================

建议结构：

{
  "images": [
    {
      "url": "...",
      "mimeType": "image/png",
      "width": 1024,
      "height": 1024,
      "seed": 123,
      "metadata": {}
    }
  ],
  "provider": "...",
  "model": "...",
  "requestedCount": 1,
  "durationMs": 1234
}

注意：

正式 Asset 创建后：

output 只是生成记录。

Asset 才是正式业务资产。

==================================================
四十一、Usage
==================================================

继续使用：

GenerationTask.usage

至少记录：

{
  durationMs,
  imageCount
}

如果 Provider 返回：

token
cost
credits

可以保存到 metadata / usage。

但：

不要伪造成本。

不要实现 Billing。

不要告诉用户：

「本次花费 $0.03」

除非 Provider 明确返回真实成本。

==================================================
四十二、错误类型
==================================================

增加必要错误：

IMAGE_GENERATION_FAILED

IMAGE_PROVIDER_NOT_CONFIGURED

IMAGE_CAPABILITY_NOT_SUPPORTED

IMAGE_MODEL_NOT_SUPPORTED

IMAGE_ASSET_SAVE_FAILED

IMAGE_REFERENCE_ASSET_NOT_FOUND

IMAGE_REFERENCE_ASSET_PROJECT_MISMATCH

GENERATION_ALREADY_APPLIED

INVALID_IMAGE_COUNT

INVALID_IMAGE_SIZE

SHOT_NOT_FOUND

SCRIPTBOARD_SHOT_NOT_FOUND

注意错误必须：

稳定
可读
不泄漏 Secret

==================================================
四十三、Provider Adapter
==================================================

当前至少实现：

OpenAiCompatibleImageProvider

但是必须做 capability 判断。

如果 Provider 不支持 IMAGE：

返回：

CAPABILITY_NOT_SUPPORTED

不要硬调用。

如果当前 DeepSeek Provider 没有 Image capability：

不能让用户点击生成后再调用 DeepSeek。

前端应该提前显示：

「当前 Provider 不支持图片生成，请在项目 AI 配置中选择支持 IMAGE 的 Provider / Model。」

==================================================
四十四、DeepSeek
==================================================

当前系统可能正在使用：

DeepSeek

作为：

CHAT
STRUCTURED_OUTPUT

这是正常的。

但是：

不要默认把 DeepSeek 用作：

IMAGE

除非它真实支持并配置了 IMAGE capability。

必须允许：

文本 Provider：

DeepSeek

图片 Provider：

其他 Image Provider

这是本项目非常重要的架构要求。

==================================================
四十五、AI Capability UI
==================================================

检查：

/ai-providers

增加清晰展示：

Provider

支持能力：

✓ Chat
✓ Structured Output
✓ Image
○ Video
○ TTS
...

Project Settings：

/projects/:id/settings

AI Configuration：

Chat
Structured Output
Image
Video
Image to Video
TTS
Voice Clone
Music
Embedding

本阶段：

IMAGE 可以真正配置。

其他仍显示：

「Coming Soon」

不要出现假功能。

==================================================
四十六、项目级 Image 配置
==================================================

用户应该可以：

Project
→ Settings
→ AI Configuration
→ Image

选择：

Provider
Model

例如：

Chat
DeepSeek

Structured Output
DeepSeek

Image
Flux

未来：

Video
Kling

TTS
ElevenLabs

这是整个产品最终 BYOK 架构的基础。

==================================================
四十七、Model
==================================================

一个 Provider 可以有：

多个 Model。

例如：

Provider：
My OpenAI Compatible

Models：

Model A
Model B
Model C

每个 Model：

capabilities

可以不同。

例如：

Model A：
CHAT
STRUCTURED_OUTPUT

Model B：
IMAGE

Resolver 必须同时验证：

Provider enabled
+
Provider has API Key
+
Provider supports IMAGE
+
Model enabled
+
Model supports IMAGE

==================================================
四十八、不要破坏 Legacy Provider
==================================================

保留：

Project.aiProviderId

旧逻辑：

CHAT / STRUCTURED_OUTPUT

继续工作。

但：

IMAGE

必须优先走：

ProjectAiConfig

不能因为 Legacy Provider 是 DeepSeek 就自动用于 Image。

==================================================
四十九、Context
==================================================

Image Generation 不需要重新构建完整 StoryContext。

直接使用：

StoryboardShot

必要时：

Character Reference Asset

不要把：

World
Story Bible
Script

整个重新塞给 Image Provider。

Storyboard 已经完成视觉规划。

==================================================
五十、未来 Video 架构预留
==================================================

本阶段不要实现 Video。

但是设计必须支持未来：

StoryboardShot
   ↓
Final Image Asset
   ↓
Video Generation
   ↓
Video Asset

以及：

StoryboardShot
   ↓
Image Asset
   ↓
Image-to-Video
   ↓
Video Asset

因此：

Asset 必须是通用媒体抽象。

不要设计成：

ImageAsset

专属模型。

==================================================
五十一、未来剪辑预留
==================================================

未来：

Video Asset
+
Voice Asset
+
Music Asset
+
Subtitle
+
StoryboardShot.duration
+
Episode

→ Timeline
→ Render
→ Final Video

本阶段不要实现。

但：

Asset.metadata

可以预留：

source
generation
shotId
provider
model
seed
duration
technicalMetadata

不要过度设计。

==================================================
五十二、不要自动修改 Storyboard
==================================================

Image Generation：

不能修改：

StoryboardShot.imagePrompt
StoryboardShot.videoPrompt
StoryboardShot.visualDescription

除非用户明确执行：

「更新分镜 Prompt」

本阶段不实现该功能。

图片生成只是消费 Storyboard。

==================================================
五十三、图片生成失败
==================================================

如果：

Provider 失败
JSON 不适用
网络错误
图片下载失败
Storage 保存失败

必须：

GenerationTask = FAILED

并记录：

sanitized error

不能创建：

READY Asset

==================================================
五十四、网络与超时
==================================================

图片生成通常比文本慢。

必须设置合理：

timeout

不要无限等待。

建议：

120000ms

但 Provider Adapter 可以支持配置。

如果超时：

IMAGE_GENERATION_TIMEOUT

GenerationTask：

FAILED

==================================================
五十五、同步 / 异步
==================================================

本阶段：

可以继续同步执行。

即：

POST generation

等待 Provider 返回。

但是：

必须保持：

GenerationExecutor

架构。

不要把 Provider 调用直接塞进 Controller。

未来 Phase 可以替换：

GenerationExecutor
→ Queue
→ Worker

而 API 形状尽量不变。

==================================================
五十六、测试
==================================================

必须新增测试。

至少覆盖：

1.
IMAGE capability resolver

2.
Project Image Provider config

3.
Provider 不支持 IMAGE

4.
Model 不支持 IMAGE

5.
Provider disabled

6.
Model disabled

7.
Provider missing API Key

8.
Project mismatch

9.
Shot mismatch

10.
Reference Asset cross-project

11.
Image Generation Preview 不创建 Asset

12.
Image Generation FAILED 不创建 Asset

13.
Apply 创建 Asset

14.
Apply 创建 StoryboardShotAsset

15.
Apply 设置 Final

16.
旧 Final 保留

17.
重新生成产生新 GenerationTask

18.
GenerationTask appliedAt

19.
重复 Apply 被拒绝

20.
事务 rollback

21.
API Key 不泄漏

22.
Provider 不支持 Image 时不调用 AI

23.
count 最大值限制

24.
count 最小值限制

25.
Image Generation usage

26.
ProjectAiConfig resolver

27.
Legacy Provider 不误用于 IMAGE

==================================================
五十七、数据库 Migration
==================================================

必须：

新增一个 migration：

例如：

20260817xxxxxx_image_generation_engine

不要修改历史 migration。

不要：

prisma migrate reset

不要：

DROP TABLE

不要删除现有数据。

执行：

pnpm prisma migrate deploy

==================================================
五十八、Seed
==================================================

新增：

pnpm db:seed:image-demo

不要默认执行。

不要调用真实 AI API。

==================================================
五十九、API Client
==================================================

所有 Web 请求：

必须通过：

packages/api-client

禁止页面：

直接 fetch API。

新增：

createImageGeneration()
getImageGeneration()
applyImageGeneration()

以及：

getShotAssets()
getAsset()
setPrimaryShotAsset()

如果已有通用接口：

优先复用。

==================================================
六十、Shared Types
==================================================

新增：

ImageGenerationInput

ImageGenerationResult

ImageGenerationImage

Asset

StoryboardShotAsset

ImageGenerationMetadata

必要的 Input / Update 类型。

禁止：

Web / API / Mobile 各自重新定义。

统一：

packages/types

==================================================
六十一、Core
==================================================

检查：

packages/core

如果有适合的：

generation state
asset state
capability

应该放 Core。

不要把业务规则全部写在 Vue 页面。

==================================================
六十二、API Module
==================================================

建议：

apps/api/src/modules/assets/

apps/api/src/modules/image-generation/

或者根据现有架构：

generation/
assets/

保持项目现有模块风格。

不要为了本阶段重构整个 API。

==================================================
六十三、代码质量
==================================================

完成后必须运行：

pnpm prisma validate

pnpm lint

pnpm typecheck

pnpm test

如果项目有：

pnpm build

也运行。

必须全部通过。

==================================================
六十四、运行验证
==================================================

完成后检查：

API：

http://localhost:3011/health

Web：

http://localhost:3010

并确认：

World 正常

Character 正常

Story Bible 正常

Season 正常

Episode 正常

Script 正常

Storyboard 正常

Provider 正常

Project AI Config 正常

Image Generation API 正常

==================================================
六十五、现有数据绝对不能丢
==================================================

必须确认：

星河碰撞

World：
灵械纪元

Characters：

沈星河
太虚真人
艾尔

Season：

Season 1

Episodes：

E01
E02
E03

Script：

E01 Script

Storyboard：

E01 Storyboard（如果已经存在）

Provider：

现有 Provider

GenerationTask：

历史任务

全部保留。

==================================================
六十六、浏览器 UX
==================================================

不要只完成 API。

必须完成 Web 实际 UI。

用户操作路径：

项目
→ 星河碰撞
→ 剧集
→ E01
→ 分镜
→ 选择 Shot
→ 生成图片
→ Preview
→ 应用
→ Shot 显示最终图片

然后：

点击图片

→ 图片历史

→ 查看 v1 / v2 / v3

→ 选择某一版本

→ 设为最终图片

==================================================
六十七、没有 IMAGE Provider 时
==================================================

页面不能：

点击以后卡死。

应该：

按钮可见但显示：

「尚未配置图片模型」

提供：

「去 AI 配置」

跳转：

/projects/:id/settings

并自动定位：

IMAGE

==================================================
六十八、生成过程中
==================================================

按钮：

「生成图片」

变：

「生成中...」

禁止重复点击。

显示：

Provider
Model

并提供：

取消

如果当前同步执行无法真正取消：

不要伪造取消功能。

可以不提供 Cancel。

==================================================
六十九、不要伪造 AI
==================================================

非常重要：

不能使用：

随机图片
假图片
随机 URL
Mock AI

冒充真实生成成功。

Demo Seed 可以使用 placeholder。

真实：

Generate

必须调用真实 IMAGE Provider。

如果没有 Provider：

明确返回：

IMAGE_PROVIDER_NOT_CONFIGURED

==================================================
七十、文档
==================================================

增加：

docs/architecture/image-generation.md

说明：

1.
Storyboard → Image

2.
Provider Resolver

3.
Capability

4.
Asset

5.
Storage

6.
GenerationTask

7.
Preview / Apply

8.
未来 Video

9.
未来用户 BYOK

10.
为什么图片不能直接从 Script 生成

==================================================
七十一、完成后的最终架构
==================================================

最终应该达到：

                    Project
                       │
          ┌────────────┴────────────┐
          │                         │
      Story Bible                 AI Config
          │                         │
        World                  Capability
          │                         │
     Characters              Provider + Model
          │                         │
        Season                     │
          │                         │
       Episode                    │
          │                         │
        Script                     │
          │                         │
      Storyboard                   │
          │                         │
     StoryboardShot                │
          │                         │
          └──────────────┬──────────┘
                         │
                  IMAGE GENERATION
                         │
                  Provider Resolver
                         │
                   Image Adapter
                         │
                      Preview
                         │
                       Apply
                         │
                       Asset
                         │
             StoryboardShotAsset
                         │
                  Final Image
                         │
             ┌───────────┴───────────┐
             │                       │
        Future Video            Future Editing


==================================================
七十二、未来完整产品的成本模型
==================================================

不要实现 Billing。

但架构必须保持：

用户
 ↓
自己的 AI Provider
 ↓
自己的 API Key
 ↓
自己的费用

例如：

Project：

「星河碰撞」

AI Config：

CHAT
→ DeepSeek

STRUCTURED_OUTPUT
→ DeepSeek

IMAGE
→ 用户自己的 Image Provider

VIDEO
→ 用户自己的 Video Provider

TTS
→ 用户自己的 TTS Provider

MUSIC
→ 用户自己的 Music Provider

这样未来：

用户可以完全 BYOK。

平台只提供：

工作流
Prompt
Context
Schema
Provider Adapter
任务管理
资产管理
项目管理
最终生产工具

==================================================
七十三、自动处理要求
==================================================

你现在处于自动执行模式。

不要停下来询问我：

- 是否创建 migration
- 是否新增 Model
- 是否创建测试
- 是否修改 UI
- 是否增加 Asset Storage
- 是否增加 API
- 是否修复 lint
- 是否修复 typecheck
- 是否修复测试

请根据以上架构要求自行判断并完成。

如果发现现有代码与本 Prompt 有轻微差异：

优先：

1. 保留现有数据
2. 保留现有 API
3. 保留现有架构
4. 增量修改
5. 向后兼容

不要为了追求 Prompt 中的命名而重构整个项目。

如果某项能力当前基础设施不足：

实现最小可用版本，并为未来扩展保留接口。

==================================================
七十四、完成标准
==================================================

Phase 9 只有同时满足以下条件才算完成：

[ ] StoryboardShot 可以发起 Image Generation

[ ] 使用 AiCapability.IMAGE

[ ] 使用 ProjectAiConfig

[ ] Provider Resolver 正确工作

[ ] Model Resolver 正确工作

[ ] 不写死 Provider

[ ] 不写死 Model

[ ] 不读取前端 API Key

[ ] Provider Key 不泄漏

[ ] Image Generation Preview

[ ] Preview 不创建正式 Asset

[ ] Apply 创建 Asset

[ ] Shot 与 Asset 建立关系

[ ] 支持多次生成

[ ] 保留历史 Asset

[ ] 支持设置 Final Image

[ ] 支持 Reference Asset

[ ] 支持 Image Prompt Override

[ ] 支持 Negative Prompt

[ ] 支持尺寸 / 比例

[ ] count 最大限制

[ ] GenerationTask 记录完整

[ ] GenerationTask usage

[ ] 失败状态正确

[ ] Apply Transaction

[ ] 重复 Apply 防护

[ ] Project 隔离

[ ] Asset Storage 抽象

[ ] Local Storage 可运行

[ ] Web UI 完成

[ ] Generation History 完成

[ ] Image Gallery / Version 完成

[ ] AI Config Image 能力可配置

[ ] Provider 不支持 IMAGE 时正确提示

[ ] Demo Seed 可运行

[ ] prisma validate 通过

[ ] lint 通过

[ ] typecheck 通过

[ ] test 全部通过

[ ] 现有 World 数据保留

[ ] 现有 Character 数据保留

[ ] 现有 Script 数据保留

[ ] 现有 Storyboard 数据保留

[ ] 现有 Provider 数据保留

[ ] 现有 GenerationTask 数据保留

==================================================
七十五、阶段边界
==================================================

完成以上内容后：

立即停止。

不要继续实现：

Phase 10 Video

不要实现：

TTS
Music
Editing
Publishing

下一阶段再处理。

最后输出一份完整 Phase 9 完成报告，包括：

1. 修改文件
2. 新增 Model
3. 新增 Enum
4. Migration
5. API
6. Provider Adapter
7. Asset Storage
8. Image Generation 流程
9. Preview / Apply
10. Web UI
11. Seed
12. 测试数量
13. lint
14. typecheck
15. prisma validate
16. 当前数据库数据是否保留
17. 当前 Provider 是否保留
18. 当前 GenerationTask 是否保留
19. 当前 API 是否正常
20. 当前 Web 是否正常
21. 遗留问题
22. 明确没有实现什么
23. 下一阶段建议

不要开始 Phase 10。