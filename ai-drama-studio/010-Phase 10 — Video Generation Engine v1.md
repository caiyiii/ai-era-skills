# AI Drama Studio — Phase 10
# Video Generation Engine v1
# 完整 Cursor 自动执行 Prompt

你现在继续开发 AI Drama Studio。

当前项目已经完成：

- Phase 4：AI Generation Engine v1
- Phase 4.5：AI Provider Management
- Phase 4.6：AI Capability / Provider Architecture
- Phase 5：Character System + AI Character Generation
- Phase 6：Season / Episode + Story Bible
- Phase 7：Script Engine
- Phase 8：Storyboard Engine
- Phase 9：Image Generation Engine v1

当前核心架构：

Project
├── StoryBible
├── World
├── Characters
├── Seasons
│   └── Episodes
│       ├── Script
│       │   └── Scenes
│       │       └── ScriptBlocks
│       └── Storyboard
│           └── StoryboardShots
│               └── StoryboardShotAssets
│                   └── Asset
└── GenerationTasks

AI 架构：

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
AiCapability
        ↓
ProjectAiConfig
        ↓
AiProvider
        ↓
AiModel
        ↓
Provider Adapter

当前端口：

Web = 3010
API = 3011

当前必须保持：

- pnpm
- Turborepo
- Nuxt 3
- Vue 3
- NestJS
- Prisma
- PostgreSQL
- Ionic Vue
- Tauri
- packages/types
- packages/core
- packages/config
- packages/api-client

绝对禁止：

- 重建项目
- 更换技术栈
- 删除现有模块
- 删除历史 migration
- prisma migrate reset
- DROP TABLE
- 清空现有业务数据
- 重建数据库
- 修改现有 API Provider Resolver 的总体架构
- 在业务代码中直接读取 AI_API_KEY
- 在前端保存 API Key
- 把 API Key 返回给前端
- 写死 DeepSeek / OpenAI / Gemini 等具体模型
- 用文本 Chat Provider 冒充 Video Provider
- 用 IMAGE Provider 冒充 VIDEO Provider
- 从 Script 直接生成 Video
- 绕过 StoryboardShot
- 绕过 Preview → Apply
- 在本阶段实现完整视频剪辑系统
- 在本阶段实现 TTS
- 在本阶段实现字幕
- 在本阶段实现音乐
- 在本阶段实现 FFmpeg Timeline Editor
- 在本阶段实现自动发布
- 在本阶段实现 Billing / Payment
- 在本阶段实现 Login / Register / JWT
- 在本阶段实现 Redis / BullMQ / Worker
- 在本阶段实现复杂 Agent / RAG / Embedding

==================================================
一、Phase 10 的目标
==================================================

实现：

# Video Generation Engine v1

核心链路：

StoryboardShot
    ↓
Final Image Asset
    ↓
ProjectAiConfig
    ↓
ProviderResolver
    ↓
VIDEO / IMAGE_TO_VIDEO Capability
    ↓
Video Provider Adapter
    ↓
GenerationTask
    ↓
Preview
    ↓
用户确认
    ↓
Apply
    ↓
AssetType.VIDEO
    ↓
StoryboardShotAsset

也就是说：

StoryboardShot 是视频生成的唯一业务上游。

禁止：

Script → Video
Character → Video
Episode → Video

必须：

StoryboardShot → Video

Phase 10 v1 的最小目标：

用户进入：

项目
→ 剧集
→ 分镜
→ 某个 Shot
→ 视频生成

系统读取：

- Shot.imagePrompt
- Shot.videoPrompt
- Shot.negativePrompt
- Shot.visualDescription
- Shot.durationSeconds
- Shot.cameraMovement
- Shot.cameraAngle
- Shot.cameraMovementParams
- Shot.lighting
- Shot.mood
- Shot.style
- Shot.characters
- Shot.dialogue
- Shot.action
- Shot.continuityNotes

以及：

Shot 的 FINAL Image Asset

然后调用用户配置的视频 Provider。

生成视频 Preview。

用户确认 Apply。

最终形成：

Asset(type=VIDEO)
+
StoryboardShotAsset(role=GENERATED / FINAL)

==================================================
二、最重要的架构原则
==================================================

Phase 10 必须区分：

VIDEO
IMAGE_TO_VIDEO

两个 Capability。

其中：

VIDEO：

纯文本 / Prompt → Video

IMAGE_TO_VIDEO：

Image + Prompt → Video

但业务层不要写死某个 Provider。

必须：

resolveForCapability(
    projectId,
    AiCapability.VIDEO
)

或者：

resolveForCapability(
    projectId,
    AiCapability.IMAGE_TO_VIDEO
)

具体使用哪个 Capability：

如果用户选择“文生视频”：

VIDEO

如果用户选择“图生视频”：

IMAGE_TO_VIDEO

默认推荐：

IMAGE_TO_VIDEO

因为 Phase 9 已经产生 StoryboardShot 的 Final Image Asset。

因此 UI 应优先：

“基于最终图片生成视频”

其次：

“纯 Prompt 生成视频”

如果 Provider 不支持对应 Capability：

必须返回明确错误：

VIDEO_PROVIDER_NOT_CONFIGURED

或：

IMAGE_TO_VIDEO_PROVIDER_NOT_CONFIGURED

或：

CAPABILITY_NOT_SUPPORTED

绝对不能 fallback 到：

STRUCTURED_OUTPUT
CHAT
IMAGE

==================================================
三、Provider Architecture
==================================================

现有：

AiProvider
AiModel
AiProviderCapability
ProjectAiConfig
ProviderResolver

全部保留。

不要重新设计。

ProviderResolver 优先级继续：

ProjectAiConfig
→ Legacy Project.aiProviderId
→ User Provider
→ Platform Default
→ System .env
→ NO_AI_PROVIDER_CONFIGURED

但是：

VIDEO / IMAGE_TO_VIDEO：

绝对不能使用 Legacy Text Provider。

也不能因为 DeepSeek 是 Default Provider 就让 DeepSeek 执行 Video。

Resolver 必须严格检查：

provider.enabled
provider.hasApiKey
provider capability
model.enabled
model capability

必须保证：

IMAGE Provider ≠ VIDEO Provider

除非数据库明确声明该 Provider / Model 同时支持。

==================================================
四、Provider Adapter
==================================================

扩展：

AiProvider interface

增加：

generateVideo()

推荐接口：

generateVideo(input: GenerateVideoInput): Promise<GenerateVideoResult>

GenerateVideoInput 至少包含：

- prompt
- negativePrompt?
- imageUrl?
- imageBase64?
- durationSeconds?
- width?
- height?
- aspectRatio?
- fps?
- cameraMovement?
- metadata?

GenerateVideoResult 至少包含：

- url?
- base64?
- mimeType
- durationSeconds?
- width?
- height?
- provider
- model
- providerRequestId?
- metadata?

注意：

不要假设所有 Video Provider 都是同步 API。

必须从架构上允许：

同步：

POST → 返回 video URL

以及异步：

POST → operation/job id
GET → 查询状态
最终 → video URL

但是：

Phase 10 v1 可以只实现同步 Adapter。

接口设计必须为未来异步 Provider 留出空间。

==================================================
五、OpenAI Compatible Video Provider
==================================================

不要假设 OpenAI Compatible Video API 具有统一标准。

Phase 10 必须设计：

VideoProviderAdapter

不要简单地在：

OpenAiCompatibleProvider

里面写死某一个厂商的 Video API。

可以：

OpenAiCompatibleProvider.generateVideo()

但必须明确：

当前仅实现一个“兼容适配协议”。

如果实际 Provider 不支持 Video：

返回：

VIDEO_CAPABILITY_NOT_SUPPORTED

而不是：

500

也不要伪造 Video。

如果当前项目中没有真实可调用的 Video Provider：

允许系统完整实现：

- Capability
- Adapter Interface
- Provider configuration
- validation
- GenerationTask
- Preview
- Apply
- Asset
- UI

但不能制造假的视频 URL。

必须让：

“没有真实 Video Provider”

成为一个明确、正常的产品状态。

==================================================
六、数据库设计
==================================================

增量修改 Prisma。

禁止重建。

GenerationTask 已存在。

扩展：

GenerationTaskType.VIDEO

如果已经存在 VIDEO，则复用。

GenerationTask.capability：

VIDEO
或
IMAGE_TO_VIDEO

usage：

Json

成功时至少记录：

{
  durationMs,
  sourceShotId,
  sourceAssetId?,
  outputAssetCount
}

不要伪造：

token
cost
billing
price

除非 Provider 实际返回。

==================================================
七、Asset 模型
==================================================

复用：

Asset

使用：

AssetType.VIDEO

不要创建：

VideoAsset

VideoGenerationTask

VideoFile

等重复模型。

Asset 至少记录：

- id
- projectId
- type
- status
- mimeType
- storageKey
- thumbnailUrl
- width
- height
- durationSeconds
- sizeBytes
- provider
- model
- version
- generationTaskId

如果现有 Asset 已经具备这些字段：

复用。

==================================================
八、StoryboardShotAsset
==================================================

复用 Phase 9 已经存在的：

StoryboardShotAsset

一个 Shot 可以：

FINAL IMAGE
IMAGE HISTORY
VIDEO HISTORY
VIDEO FINAL

建议 role：

IMAGE：

REFERENCE
GENERATED
FINAL
THUMBNAIL

VIDEO：

GENERATED
FINAL

如果当前枚举设计不适合：

增量增加：

VIDEO_GENERATED
VIDEO_FINAL

或者采用现有 role + Asset.type 判断。

优先保持现有设计简单。

不要建立新的 ShotVideo 表。

==================================================
九、Video Generation API
==================================================

新增：

POST

/projects/:projectId/generations/video

用于：

VIDEO

新增：

POST

/projects/:projectId/generations/image-to-video

用于：

IMAGE_TO_VIDEO

请求至少支持：

{
  "shotId": "...",
  "prompt": "...",
  "negativePrompt": "...",
  "durationSeconds": 5,
  "width": 1280,
  "height": 720
}

IMAGE_TO_VIDEO 额外：

{
  "sourceAssetId": "..."
}

但：

sourceAssetId 必须：

- 属于当前 project
- 属于当前 shot
- Asset.type = IMAGE
- Asset.status = READY
- 最好是 FINAL

如果没有：

返回：

SOURCE_IMAGE_REQUIRED

==================================================
十、生成前校验
==================================================

Video Generation 调用 AI 前必须验证：

1. Project 存在
2. Shot 存在
3. Shot 属于 Project
4. Storyboard 属于 Episode
5. Episode 属于 Project
6. StoryboardShot 状态合法
7. Shot 有有效 videoPrompt 或 fallback prompt
8. IMAGE_TO_VIDEO 必须存在有效 Source Image
9. Source Image 属于当前 Project
10. Source Image 属于当前 Shot
11. Source Asset = READY
12. Provider 支持 VIDEO / IMAGE_TO_VIDEO
13. Model 支持对应 Capability
14. Provider enabled
15. API Key 存在

任意失败：

不要调用 AI。

==================================================
十一、Prompt 生成
==================================================

必须新增：

video-generation.prompt.ts

不要直接：

JSON.stringify(shot)

发送给 AI。

必须创建：

buildVideoGenerationContext()

或者：

buildVideoPrompt()

输入：

StoryboardShot

只提取：

- visualDescription
- action
- dialogue
- narration
- camera
- movement
- composition
- lighting
- mood
- style
- duration
- characters
- continuityNotes

如果 IMAGE_TO_VIDEO：

加入：

Source Image description / metadata

但不要发送：

API Key
GenerationTask 全量
内部数据库敏感字段
encryptedApiKey

==================================================
十二、Prompt 优先级
==================================================

如果用户手动修改：

videoPrompt

则：

用户修改优先。

如果没有：

自动根据 Shot 信息生成默认 Video Prompt。

最终：

用户 Prompt
>
StoryboardShot.videoPrompt
>
StoryboardShot.imagePrompt
>
visualDescription + action

不能强制覆盖用户修改。

==================================================
十三、Preview
==================================================

Video Generation：

创建：

GenerationTask

状态：

PENDING
→ RUNNING
→ SUCCEEDED / FAILED

Preview 阶段：

不创建正式 Asset。

如果 Provider 返回视频 URL：

可以：

- 下载临时文件
- 或保存临时 Preview 引用

但不要在 Apply 前创建正式 Asset。

Preview 返回：

- taskId
- status
- provider
- model
- capability
- duration
- width
- height
- previewUrl（如果安全且有效）
- sourceImage
- usage
- error

注意：

Preview URL 不应该永久依赖第三方 URL。

如果 Provider URL 是短期 URL：

需要在 Apply 时重新下载到本地 Asset Storage。

==================================================
十四、Apply
==================================================

POST：

/projects/:projectId/generations/:id/apply

继续复用现有 Generation Apply。

根据：

GenerationTask.type

分支：

VIDEO
IMAGE_TO_VIDEO

Apply：

必须 transaction。

流程：

1. 验证 task
2. 验证 project
3. 验证 task.status = SUCCEEDED
4. 验证 task.appliedAt = null
5. 验证 output
6. 下载 / 获取 Video
7. 保存到 AssetStorage
8. 创建 Asset(type=VIDEO)
9. 创建 StoryboardShotAsset
10. 设置 role = FINAL
11. 设置 isPrimary = true
12. 旧 FINAL VIDEO 设置 isPrimary=false
13. GenerationTask.appliedAt
14. commit

如果任何步骤失败：

rollback。

不要留下：

READY Asset
没有 GenerationTask appliedAt

这种脏状态。

如果 Storage 无法参与数据库 transaction：

设计补偿删除：

DB transaction 失败
→ 删除刚刚写入的文件

保证最终一致性。

==================================================
十五、Video Asset History
==================================================

必须保留历史。

用户重新生成：

v1
v2
v3

不要删除旧视频。

Shot 页面显示：

Video History

每个版本：

- version
- provider
- model
- duration
- createdAt
- status
- final / candidate

支持：

“设为最终”

如果已经 Apply：

不要重新调用 AI。

直接将历史 Asset：

isPrimary = true

并把旧 Final：

isPrimary = false

==================================================
十六、视频参数
==================================================

Phase 10 v1 支持基础参数：

- durationSeconds
- width
- height
- aspectRatio
- fps（如果 Provider 支持）
- seed（如果 Provider 支持）

但：

不要强制所有 Provider 支持。

Capability metadata 可以记录：

supportedParameters

例如：

{
  "duration": true,
  "resolution": true,
  "fps": false,
  "seed": false
}

UI 根据 Provider / Model capability 展示。

不要让用户填写 Provider 不支持的参数。

==================================================
十七、UI
==================================================

在：

/projects/:id/episodes/:episodeId/storyboard

Shot Inspector 增加：

# 视频

如果没有 VIDEO / IMAGE_TO_VIDEO Provider：

显示：

“尚未配置视频模型”

按钮：

“配置 AI”

跳转：

/projects/:id/settings#VIDEO

如果有：

显示：

视频生成方式：

○ 基于最终图片生成
○ 纯 Prompt 生成

默认：

基于最终图片生成。

==================================================
十八、Shot Video Panel
==================================================

新增：

ShotVideoPanel.vue

显示：

### Source

最终图片：

[Image Preview]

### Prompt

[可编辑 Video Prompt]

### Negative Prompt

[可编辑]

### 参数

时长：

5s

分辨率：

1280 × 720

生成模式：

Image to Video

### Provider

Provider 名称

Model 名称

### 生成

[生成视频]

==================================================
十九、Preview Modal
==================================================

生成后：

VideoGenerationPreviewModal.vue

显示：

视频 Preview

信息：

- Provider
- Model
- Capability
- Duration
- Resolution
- Source Image
- Prompt
- Generation Time

按钮：

[重新生成]

[取消]

[应用为最终视频]

如果失败：

显示：

失败原因

不要显示：

API Key

encryptedApiKey

内部 stack trace

==================================================
二十、项目 Videos 页面
==================================================

新增：

/projects/:id/videos

显示项目所有：

Asset.type = VIDEO

支持：

- Episode
- Scene
- Shot
- Provider
- Model
- Status
- Created At

点击视频：

查看详情。

不要在 Phase 10 做：

视频剪辑。

==================================================
二十一、Video Player
==================================================

使用 HTML5：

<video controls>

支持：

- play
- pause
- seek
- fullscreen
- volume

不做：

Timeline Editor。

==================================================
二十二、视频与图片关系
==================================================

Shot：

Image Final
↓
Image-to-Video
↓
Video Candidate
↓
Video Final

必须支持：

一个 Shot：

多个 Image Asset
多个 Video Asset

但：

同一时刻：

最多一个 Primary Final Image
最多一个 Primary Final Video

不要删除历史。

==================================================
二十三、Storyboard 版本控制
==================================================

如果 Shot 修改：

imagePrompt
videoPrompt
camera
duration
visualDescription
characters

则：

现有 Video 不删除。

但应提示：

“分镜已更新，当前视频可能已过期。”

可以增加：

videoStale

或者动态计算：

video.generatedFromStoryboardVersion
vs
storyboard.version

优先使用动态计算，不要每次 GET 修改数据库。

如果 Phase 9 已经有类似 stale：

复用。

==================================================
二十四、Generation Context
==================================================

Video Generation：

不要重新读取整个项目生成 Prompt。

只读取：

Project
Episode
Storyboard
StoryboardShot
Characters（必要视觉摘要）
Final Image Asset（IMAGE_TO_VIDEO）

不要注入：

Story Bible 全量
World 全量
GenerationTask 全量
所有 Character metadata
API Key

==================================================
二十五、Cost / BYOK 原则
==================================================

非常重要。

本项目最终产品不是：

平台无限免费生成。

最终商业模式：

用户自己承担 AI 成本。

因此 UI 必须明确：

“视频生成将使用当前项目配置的 AI Provider，费用由该 Provider 账户承担。”

不要：

- 自己计算虚假价格
- 写死价格
- 承诺免费
- 伪造 token cost
- 在 Phase 10 做 Billing

可以记录：

provider
model
durationMs
providerRequestId

如果 Provider 未来返回：

usage
cost

再记录。

==================================================
二十六、Provider Settings
==================================================

在：

/projects/:id/settings

AI 配置页面增加：

Video Generation
Image to Video

两个 Capability。

显示：

未配置

已配置

Provider

Model

Status

Source

如果用户点击：

配置

跳转 Provider / Model 配置。

不能把 Video 默认绑定到 DeepSeek。

如果当前平台 Default Provider 没有 VIDEO：

显示：

“平台默认 Provider 不支持视频生成，请配置自己的 Video Provider。”

==================================================
二十七、Platform Provider
==================================================

继续保留 Platform Default。

但是：

Platform Default Video Provider 只是：

Demo / Development / Platform capability。

不要设计成：

平台长期承担用户生产成本。

UI 文案：

“默认 Provider 仅用于开发 / Demo。正式生产建议配置自己的 Provider。”

==================================================
二十八、Image Provider 与 Video Provider
==================================================

必须允许：

Provider A：
IMAGE

Provider B：
VIDEO

Provider C：
IMAGE_TO_VIDEO

Provider D：
IMAGE + VIDEO + IMAGE_TO_VIDEO

例如：

项目可以：

IMAGE = Provider A / Model A

VIDEO = Provider B / Model B

IMAGE_TO_VIDEO = Provider C / Model C

这是 Phase 4.6 Capability Architecture 的核心价值。

不要简化成：

一个项目只能使用一个 AI Provider。

==================================================
二十九、错误码
==================================================

新增必要错误：

VIDEO_PROVIDER_NOT_CONFIGURED

IMAGE_TO_VIDEO_PROVIDER_NOT_CONFIGURED

VIDEO_CAPABILITY_NOT_SUPPORTED

IMAGE_TO_VIDEO_CAPABILITY_NOT_SUPPORTED

SOURCE_IMAGE_REQUIRED

SOURCE_IMAGE_NOT_FOUND

SOURCE_IMAGE_NOT_READY

SOURCE_IMAGE_PROJECT_MISMATCH

SOURCE_IMAGE_SHOT_MISMATCH

SHOT_NOT_FOUND

SHOT_PROJECT_MISMATCH

VIDEO_GENERATION_FAILED

VIDEO_DOWNLOAD_FAILED

VIDEO_ASSET_APPLY_FAILED

GENERATION_ALREADY_APPLIED

VIDEO_NOT_FOUND

所有错误：

不能包含 API Key。

==================================================
三十、Security
==================================================

必须确保：

GET /ai/providers

绝对不返回：

apiKey

encryptedApiKey

Authorization

secret

Provider 原始错误如果包含 Key：

必须 sanitize。

日志：

不能打印：

Bearer xxx

API Key

完整 Provider request header。

==================================================
三十一、API Client
==================================================

所有 Web API：

必须通过：

packages/api-client

新增：

createVideoGeneration()

createImageToVideoGeneration()

getVideoAssets()

getShotVideoAssets()

setPrimaryVideoAsset()

getVideoGeneration()

applyVideoGeneration()

如果已有通用 Generation API：

可以复用。

不要重复实现 fetch。

==================================================
三十二、Shared Types
==================================================

新增：

VideoGenerationInput

ImageToVideoGenerationInput

VideoGenerationResult

VideoGenerationPreview

VideoAsset

VideoGenerationUsage

VideoGenerationMode

例如：

enum / union：

TEXT_TO_VIDEO
IMAGE_TO_VIDEO

不要在：

Web
API
Mobile
Desktop

分别定义相同 Interface。

统一放：

packages/types

==================================================
三十三、Core
==================================================

packages/core 新增：

video.ts

包含纯函数：

buildVideoPrompt()

validateVideoGenerationInput()

resolveVideoGenerationMode()

normalizeVideoGenerationResult()

不要放：

Prisma
NestJS
Vue
fs

Core 保持纯业务逻辑。

==================================================
三十四、Tests
==================================================

必须新增完整测试。

至少覆盖：

1. VIDEO Capability Resolver
2. IMAGE_TO_VIDEO Capability Resolver
3. Provider disabled
4. Provider missing key
5. Model disabled
6. Model capability mismatch
7. IMAGE Provider 不得用于 VIDEO
8. DeepSeek Text Provider 不得用于 VIDEO
9. Project Provider isolation
10. Cross-project Shot rejected
11. Source Image required
12. Source Image project mismatch
13. Source Image shot mismatch
14. Source Image not READY
15. Preview 不创建 Asset
16. Provider 失败 → GenerationTask FAILED
17. Video success → SUCCEEDED
18. Apply 创建 Asset
19. Apply 创建 StoryboardShotAsset
20. Apply 设置 FINAL
21. Apply 旧 FINAL 取消 primary
22. Apply transaction rollback
23. Duplicate Apply rejected
24. Regeneration 保留历史
25. setPrimaryVideoAsset
26. stale video detection
27. API Key 不泄漏
28. Provider error sanitize
29. VIDEO generation usage
30. IMAGE_TO_VIDEO sourceAssetId
31. API Client tests
32. Schema validation
33. Project isolation
34. GenerationTask capability 正确
35. Asset.type = VIDEO

目标：

Phase 10 完成后：

至少：

45+ test files

或者在现有测试架构基础上合理增加测试。

不要为了凑数量写无意义测试。

==================================================
三十五、Migration
==================================================

如果确实需要数据库变化：

创建新的 migration：

20260817190000_video_generation_engine

或者根据实际当前时间生成合理名称。

必须：

pnpm prisma migrate deploy

禁止：

prisma migrate reset

禁止：

DROP

禁止：

修改历史 migration。

==================================================
三十六、Seed
==================================================

新增：

prisma/seed-video-demo.ts

命令：

pnpm db:seed:video-demo

不要自动执行。

不要调用真实 AI。

如果没有真实视频：

可以创建 placeholder / fixture。

但必须明确：

Demo Asset

不要伪装成真实 AI 生成的视频。

如果可以生成一个简单 MP4 fixture：

优先。

否则：

Asset.status 可以使用 READY + fixture 文件。

但 metadata 必须：

{
  "demo": true,
  "source": "seed"
}

==================================================
三十七、不要做自动批量生成
==================================================

Phase 10 v1：

一次只生成一个 Shot。

不要实现：

“整集生成所有视频”

不要实现：

“全季生成所有视频”

不要实现：

“项目一键生成整部剧”

原因：

后续需要：

Queue
Worker
Concurrency
Retry
Cost Control
Rate Limit
Cancellation

这些属于未来异步生产系统。

==================================================
三十八、不要实现真正的异步队列
==================================================

虽然 Video Provider 很可能未来是：

submit job
→ polling
→ callback

但是 Phase 10 v1：

保持当前 GenerationExecutor 架构。

只定义：

VideoGenerationProvider

未来可以扩展：

startGeneration()
getGenerationStatus()
cancelGeneration()

但本阶段：

不接 Redis
不接 BullMQ
不接 Worker。

==================================================
三十九、未来兼容异步 Provider
==================================================

设计接口时预留：

GenerationMode：

SYNC
ASYNC

Provider 能力 metadata：

supportsAsync

但是：

当前实现可以：

SYNC

未来 Phase 10.x：

再增加异步 Job。

不要现在提前实现复杂系统。

==================================================
四十、Asset Storage
==================================================

继续复用：

AssetStorageService

视频文件：

storage/assets/{projectId}/{assetId}/...

例如：

storage/assets/{projectId}/{assetId}/video.mp4

不要：

把 MP4 Base64 存 PostgreSQL。

必须：

文件 → Storage

DB：

只存 metadata。

==================================================
四十一、文件安全
==================================================

Asset file route：

GET

/projects/:projectId/assets/:assetId/file

必须：

Project isolation。

用户不能通过：

assetId

读取其他 Project 文件。

禁止：

path traversal。

storageKey 必须不能直接拼接用户任意路径。

==================================================
四十二、视频格式
==================================================

优先支持：

video/mp4

可以预留：

video/webm

如果 Provider 返回其他格式：

必须检查。

不要直接假设：

所有 URL 都是 MP4。

Asset.mimeType 必须真实。

==================================================
四十三、Thumbnail
==================================================

如果 Provider 返回视频但没有 thumbnail：

Phase 10 v1 可以：

不生成 thumbnail。

UI：

使用 video player。

或者如果本地 ffmpeg 已经存在并且项目已有统一媒体工具：

可以生成 thumbnail。

但：

不要为了 thumbnail 引入 FFmpeg。

不要新增复杂媒体依赖。

==================================================
四十四、前端交互
==================================================

Shot Card 增加：

🎬 视频状态：

未生成
生成中
有候选
已完成
分镜已更新

点击：

“生成视频”

打开：

ShotVideoPanel

生成过程中：

按钮 disabled。

生成失败：

可以：

重新生成

不能：

自动无限 retry。

==================================================
四十五、最终 Shot 工作流
==================================================

完整用户流程：

1.
用户进入：

星河碰撞
→ Season 1
→ Episode 1
→ Storyboard

2.
选择 Shot

3.
看到：

Final Image

4.
点击：

“基于图片生成视频”

5.
系统：

resolve IMAGE_TO_VIDEO

6.
检查：

Provider
Model
Capability
API Key
Source Asset

7.
生成：

GenerationTask

8.
Provider 调用

9.
成功：

Preview

10.
用户观看：

Preview Video

11.
点击：

应用为最终视频

12.
系统：

Asset(type=VIDEO)

13.
StoryboardShotAsset：

role=FINAL

14.
Shot：

Video Ready

15.
历史：

Video v1

16.
用户修改 Prompt

17.
重新生成：

Video v2

18.
v1 保留

19.
用户可以：

“设为最终”

==================================================
四十六、对后续阶段的架构要求
==================================================

Phase 10 完成后必须能够自然扩展：

Phase 11：

TTS / Voice

Phase 12：

Music / Sound Effects

Phase 13：

Subtitle / Caption

Phase 14：

Video Editing Engine

Phase 15：

Timeline / FFmpeg

Phase 16：

Episode Render

Phase 17：

Batch Generation

Phase 18：

Async Queue / Worker

Phase 19：

Cost Tracking / Billing

Phase 20：

User / Auth / BYOK

Phase 21：

Publishing

YouTube
TikTok
Bilibili
Instagram

所以：

不要把：

Video

直接做成：

Episode final video。

Video 只是：

StoryboardShot 的 Asset。

最终：

Shot Video
↓
Timeline
↓
Episode Render
↓
Final Episode Video

==================================================
四十七、非常重要：不要提前做“完整成片”
==================================================

本阶段绝对不要实现：

Shot 1
+
Shot 2
+
Shot 3
↓
完整 Episode.mp4

这是后面的 Editing / Render Engine。

本阶段只做到：

一个 Shot → 一个 Video Asset。

==================================================
四十八、产品定位
==================================================

请始终理解这个项目最终目标：

AI Drama Studio

不是：

AI Chatbot。

也不是：

简单 AI 图片生成器。

而是：

Story Idea
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
Image
↓
Video
↓
Voice
↓
Music
↓
Timeline
↓
Render
↓
Publish

最终用户只需要提供：

故事大纲
+
每季大纲
+
每集大纲
+
必要的创作偏好

系统负责把它逐层结构化为：

可以真正生产的影视资产。

因此每一个阶段都必须：

结构化
可编辑
可追溯
可重新生成
可版本化
可回滚
可更换 Provider
可更换 Model
可由用户承担 AI 成本。

==================================================
四十九、用户成本模型
==================================================

必须继续贯彻：

BYOK / User-Owned AI Cost。

最终：

用户可以自己配置：

World Model
Character Model
Story Model
Storyboard Model
Image Model
Image-to-Video Model
Video Model
TTS Model
Voice Clone Model
Music Model

不同能力可以使用不同 Provider。

例如：

Structured Output
→ DeepSeek

Image
→ Flux / 用户自己的 Image Provider

Image-to-Video
→ 用户自己的 Video Provider

TTS
→ 用户自己的 TTS Provider

Music
→ 用户自己的 Music Provider

不要设计成：

一个 API Key 解决所有能力。

也不要设计成：

平台统一承担所有 AI 成本。

==================================================
五十、完成后必须执行
==================================================

自动执行：

pnpm prisma validate

pnpm lint

pnpm typecheck

pnpm test

如果项目存在：

pnpm build

也执行。

检查：

API 3011

Web 3010

健康：

GET /health

必须正常。

==================================================
五十一、数据安全检查
==================================================

完成后检查：

Project 数量

World 数量

Character 数量

Season 数量

Episode 数量

Script 数量

Storyboard 数量

StoryboardShot 数量

Asset 数量

AiProvider 数量

GenerationTask 数量

确保没有因为 migration / seed / test：

删除现有数据。

特别检查：

“星河碰撞”

World：

“灵械纪元”

Characters：

沈星河
太虚真人
艾尔

Season 1

E01
E02
E03

全部保留。

==================================================
五十二、Seed 不允许自动执行
==================================================

禁止把：

seed-video-demo

加入：

dev

postinstall

migration

startup

API bootstrap

必须手动：

pnpm db:seed:video-demo

==================================================
五十三、完成后的输出报告
==================================================

Phase 10 完成后，不要继续自动开始 Phase 11。

输出完整报告：

1. 修改文件
2. 新增 Prisma Model
3. 新增字段
4. Migration 名称
5. 是否执行 Migration
6. Video Capability 架构
7. IMAGE_TO_VIDEO Capability 架构
8. Provider Adapter
9. Generation API
10. Preview / Apply
11. Asset Storage
12. StoryboardShotAsset
13. Video History
14. Stale 检测
15. UI
16. Project Videos
17. Seed
18. Tests
19. lint
20. typecheck
21. prisma validate
22. build
23. API health
24. 当前数据库数据是否保留
25. Provider 是否保留
26. GenerationTask 是否保留
27. 是否修改 Monorepo
28. 是否存在遗留问题
29. 当前没有实现什么
30. 下一阶段建议

==================================================
五十四、Phase 10 完成标准
==================================================

只有以下全部满足，Phase 10 才算完成：

[ ] VIDEO Capability 完整支持
[ ] IMAGE_TO_VIDEO Capability 完整支持
[ ] ProviderResolver 正确工作
[ ] ProjectAiConfig 正确工作
[ ] Model Capability 校验正确
[ ] Source Image 校验正确
[ ] StoryboardShot 是唯一上游
[ ] GenerationTask 正确记录
[ ] Preview 不创建正式 Asset
[ ] Apply 创建 Asset
[ ] Asset.type = VIDEO
[ ] StoryboardShotAsset 正确关联
[ ] Final Video 正确管理
[ ] Video History 保留
[ ] 重新生成不删除旧视频
[ ] setPrimaryVideoAsset 工作
[ ] Project Isolation 正确
[ ] API Key 不泄漏
[ ] Provider 错误脱敏
[ ] Video Prompt 可编辑
[ ] Shot Inspector 有视频生成 UI
[ ] Project Videos 页面存在
[ ] HTML5 Video Player 可用
[ ] 没有 Video Provider 时有明确提示
[ ] DeepSeek Chat 不会被误用于 Video
[ ] 没有伪造真实 Video Provider
[ ] 没有引入 Billing
[ ] 没有引入 Login
[ ] 没有引入 Redis
[ ] 没有引入 BullMQ
[ ] 没有引入 FFmpeg Timeline
[ ] 没有实现整集自动生成
[ ] 没有实现批量视频生成
[ ] 没有删除历史数据
[ ] Migration 成功
[ ] prisma validate 通过
[ ] lint 通过
[ ] typecheck 通过
[ ] test 全部通过
[ ] API health 正常
[ ] Web 正常

==================================================
五十五、自动执行规则
==================================================

你现在直接开始实施 Phase 10。

不要询问我是否继续。

不要等待确认。

遇到可以合理判断的实现细节，按照以上架构自动选择最稳妥方案。

如果发现当前代码与 Prompt 存在差异：

优先：

1. 保留现有架构
2. 增量修改
3. 保持向后兼容
4. 不删除历史数据
5. 不改变现有 API 行为
6. 不破坏 Phase 4–9

如果发现某个功能依赖尚未实现的能力：

不要为了完成 Phase 10 擅自扩大范围。

例如：

没有真实 Video Provider：

不要自己新增一个假的 Provider。

没有异步 Worker：

不要自己引入 BullMQ。

没有 FFmpeg：

不要为了视频 thumbnail 引入 FFmpeg。

没有 Billing：

不要实现计费。

用清晰的：

NOT_CONFIGURED
NOT_SUPPORTED
COMING_SOON

状态处理。

完成 Phase 10 后：

停止。

不要自动进入 Phase 11。