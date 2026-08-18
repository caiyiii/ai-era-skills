# Phase 14 — Local Render Engine / FFmpeg Episode MP4 v1

你现在开始执行：

# Phase 14 — Local Render Engine / FFmpeg Episode MP4 v1

这是 AI Drama Studio 的第 14 阶段。

==================================================
一、最高优先级目标
=========

Phase 13 已经完成：

```text
Project
→ World
→ Characters
→ Story Bible
→ Season
→ Episode
→ Script
→ Storyboard
→ Image Assets
→ Video Assets
→ Dialogue TTS
→ Music
→ SFX
→ Timeline
→ Composition Manifest
→ Browser Composition Preview
```

Phase 14 的目标是第一次真正打通：

```text
LOCKED Timeline
      ↓
Render Job
      ↓
Immutable Manifest Snapshot
      ↓
Local Render Worker
      ↓
FFmpeg
      ↓
Episode MP4
      ↓
Render Artifact
```

最终必须能够真实生成一个：

```text
.mp4
```

而不是：

```text
假 MP4
假 Render
假完成状态
假进度
假 Worker
```

必须使用真实 FFmpeg 执行真实视频编码。

==================================================
二、Phase 13 当前已知基线
=================

Phase 13 已完成以下内容：

* EpisodeTimeline
* TimelineTrack
* TimelineClip
* TimelineStatus
* TimelineTrackType
* TimelineClipType
* TimelineClipSourceType
* Timeline Builder
* Timeline Timing
* Timeline Continuity
* Timeline Stale
* Timeline Rebuild
* Composition Manifest
* Composition Preview
* Web Timeline
* Mobile 基础查看
* Desktop 基础查看
* Timeline CRUD
* Track CRUD
* Clip CRUD

Phase 13 最终状态：

```text
64 files / 406 tests
pnpm lint           PASS
pnpm typecheck      PASS
pnpm prisma validate PASS
pnpm build          PASS
API health          PASS
```

Phase 13 使用：

```text
20260818110000_episode_timeline_composition
```

Migration 已执行。

数据库没有 reset。

没有 DROP。

没有删除历史数据。

当前数据库曾确认：

```text
Project          = 1
World            = 1
Character        = 3
Season           = 1
Episode          = 3
Provider         = 1
GenerationTask   = 3
Script           = 0
Storyboard       = 0
Asset            = 0
```

因此：

不要假设数据库当前已经存在 Timeline Demo 数据。

不要自动 reset。

不要自动创建重复的上游 Demo 数据。

如果需要真实 Render Demo：

先检查当前数据库。

如果上游 Script / Storyboard / Asset 不存在：

明确提示用户需要先执行已有 seed。

不要偷偷 reset。

==================================================
三、严格执行纪律
========

绝对不要重建 Monorepo。

绝对不要换技术栈。

继续使用：

```text
pnpm
Turborepo
Nuxt 3
Vue 3
Tailwind
Pinia
VueUse
NestJS
Ionic Vue
Tauri 2
Prisma
PostgreSQL
```

不要删除历史 migration。

不要修改历史 migration。

只允许新增 migration。

绝对禁止：

```text
prisma migrate reset
DROP TABLE
删除现有数据
重新初始化数据库
```

必须保证现有功能不被破坏。

最终必须通过：

```text
pnpm prisma validate
pnpm lint
pnpm typecheck
pnpm test
pnpm build
```

==================================================
四、Phase 14 明确边界
===============

本阶段实现：

```text
Render Domain
Render Job
Render Manifest Snapshot
Render Worker
FFmpeg Adapter
Video Composition
Image Composition
Dialogue Audio
Music
SFX
MP4 Output
Render Artifact
Render Progress
Render Failure
Render Retry
Render Cancellation
Web Render UI
```

本阶段绝对不要实现：

```text
Redis
BullMQ
Cloud Render
GPU Render
Kubernetes
分布式 Worker
云存储 Provider
YouTube
TikTok
Bilibili
Instagram
Billing
Subscription
Login
JWT
User Account
复杂权限系统
AI 自动剪辑
AI 自动补镜
AI 自动补声
AI 自动配乐
AI Provider
Render Provider
```

不要提前实现 Phase 15 / Phase 16 / Phase 17 / Phase 18。

==================================================
五、核心架构原则
========

Phase 14 最重要的架构原则：

# Render 必须消费 Snapshot，而不是实时读取 Timeline。

正确：

```text
Timeline LOCKED
      ↓
CompositionManifest
      ↓
RenderManifestSnapshot
      ↓
RenderJob
      ↓
Worker
      ↓
FFmpeg
```

错误：

```text
Timeline
      ↓
Worker
      ↓
不断读取数据库
      ↓
FFmpeg
```

原因：

Render 开始之后，Timeline 不应该再影响当前 Render。

例如：

```text
Timeline version = 5
RenderJob 创建
Snapshot version = 5
```

此时即使用户：

```text
unlock Timeline
修改 Clip
重新 build Timeline
version = 6
```

正在执行的 Render Job：

```text
仍然必须渲染 version = 5
```

不能自动切换到 version 6。

==================================================
六、RenderJob 数据模型
================

新增：

```text
RenderJob
```

建议至少包含：

```text
id
projectId
episodeId
timelineId
timelineVersion

status

manifestSnapshot

outputFormat
outputContainer

width
height
fps

durationSeconds

progress
currentStage

outputArtifactId

errorCode
errorMessage

attempt
maxAttempts

startedAt
completedAt
cancelledAt
failedAt

createdAt
updatedAt
```

如果现有项目命名规范不同：

以现有项目实际代码风格为准。

不要机械复制上述字段。

==================================================
七、RenderJob Status
==================

新增：

```text
RenderJobStatus
```

至少：

```text
QUEUED
PREPARING
RENDERING
SUCCEEDED
FAILED
CANCEL_REQUESTED
CANCELLED
```

不要加入：

```text
EXPORTED
PUBLISHED
UPLOADING
UPLOADED
```

这些属于未来阶段。

RenderJobStatus 和 TimelineStatus 必须保持职责分离。

Timeline：

```text
DRAFT
PREVIEW_READY
STALE
LOCKED
```

Render：

```text
QUEUED
PREPARING
RENDERING
SUCCEEDED
FAILED
CANCEL_REQUESTED
CANCELLED
```

==================================================
八、RenderJobEvent
================

建议新增：

```text
RenderJobEvent
```

用于保存 Render 生命周期事件。

至少支持：

```text
QUEUED
PREPARING
FFMPEG_STARTED
FFMPEG_PROGRESS
FFMPEG_COMPLETED
ARTIFACT_CREATED
SUCCEEDED
FAILED
CANCEL_REQUESTED
CANCELLED
```

建议字段：

```text
id
renderJobId
type
message
progress
metadata
createdAt
```

metadata 中：

禁止保存：

```text
apiKey
encryptedApiKey
provider secret
credential
```

RenderJobEvent 不是完整日志系统。

本阶段只保存关键生命周期事件。

==================================================
九、Render Artifact
=================

Render 成功后必须有真实输出文件。

建议新增：

```text
RenderArtifact
```

至少：

```text
id
projectId
episodeId
renderJobId

type
storageKey
mimeType

fileSize
durationSeconds

width
height
fps

createdAt
updatedAt
```

type 至少：

```text
EPISODE_VIDEO
```

如果项目已有通用 Artifact / File / Asset 模型：

优先复用现有架构。

不要重复创建一个与现有文件存储完全冲突的新体系。

但是：

Render Artifact 必须能够独立表示：

```text
一个真实生成出来的 MP4
```

==================================================
十、Manifest Snapshot
===================

RenderJob 创建时：

必须调用现有：

```text
CompositionService
```

或者等价的 Composition Engine。

生成：

```text
CompositionManifest
```

然后深拷贝保存到：

```text
RenderJob.manifestSnapshot
```

必须保证：

```text
RenderJob.manifestSnapshot
```

是不可变快照。

后续 Timeline 修改：

不能修改 snapshot。

Render Worker：

只允许消费 snapshot。

不要在 Worker 中重新调用：

```text
TimelineBuilderService
```

不要在 Worker 中重新读取：

```text
Storyboard
Script
TimelineClip
TimelineTrack
```

Worker 只消费：

```text
RenderJob
+
manifestSnapshot
```

==================================================
十一、Render 创建规则
==============

新增 API：

```http
POST
/projects/:projectId/episodes/:episodeId/render
```

创建 Render Job。

流程：

```text
1. 查询 Episode
2. 校验 Project ownership
3. 查询 Timeline
4. 检查 Timeline 是否存在
5. 检查 Timeline 是否 LOCKED
6. 检查 Timeline 是否 STALE
7. 生成 CompositionManifest
8. 创建 manifest snapshot
9. 创建 RenderJob
10. status = QUEUED
11. 返回 RenderJob
```

如果 Timeline：

```text
DRAFT
```

拒绝。

如果：

```text
PREVIEW_READY
```

拒绝。

如果：

```text
STALE
```

拒绝。

只有：

```text
LOCKED
```

允许 Render。

==================================================
十二、Render Lock 规则
=================

Phase 13 已经提供：

```text
LOCKED
```

Phase 14 必须正式使用这个状态。

Render API 不允许自动：

```text
DRAFT → LOCKED
```

也不允许：

```text
PREVIEW_READY → LOCKED
```

用户必须先明确 Lock Timeline。

推荐流程：

```text
Timeline
  ↓
Preview
  ↓
用户确认
  ↓
LOCK
  ↓
Render
```

Render 不得偷偷修改 Timeline 状态。

==================================================
十三、STALE 检查
===========

Render 前必须重新检查：

```text
Timeline.status
```

以及：

```text
detectTimelineStale()
```

如果 Timeline 当前已经过期：

拒绝 Render。

错误：

```text
TIMELINE_STALE
```

不要自动 rebuild。

不要自动修改 Timeline。

不要自动生成新的素材。

正确行为：

```text
Timeline stale
↓
提示用户重新 Build
↓
Preview
↓
LOCK
↓
Render
```

==================================================
十四、Render Job 创建幂等性
===================

必须考虑重复点击 Render。

如果相同：

```text
projectId
episodeId
timelineId
timelineVersion
```

已经存在：

```text
QUEUED
PREPARING
RENDERING
```

状态的 RenderJob：

不要重复创建。

返回已有 Job。

如果：

```text
SUCCEEDED
```

允许用户显式重新 Render。

重新 Render：

创建新的 RenderJob。

不能覆盖旧 RenderJob。

Render 历史必须保留。

==================================================
十五、Render Service
=================

不要创建一个巨大的 RenderService。

至少拆成：

```text
RenderService
RenderManifestService
RenderWorkerService
FFmpegService
RenderArtifactService
RenderProgressService
```

职责：

```text
RenderService
→ 创建 / 查询 / 取消 / Retry

RenderManifestService
→ Timeline → Manifest Snapshot

RenderWorkerService
→ 执行 Render Job

FFmpegService
→ FFmpeg 进程抽象

RenderArtifactService
→ 输出文件保存与读取

RenderProgressService
→ FFmpeg progress → RenderJob progress
```

如果现有项目结构更适合：

可以合理调整。

核心原则：

不要把：

```text
DB
Manifest
Worker
FFmpeg
Storage
API
```

全部塞到一个 Service。

==================================================
十六、Local Render Worker
======================

本阶段使用：

```text
Local Render Worker
```

不要引入：

```text
Redis
BullMQ
RabbitMQ
Kafka
SQS
```

Worker 可以是：

```text
NestJS background service
```

或者：

```text
独立 Node process
```

但必须根据现有 Monorepo 实际架构选择。

优先：

```text
简单
可靠
可测试
Windows 可运行
```

Worker 必须能够：

```text
领取 QUEUED RenderJob
↓
PREPARING
↓
构建 FFmpeg 输入
↓
执行 FFmpeg
↓
读取 progress
↓
更新 RENDERING
↓
完成
↓
创建 RenderArtifact
↓
SUCCEEDED
```

==================================================
十七、不要假设 FFmpeg 已安装
==================

必须先检查当前机器环境。

检查：

```text
ffmpeg -version
```

或者项目配置的 FFmpeg binary。

如果不存在：

不能伪造成功。

必须返回：

```text
FFMPEG_NOT_FOUND
```

并明确告诉用户：

需要安装 FFmpeg 或配置：

```text
FFMPEG_PATH
```

如果项目已有 FFmpeg 路径配置：

优先复用。

不要在 Windows 上偷偷下载二进制。

不要把大型 FFmpeg binary commit 到 Git。

==================================================
十八、FFmpeg Adapter
=================

新增：

```text
FFmpegService
```

必须通过：

```text
child_process.spawn
```

或项目现有等价安全进程 API。

禁止：

```text
exec("...用户输入...")
```

避免命令注入。

所有输入：

必须通过：

```text
argv[]
```

传递。

不要拼接 shell command。

==================================================
十九、FFmpeg 输入安全
==============

RenderManifest 中的：

```text
asset URL
storageKey
filename
```

都属于不可信输入。

禁止直接拼接：

```text
ffmpeg -i ${url}
```

必须使用参数数组。

同时检查：

```text
projectId
assetId
storage ownership
```

禁止跨项目读取 Asset。

继续使用：

```text
AssetStorageService
```

如果已有：

```text
AssetStorageService
```

不要让 Web / Worker 直接访问数据库 storage filesystem。

==================================================
二十、Asset → FFmpeg Input
=======================

Render Worker 必须能够把 Manifest 中的 Asset 引用解析成 FFmpeg 可以读取的输入。

推荐：

```text
Manifest Asset Reference
↓
AssetStorageService
↓
Local temporary render input
↓
FFmpeg
```

如果现有 AssetStorageService 已经能够提供可读路径：

直接复用。

如果只能提供 API URL：

Render Worker 必须使用项目已有安全 Asset API / Storage Service 获取数据。

不要自己创建另一套 Storage。

==================================================
二十一、临时 Render Workspace
=======================

每一个 RenderJob 使用独立临时目录：

例如：

```text
tmp/render/<renderJobId>/
```

目录至少：

```text
inputs/
work/
output/
logs/
```

不要使用固定目录。

不要让两个 RenderJob 共用：

```text
input.mp4
output.mp4
concat.txt
```

避免并发冲突。

Render 完成后：

可以清理临时文件。

但是：

如果失败：

保留足够的诊断信息，或者保存安全的 FFmpeg stderr 摘要。

不要无限保留大型临时文件。

==================================================
二十二、第一版 Render 能力范围
===================

Phase 14 v1 必须真实支持：

```text
VIDEO
IMAGE
DIALOGUE AUDIO
MUSIC
SFX
```

最终：

```text
MP4
```

必须真实存在。

==================================================
二十三、Video Composition
=====================

Manifest：

```text
VIDEO Track
IMAGE Track
```

必须根据：

```text
startTime
duration
sourceStartTime
sourceDuration
speed
opacity
zIndex
```

构建最终视频。

第一版不需要复杂转场。

不要实现：

```text
fade transition
wipe
slide
zoom transition
AI transition
```

==================================================
二十四、Video 优先规则
==============

与 Phase 13 保持一致。

如果同一时间存在：

```text
VIDEO
IMAGE
```

必须根据：

```text
zIndex
```

选择视觉层。

如果没有显式层级：

保持 Phase 13 的：

```text
VIDEO 优先于 IMAGE
```

不能改变 Phase 13 已确定的行为。

==================================================
二十五、Image Composition
=====================

Image Clip 必须可以作为视频输入。

例如：

```text
Image
+
duration
```

转换成：

```text
video segment
```

但：

不要修改 Asset。

不要把 Image Asset 转换成新的 AI Asset。

Render 时只是：

```text
Image → FFmpeg video frame sequence
```

最终仍然属于 Render 临时过程。

==================================================
二十六、Image / Video 分辨率
=====================

必须使用 Timeline / Manifest：

```text
resolution
fps
aspectRatio
```

作为最终输出标准。

如果输入 Asset 分辨率不同：

使用 FFmpeg scale / pad 等确定性处理。

默认：

不要拉伸变形。

优先：

```text
scale
+
pad
```

保持原始 aspect ratio。

具体算法以当前项目已有 Composition Manifest 约定为准。

==================================================
二十七、FPS
=======

Render 必须使用 Manifest：

```text
fps
```

而不是：

```text
Asset fps
```

作为最终输出 FPS。

如果输入视频 FPS 不同：

FFmpeg 负责转换。

==================================================
二十八、Audio Composition
=====================

必须真实合成：

```text
Dialogue
Music
SFX
```

支持：

```text
Track muted
Track volume
Clip volume
```

最终：

```text
effectiveVolume
=
Track.volume × Clip.volume
```

保持 Phase 13 行为。

==================================================
二十九、Audio 不实现高级效果
=================

本阶段不要实现：

```text
Fade
Ducking
Compressor
Limiter
EQ
Reverb
Pan
Noise Reduction
Voice Enhancement
Loudness Mastering
```

只做：

```text
基础音量
基础静音
基础时间定位
基础混音
```

==================================================
三十、Music 行为
===========

保持 Phase 13：

```text
startTime = 0
```

如果：

```text
Music.duration > Timeline.duration
```

截断。

如果：

```text
Music.duration < Timeline.duration
```

不要循环。

不要自动生成 Music。

==================================================
三十一、SFX 行为
==========

保持 Phase 13：

```text
shotId → Shot 时间
sceneId → Scene 时间
episode-level → 0 秒
```

不增加 AI 自动音效定位。

Render 只执行 Timeline 已经决定好的时间。

==================================================
三十二、Dialogue 行为
===============

Render 不重新计算 Dialogue Timing。

Timeline 已经决定：

```text
startTime
duration
```

Render 只负责执行。

如果 Dialogue：

```text
startTime = 10
duration = 5
```

则 FFmpeg 必须把音频放到：

```text
10s → 15s
```

允许跨 Shot。

==================================================
三十三、缺失 Asset
============

必须保持 Phase 13 的 best-effort 思路。

如果 Timeline 中：

```text
missingVisualAsset
```

则 Render：

默认应该拒绝正式 MP4 Render。

原因：

Preview 可以 best-effort。

Final Render 不应该生成一个看似完整但缺镜头的视频。

因此：

```text
Preview:
允许缺失

Render:
默认拒绝缺失关键素材
```

错误：

```text
RENDER_MISSING_REQUIRED_ASSET
```

返回：

```text
缺少 X 个视觉素材
缺少 X 个对白音频
...
```

不要：

```text
自动补素材
```

不要：

```text
调用 AI
```

==================================================
三十四、是否允许“部分 Render”
===================

本阶段：

不实现用户选择：

```text
best effort render
```

也不实现：

```text
render selected clips
```

默认：

Render = 整集。

必须要求完整 Render Manifest。

==================================================
三十五、Render Progress
===================

必须支持真实进度。

不能：

```text
0%
50%
100%
```

假跳。

应该解析 FFmpeg：

```text
out_time
duration
progress
```

计算：

```text
0 → 100
```

例如：

```text
QUEUED       0%
PREPARING    5%
RENDERING    5% → 95%
FINALIZING   95% → 100%
SUCCEEDED    100%
```

如果 FFmpeg 无法提供精确 progress：

允许：

```text
indeterminate
```

但不能伪造精确百分比。

==================================================
三十六、Render Current Stage
========================

RenderJob：

```text
currentStage
```

至少：

```text
QUEUED
PREPARING
ENCODING_VIDEO
MIXING_AUDIO
FINALIZING
COMPLETED
```

不要让 stage 与 RenderJobStatus 混淆。

例如：

```text
status = RENDERING
currentStage = ENCODING_VIDEO
```

==================================================
三十七、Render Cancellation
=======================

新增：

```http
POST
/projects/:projectId/render-jobs/:renderJobId/cancel
```

行为：

如果：

```text
QUEUED
```

直接：

```text
CANCELLED
```

如果：

```text
PREPARING
RENDERING
```

设置：

```text
CANCEL_REQUESTED
```

Worker 检测后：

终止 FFmpeg process。

然后：

```text
CANCELLED
```

必须真实终止进程。

不能只是数据库：

```text
status = CANCELLED
```

而 FFmpeg 仍然运行。

==================================================
三十八、Render Retry
================

新增：

```http
POST
/projects/:projectId/render-jobs/:renderJobId/retry
```

只允许：

```text
FAILED
```

状态 Retry。

不要 Retry：

```text
SUCCEEDED
RENDERING
QUEUED
CANCELLED
```

Retry 应该：

创建新的 RenderJob。

必须复用原来的：

```text
manifestSnapshot
timelineVersion
```

不要重新读取当前 Timeline。

这样：

```text
Render Job #1
version 5
FAILED

Retry

Render Job #2
version 5
```

保持完全一致的 Render 输入。

==================================================
三十九、失败处理
========

FFmpeg：

```text
exit code != 0
```

必须：

```text
FAILED
```

保存：

```text
errorCode
errorMessage
```

以及有限长度的：

```text
stderr
```

摘要。

不要把整个巨大 stderr 无限写入数据库。

日志中：

禁止：

```text
API Key
encryptedApiKey
provider secret
```

==================================================
四十、Render Artifact 生成
=====================

只有：

```text
FFmpeg exit code = 0
```

并且：

```text
output.mp4
```

真实存在且：

```text
fileSize > 0
```

才允许：

```text
SUCCEEDED
```

然后：

```text
RenderArtifact
```

必须真实创建。

检查：

```text
exists
size
mimeType
duration
width
height
fps
```

如果输出不存在：

必须：

```text
FAILED
```

不能：

```text
SUCCEEDED
```

==================================================
四十一、Artifact Storage
====================

优先使用现有：

```text
AssetStorageService
```

如果当前服务不适合 Render Artifact：

在其上建立最小扩展。

不要创建第二套完全独立的 Storage Architecture。

Render Artifact 必须支持：

```text
projectId
episodeId
renderJobId
storageKey
```

确保项目隔离。

==================================================
四十二、MP4 格式
==========

第一版默认：

```text
container = mp4
video codec = H.264
audio codec = AAC
```

具体 encoder：

优先使用 FFmpeg 当前机器支持的：

```text
libx264
```

如果不可用：

必须明确失败。

不要偷偷使用：

```text
fake codec
```

或者输出其他格式却标记为 MP4。

==================================================
四十三、输出兼容性
=========

默认目标：

```text
H.264 + AAC
MP4
```

便于未来：

```text
Bilibili
YouTube
TikTok
Instagram
```

但本阶段：

绝对不要实现平台发布。

==================================================
四十四、Render Manifest 不允许泄漏敏感信息
=============================

RenderJob manifestSnapshot：

禁止：

```text
apiKey
encryptedApiKey
provider credentials
GenerationTask secrets
```

允许：

```text
assetId
storageKey
safe asset URL
mimeType
duration
timeline timing
```

如果 CompositionManifest 当前已经有安全引用：

继续复用。

不要为了 Render 直接把 Provider Credential 塞进 Manifest。

==================================================
四十五、Provider 系统
===============

Phase 14：

不要新增 AI Provider。

不要调用：

```text
ProviderResolver
```

来生成：

```text
VIDEO
IMAGE
TTS
MUSIC
SFX
```

Render 只是消费已经存在的 Asset。

Timeline：

不产生 AI 成本。

Render：

不产生 AI Provider 成本。

FFmpeg：

属于本地基础设施。

==================================================
四十六、Render API
==============

至少实现：

```http
POST
/projects/:projectId/episodes/:episodeId/render
```

创建 RenderJob。

```http
GET
/projects/:projectId/render-jobs
```

获取项目 Render 历史。

```http
GET
/projects/:projectId/render-jobs/:renderJobId
```

获取 Render Job。

```http
POST
/projects/:projectId/render-jobs/:renderJobId/cancel
```

取消。

```http
POST
/projects/:projectId/render-jobs/:renderJobId/retry
```

Retry。

```http
GET
/projects/:projectId/render-jobs/:renderJobId/artifact
```

获取 Render Artifact 信息。

如果现有 API 命名习惯不同：

以项目已有 REST 风格为准。

==================================================
四十七、Render API 安全
=================

所有 API 必须检查：

```text
projectId
episodeId
renderJobId
timelineId
```

ownership。

禁止：

```text
Project A
读取
Project B RenderJob
```

禁止：

```text
Project A
读取
Project B MP4
```

禁止：

```text
Project A
取消
Project B Render
```

==================================================
四十八、错误码
=======

至少新增：

```text
RENDER_JOB_NOT_FOUND

RENDER_TIMELINE_NOT_FOUND
RENDER_TIMELINE_NOT_LOCKED
RENDER_TIMELINE_STALE

RENDER_JOB_ALREADY_RUNNING
RENDER_JOB_INVALID_STATE

RENDER_MISSING_REQUIRED_ASSET

RENDER_ASSET_NOT_FOUND
RENDER_ASSET_PROJECT_MISMATCH

RENDER_MANIFEST_INVALID

FFMPEG_NOT_FOUND
FFMPEG_EXECUTION_FAILED
FFMPEG_CANCELLED

RENDER_OUTPUT_NOT_FOUND
RENDER_OUTPUT_INVALID

RENDER_ARTIFACT_NOT_FOUND

RENDER_RETRY_NOT_ALLOWED
RENDER_CANCEL_NOT_ALLOWED

RENDER_PROJECT_MISMATCH
RENDER_EPISODE_MISMATCH
```

如果项目已经存在统一 ErrorCode 机制：

继续复用。

不要创建第二套错误体系。

==================================================
四十九、Core Package
================

新增：

```text
packages/core/src/render.ts
```

核心逻辑必须尽可能纯函数。

例如：

```text
validateRenderManifest()
calculateRenderProgress()
calculateEffectiveVolume()
resolveRenderLayers()
resolveRenderInputs()
validateRenderJobTransition()
```

不要把：

```text
Prisma
FFmpeg
filesystem
```

写进 Core。

==================================================
五十、Render State Machine
=======================

必须限制非法状态转换。

允许：

```text
QUEUED
  → PREPARING
  → RENDERING
  → SUCCEEDED
```

失败：

```text
QUEUED
  → FAILED

PREPARING
  → FAILED

RENDERING
  → FAILED
```

取消：

```text
QUEUED
  → CANCELLED

PREPARING
  → CANCEL_REQUESTED
  → CANCELLED

RENDERING
  → CANCEL_REQUESTED
  → CANCELLED
```

Retry：

```text
FAILED
  → new RenderJob
```

不要让：

```text
SUCCEEDED → RENDERING
```

或：

```text
CANCELLED → RENDERING
```

直接发生。

==================================================
五十一、数据库 Migration
=================

新增 Render 相关 Migration。

例如：

```text
20260818xxxxxx_render_engine
```

具体时间戳：

以实际执行时间和项目 migration naming convention 为准。

新增：

```text
RenderJob
RenderJobEvent
RenderArtifact
RenderJobStatus
RenderJobEventType
RenderArtifactType
```

如果实际模型设计需要调整：

可以合理调整。

但是必须：

```text
pnpm exec prisma migrate deploy
pnpm prisma validate
```

禁止：

```text
prisma migrate reset
```

==================================================
五十二、数据库索引
=========

至少考虑：

```text
RenderJob.projectId
RenderJob.episodeId
RenderJob.timelineId
RenderJob.status
RenderJob.createdAt
```

以及：

```text
RenderJobEvent.renderJobId
RenderArtifact.renderJobId
RenderArtifact.projectId
RenderArtifact.episodeId
```

如果 Prisma schema 当前关系已经自然创建索引：

避免重复索引。

==================================================
五十三、Web Render UI
=================

新增：

```text
/projects/:id/episodes/:episodeId/render
```

或者在 Timeline 页面直接提供 Render Panel。

推荐：

```text
Timeline
↓
Lock
↓
Render
```

Render UI 至少显示：

```text
Timeline Version
Render Status
Progress
Current Stage
Created At
Started At
Completed At
Error
Artifact
```

==================================================
五十四、Timeline 页面 Render 操作
=========================

如果 Timeline：

```text
DRAFT
PREVIEW_READY
STALE
```

Render Button：

disabled。

显示：

```text
请先锁定时间线
```

如果：

```text
LOCKED
```

显示：

```text
Render Episode
```

点击后：

```text
POST /render
```

然后进入：

```text
Render Job
```

状态轮询。

==================================================
五十五、Render Progress UI
======================

必须真实反映：

```text
QUEUED
PREPARING
RENDERING
SUCCEEDED
FAILED
CANCELLED
```

如果：

```text
progress = null
```

显示：

```text
Rendering...
```

不要显示假的：

```text
37%
```

==================================================
五十六、Render Artifact UI
======================

Render 成功后：

显示：

```text
Episode MP4 Ready
```

允许：

```text
Preview
Open
Download
```

如果现有 Asset API / Storage API 已经有安全访问机制：

继续复用。

不要直接暴露服务器 filesystem path。

绝对不要向浏览器返回：

```text
C:\...
```

这种内部路径。

==================================================
五十七、Web API Client
==================

所有 Web API：

必须通过：

```text
packages/api-client
```

禁止：

```text
fetch()
axios()
```

散落在页面。

新增：

```text
createRenderJob()
getRenderJobs()
getRenderJob()
cancelRenderJob()
retryRenderJob()
getRenderArtifact()
```

如果已有 client naming convention：

遵循现有风格。

==================================================
五十八、Mobile
==========

Mobile Phase 14 不需要实现完整 Render 控制台。

至少：

```text
查看 Render 状态
查看 Render Progress
查看 Render History
查看成功 Artifact
```

可以：

```text
取消 Render
```

但如果实现复杂：

可以只读。

复杂 Render 管理：

引导 Web。

==================================================
五十九、Desktop
===========

Desktop：

至少：

```text
查看 Render Job
查看 Progress
查看 Artifact
播放最终 MP4
```

基础控制：

```text
Render
Cancel
Retry
```

可以实现。

不要做专业 NLE。

==================================================
六十、Render Preview 与 Final Render 的关系
====================================

必须明确区分：

```text
Composition Preview
```

和：

```text
Final Render
```

UI 文案：

Preview：

```text
这是合成预览，不是最终视频导出。
```

Render：

```text
这是最终 Episode MP4 Render。
```

不要把 Browser Preview 标记成：

```text
Final Video
```

==================================================
六十一、Render 与 Timeline Lock
==========================

最终产品流程：

```text
Timeline
↓
Preview
↓
用户修改
↓
Preview
↓
确认
↓
LOCK
↓
Render
↓
MP4
```

Render 成功后：

不要自动：

```text
unlock
```

Timeline 保持：

```text
LOCKED
```

==================================================
六十二、Render 完成后 Timeline 行为
==========================

Render 成功：

Timeline：

仍然：

```text
LOCKED
```

RenderJob：

```text
SUCCEEDED
```

Artifact：

```text
存在
```

如果用户需要重新编辑：

```text
unlock
```

修改 Timeline。

此时：

```text
Timeline → STALE / DRAFT
```

具体以 Phase 13 已有状态逻辑为准。

新的 Timeline version：

重新 Lock。

重新 Render。

旧 RenderArtifact：

不能自动删除。

==================================================
六十三、Render History
==================

必须保留历史。

例如：

```text
Render #1
Timeline v3
FAILED

Render #2
Timeline v3
SUCCEEDED

Render #3
Timeline v4
SUCCEEDED
```

不能只保存：

```text
latestRender
```

==================================================
六十四、Render Artifact 与旧 Artifact
===============================

不要覆盖旧 MP4。

使用：

```text
renderJobId
```

生成唯一 storageKey。

例如：

```text
projects/<projectId>/episodes/<episodeId>/renders/<renderJobId>/episode.mp4
```

具体路径：

遵循当前 AssetStorageService 命名约定。

绝对不能：

```text
episodes/1/final.mp4
```

让多个 Render Job 覆盖同一个文件。

==================================================
六十五、Render Job Snapshot 验证
==========================

Worker 开始执行前：

必须验证：

```text
manifestSnapshot
```

至少：

```text
episodeId
durationSeconds
fps
resolution
tracks
clips
```

有效。

如果 manifest 无效：

```text
FAILED
```

错误：

```text
RENDER_MANIFEST_INVALID
```

不能启动 FFmpeg。

==================================================
六十六、Asset 验证
============

Worker 在 FFmpeg 前：

必须确认所有输入 Asset：

```text
存在
可读取
属于正确 Project
mimeType 合法
```

如果：

```text
Asset missing
```

Render 失败。

不要调用 AI。

不要生成 replacement。

==================================================
六十七、FFmpeg 输入清单
===============

建议生成内部：

```text
render-inputs.json
```

或者：

```text
inputs.json
```

用于 Worker 调试。

但不要让浏览器直接访问。

其中可以包含：

```text
assetId
localPath
startTime
duration
track
volume
```

不要包含：

```text
apiKey
provider credentials
```

==================================================
六十八、FFmpeg Command Builder
==========================

不要在 Service 中大量拼字符串。

新增：

```text
FFmpegCommandBuilder
```

负责：

```text
Manifest
↓
FFmpeg argv[]
```

必须可测试。

例如：

```text
buildVideoArguments()
buildAudioArguments()
buildFilterComplex()
buildOutputArguments()
```

不要直接执行。

执行由：

```text
FFmpegService
```

负责。

这样：

```text
CommandBuilder
```

可以完全单元测试。

==================================================
六十九、FFmpeg 不允许 shell 注入
=======================

禁止：

```text
exec(commandString)
```

必须：

```text
spawn(ffmpegPath, args)
```

所有路径：

作为独立参数。

==================================================
七十、Filter Complex
=================

第一版可以使用：

```text
-filter_complex
```

实现：

```text
video composition
audio mixing
scale
pad
overlay
trim
setpts
adelay
volume
amix
```

但：

不要为了“看起来高级”实现复杂 filter。

优先：

```text
稳定
可测试
可 Debug
```

==================================================
七十一、Video Layer Composition
===========================

如果 Timeline 存在：

```text
VIDEO Track
IMAGE Track
```

需要将视觉内容按照：

```text
startTime
duration
zIndex
opacity
```

合成。

至少支持：

```text
单层 VIDEO
单层 IMAGE
VIDEO + IMAGE
多视觉 Clip
```

如果实现多层 FFmpeg overlay 非常复杂：

优先保证：

```text
VIDEO + IMAGE fallback
```

以及正常连续 Shot Render。

不要为了 Phase 14 引入第三方 NLE。

==================================================
七十二、Audio Mixing
================

Audio Filter：

至少支持：

```text
Dialogue
Music
SFX
```

每个：

```text
volume
delay
trim
```

最后：

```text
amix
```

输出：

```text
AAC
```

注意：

没有 Audio Track：

也必须允许纯视频 Render。

没有 Music：

允许 Render。

没有 SFX：

允许 Render。

没有 Dialogue：

如果 Timeline 本身没有缺失：

允许 Render。

但如果 Timeline 明确存在：

```text
missingDialogueAudio
```

默认拒绝正式 Render。

==================================================
七十三、纯视频 Episode
===============

必须支持：

```text
VIDEO / IMAGE
```

没有任何 Audio 的情况。

输出：

```text
H.264 MP4
```

不能因为没有音频而失败。

==================================================
七十四、纯图片 Episode
===============

必须支持：

```text
IMAGE Clips
```

即使：

```text
没有 Video Asset
```

也能真实 Render。

使用 FFmpeg：

```text
image → video
```

根据 Timeline duration 生成。

==================================================
七十五、Video + Audio Episode
=========================

必须支持：

```text
Video
+
Dialogue
+
Music
+
SFX
```

这是 Phase 14 最重要的真实 Demo。

最终 MP4：

必须可以播放：

```text
画面
+
对白
+
音乐
+
音效
```

==================================================
七十六、Render Duration
===================

最终输出 duration：

必须以：

```text
Manifest.durationSeconds
```

为准。

不要简单使用：

```text
最长 Asset duration
```

如果最后一个 Clip：

```text
startTime = 20
duration = 5
```

Timeline：

```text
duration = 25
```

最终视频：

必须接近：

```text
25 seconds
```

允许 FFmpeg container / encoding 存在极小误差。

==================================================
七十七、Render 输出校验
===============

Render 完成后：

使用：

```text
ffprobe
```

或者等价方式检查：

```text
duration
width
height
fps
video codec
audio codec
file size
```

不要仅仅：

```text
fs.existsSync()
```

就认为 Render 成功。

==================================================
七十八、FFprobe
===========

如果系统存在：

```text
ffprobe
```

优先使用。

如果 FFmpeg distribution 已经包含：

```text
ffprobe
```

自动定位。

如果没有：

明确：

```text
FFPROBE_NOT_FOUND
```

不要假装验证成功。

==================================================
七十九、Render Artifact Metadata
============================

Render 成功后：

Artifact 必须记录：

```text
mimeType = video/mp4
fileSize
durationSeconds
width
height
fps
```

如果当前模型不需要全部字段：

至少保留未来可以查询这些信息的能力。

==================================================
八十、Render API Response
======================

创建 Job：

返回：

```json
{
  "id": "...",
  "status": "QUEUED",
  "timelineVersion": 3,
  "progress": 0,
  "currentStage": "QUEUED"
}
```

不要返回：

```text
apiKey
encryptedApiKey
```

不要返回：

```text
GenerationTask
```

不要返回：

```text
FFmpeg internal command
```

==================================================
八十一、Render Job 查询
=================

GET：

必须返回：

```text
id
projectId
episodeId
timelineId
timelineVersion
status
progress
currentStage
attempt
errorCode
errorMessage
createdAt
startedAt
completedAt
cancelledAt
artifact
```

manifestSnapshot：

如果 API 没必要：

不要默认返回完整 Snapshot。

如果需要：

提供受控 endpoint。

避免 response 过大。

==================================================
八十二、不要泄漏内部路径
============

API 禁止返回：

```text
C:\Users\...
C:\project\...
/tmp/render/...
```

Storage path：

只能通过安全 Asset / Artifact URL。

==================================================
八十三、Artifact URL
================

Render Artifact：

必须通过已有项目安全 Storage API。

如果需要：

新增：

```http
GET
/projects/:projectId/render-artifacts/:artifactId
```

或者返回项目已有：

```text
safe URL
```

不要：

```text
sendFile(rawAbsolutePath)
```

绕过项目权限。

==================================================
八十四、测试要求
========

必须新增完整测试。

至少覆盖：

### Render Domain

```text
1. RenderJob create
2. RenderJob get
3. RenderJob history
4. RenderJob status transition
5. Invalid status transition
```

### Timeline Lock

```text
6. DRAFT cannot render
7. PREVIEW_READY cannot render
8. STALE cannot render
9. LOCKED can render
10. Render does not auto-lock
```

### Snapshot

```text
11. Manifest snapshot created
12. Snapshot immutable
13. Timeline changes do not change snapshot
14. Retry reuses snapshot
15. Render uses timelineVersion
```

### Project Isolation

```text
16. Cross-project RenderJob rejected
17. Cross-project Timeline rejected
18. Cross-project Asset rejected
19. Cross-project Artifact rejected
```

### FFmpeg

```text
20. FFmpeg not found
21. FFmpeg command builder
22. FFmpeg args do not use shell interpolation
23. FFmpeg success
24. FFmpeg failure
25. FFmpeg cancellation
```

### Progress

```text
26. Progress parsing
27. Progress monotonicity
28. Invalid progress handling
29. Current stage
```

### Artifact

```text
30. Output exists
31. Empty output rejected
32. Invalid MP4 rejected
33. Artifact created
34. Artifact metadata
35. Old artifact preserved
```

### Audio

```text
36. Dialogue volume
37. Music volume
38. SFX volume
39. Track muted
40. Clip disabled
41. Track × Clip volume
42. Missing music allowed
43. Missing SFX allowed
```

### Video

```text
44. Video render
45. Image render
46. Video priority
47. Image fallback
48. Multiple clips
49. Duration
50. FPS
51. Resolution
52. Aspect ratio
```

### Failure

```text
53. Missing required asset
54. Manifest invalid
55. Retry failed job
56. Retry preserves snapshot
57. Cancel queued
58. Cancel running
59. Cancel actually terminates worker
60. Failed job contains safe error
```

### Security

```text
61. API key never exposed
62. encrypted API key never exposed
63. GenerationTask never exposed
64. filesystem path never exposed
65. cross-project artifact rejected
```

### API

```text
66. Create Render API
67. Get Render API
68. Cancel Render API
69. Retry Render API
70. Artifact API
```

### Web

```text
71. Render button disabled when not LOCKED
72. Render button enabled when LOCKED
73. Progress UI
74. Failure UI
75. Artifact UI
```

测试数量不要求固定。

但必须保证新增测试足够覆盖真正 Render Engine。

==================================================
八十五、Integration Test
====================

必须尽可能增加一个真实 Integration Render Test。

如果当前测试环境允许：

准备最小：

```text
1 image
1 short audio
```

或者：

```text
1 short video
1 short audio
```

真实调用：

```text
FFmpeg
```

生成：

```text
.mp4
```

然后验证：

```text
file exists
file size > 0
ffprobe succeeds
duration > 0
video stream exists
```

如果测试环境没有 FFmpeg：

不要假装通过。

应该：

```text
明确标记环境依赖
```

但项目本身必须提供：

```text
Render Engine integration path
```

以及：

```text
FFmpeg availability check
```

==================================================
八十六、Seed
========

新增：

```text
prisma/seed-render-demo.ts
```

命令：

```text
pnpm db:seed:render-demo
```

不要自动执行。

Seed 必须依赖：

```text
星河碰撞
Season 1
Episode 1
Script
Storyboard
Image
Video
TTS
Music
SFX
Timeline
```

如果不存在：

明确提示：

```text
请先执行 Phase 13 上游 seed。
```

不要 reset。

不要创建重复上游数据。

==================================================
八十七、Render Demo Seed
====================

如果依赖完整：

Seed 应确保：

```text
Timeline exists
Timeline is LOCKED
```

或者：

创建 Demo Render Job 前：

明确执行：

```text
Timeline Build
Preview
Lock
```

但不要自动生成 AI Asset。

Demo 必须真实引用：

```text
existing Asset
```

然后允许：

```text
Render
```

最终可以验证：

```text
真实 MP4
```

==================================================
八十八、Render Demo 不得伪造
====================

绝对禁止：

```text
seed 一个假的 MP4 URL
```

禁止：

```text
创建 fake RenderArtifact
```

禁止：

```text
status = SUCCEEDED
但文件不存在
```

Demo 只能在真实 Render 完成后：

```text
SUCCEEDED
```

==================================================
八十九、CLI / Developer Experience
==============================

建议提供：

```text
pnpm render:check
```

检查：

```text
FFmpeg
FFprobe
```

以及：

```text
pnpm render:episode --project <id> --episode <id>
```

如果当前项目已有 CLI 结构：

遵循现有方式。

CLI 不是强制要求。

但是必须让开发者能够方便验证：

```text
Timeline LOCKED
→ Render
→ MP4
```

==================================================
九十、环境变量
=======

如果需要：

支持：

```text
FFMPEG_PATH
FFPROBE_PATH
```

不要硬编码：

```text
C:\ffmpeg\bin\ffmpeg.exe
```

Linux / macOS / Windows 都应该可以配置。

默认：

尝试：

```text
ffmpeg
ffprobe
```

PATH。

==================================================
九十一、Windows 兼容
==============

开发环境主要是 Windows。

必须特别注意：

```text
path separator
spawn
process termination
temporary directory
file locking
EPERM
```

不要使用：

```text
bash-only
```

脚本。

不要假设：

```text
/bin/sh
```

存在。

==================================================
九十二、API 运行纪律
============

如果：

```text
3011
```

已经被旧 API 进程占用：

不要杀进程。

不要：

```text
taskkill /F
```

不要暴力关闭 Node。

先检查：

```http
GET http://localhost:3011/health
```

如果健康：

说明需要用户手动重启 API 才能加载新 Render 路由。

不要破坏现有服务。

==================================================
九十三、Prisma Windows EPERM
========================

如果：

```text
prisma generate
```

因为运行中的 API 占用：

```text
query_engine.dll
```

导致：

```text
EPERM
```

不要：

```text
删除 DLL
杀 Node
强制结束 API
```

遵循项目现有纪律。

如果必须重新加载：

提示用户手动重启 API。

==================================================
九十四、Build 顺序
============

完成后必须运行：

```text
pnpm prisma validate
pnpm lint
pnpm typecheck
pnpm test
pnpm build
```

如果需要：

```text
pnpm exec prisma migrate deploy
```

也必须执行。

==================================================
九十五、数据库数据完整性
============

完成后检查：

```text
Project
World
Character
Season
Episode
Provider
GenerationTask
Script
Storyboard
Asset
```

不得因为 Phase 14：

```text
变成 0
```

不得删除 Phase 13 数据。

不得删除旧 GenerationTask。

不得删除旧 Asset。

不得删除旧 Timeline。

==================================================
九十六、现有功能回归
==========

必须确认：

```text
World
Character
Story Bible
Season
Episode
Script
Storyboard
Image
Video
TTS
Music
SFX
Provider
Project AI Config
Generation History
Timeline
Composition Preview
```

仍然正常。

尤其继续保证：

```text
IMAGE ≠ TEXT fallback
VIDEO ≠ IMAGE fallback
TTS ≠ CHAT fallback
MUSIC ≠ TTS fallback
SFX ≠ TTS fallback
```

Render 不改变 Provider Capability Resolution。

==================================================
九十七、Render 与 AI 成本
==================

必须明确：

```text
Render Engine
```

不产生 AI Provider 成本。

不调用：

```text
IMAGE
VIDEO
TTS
MUSIC
SFX
```

Provider。

如果 Asset 不存在：

Render 失败。

不是：

```text
AI 自动生成
```

==================================================
九十八、不要引入“假功能”
=============

严格禁止：

```text
假 Render
假 MP4
假 Worker
假 Progress
假 Cancellation
假 Retry
假 Artifact
假 FFmpeg
假 Cloud Render
假 Provider
假 Storage
```

如果环境没有 FFmpeg：

显示：

```text
FFMPEG_NOT_FOUND
```

而不是：

```text
Render succeeded
```

==================================================
九十九、未来 Render 扩展点
=================

Phase 14 必须给未来留下：

```text
Local Worker
↓
未来可以替换成
↓
Redis/BullMQ Worker
↓
Cloud Worker
↓
GPU Render
```

因此：

不要让业务代码直接依赖：

```text
child_process
```

应该：

```text
RenderWorker
    ↓
RenderEngine
    ↓
FFmpegAdapter
```

未来可以替换：

```text
FFmpegAdapter
```

或者：

```text
LocalRenderWorker
CloudRenderWorker
```

但本阶段只实现：

```text
LocalRenderWorker
FFmpegAdapter
```

==================================================
一百、Render Engine 抽象
===================

建议定义：

```ts
interface RenderEngine {
  render(input: RenderInput): Promise<RenderResult>
  cancel(renderId: string): Promise<void>
}
```

或者根据当前项目实际架构定义。

不要把接口设计得过度复杂。

第一版目标：

```text
Local FFmpeg
```

但未来可以：

```text
FFmpeg
Remotion
WebCodecs
Cloud
GPU
```

==================================================
一百零一、Render Input
=================

Render Engine 输入应该接近：

```text
RenderManifestSnapshot
```

而不是：

```text
Prisma Timeline
```

建议：

```text
RenderInput
├── manifest
├── workspace
├── output
└── options
```

Render Engine 不知道：

```text
Project
Episode
Script
Storyboard
Provider
GenerationTask
```

它只知道：

```text
Manifest
```

==================================================
一百零二、Render Result
==================

Render Engine 返回：

```text
RenderResult
```

至少：

```text
success
outputPath
durationSeconds
width
height
fps
fileSize
videoCodec
audioCodec
```

失败：

```text
error
```

不要直接返回：

```text
Prisma RenderJob
```

保持 Domain 与 Infrastructure 分离。

==================================================
一百零三、Render Worker 与 Database
=============================

Worker 可以读取：

```text
RenderJob
```

但：

FFmpeg Engine 不应该直接访问 Prisma。

正确：

```text
RenderWorkerService
↓
RenderJobRepository
↓
RenderEngine
```

或者使用项目当前 repository pattern。

==================================================
一百零四、并发
=======

Phase 14：

默认：

```text
concurrency = 1
```

即：

一次只运行一个本地 Render Job。

原因：

避免：

```text
CPU 爆满
RAM 爆满
FFmpeg 冲突
Windows 文件锁
```

未来再做：

```text
worker concurrency
```

不要现在实现复杂调度器。

==================================================
一百零五、Worker Crash
=================

如果 Worker：

```text
FFmpeg process crash
```

必须：

```text
RenderJob = FAILED
```

不能：

```text
一直 RENDERING
```

如果发现：

```text
stale RENDERING job
```

本阶段可以提供：

```text
startup recovery
```

例如：

应用启动时：

```text
RENDERING
```

如果没有对应活跃 process：

可以标记：

```text
FAILED
```

但必须谨慎。

不要误杀正在运行的 Render。

如果无法可靠判断：

不要自动修改。

==================================================
一百零六、Render Startup Recovery
============================

如果实现：

只恢复：

```text
QUEUED
```

Job。

不要自动重新执行：

```text
RENDERING
```

Job。

否则可能产生：

```text
重复 FFmpeg
```

==================================================
一百零七、Render Job Queue
=====================

虽然本阶段不使用 Redis：

但可以在数据库中：

```text
RenderJob.status = QUEUED
```

作为最简单的本地任务队列。

Worker：

```text
SELECT next QUEUED job
```

然后：

```text
atomic claim
```

必须避免同一 Job 被两个 worker 同时执行。

即使当前 concurrency = 1：

也设计好状态抢占。

==================================================
一百零八、Render Claim
=================

Claim Job 时：

必须使用事务或者条件更新：

```text
QUEUED → PREPARING
```

只有成功更新的 Worker：

才能执行。

避免：

```text
Worker A
Worker B
```

同时执行同一个 Job。

==================================================
一百零九、Render Job 事务
==================

创建 RenderJob：

必须保证：

```text
Manifest Snapshot
+
RenderJob
```

一致创建。

如果其中一个失败：

不能留下半成品。

优先使用：

```text
prisma.$transaction
```

但：

FFmpeg 本身不能放进数据库 transaction。

正确：

```text
DB transaction
↓
RenderJob QUEUED
↓
commit
↓
Worker
↓
FFmpeg
```

==================================================
一百一十、Render Artifact Transaction
================================

FFmpeg 成功：

先确认：

```text
output valid
```

再：

```text
create RenderArtifact
update RenderJob = SUCCEEDED
```

最好在一个 DB transaction 中完成：

```text
RenderArtifact
+
RenderJob
```

确保不会出现：

```text
Artifact created
Job FAILED
```

或者：

```text
Job SUCCEEDED
Artifact missing
```

==================================================
一百一十一、Render API Key 安全回归
=========================

必须测试：

```text
GET RenderJob
GET RenderHistory
GET RenderArtifact
```

任何 response：

都不能包含：

```text
apiKey
encryptedApiKey
provider secret
```

Render Manifest Snapshot：

也必须安全。

==================================================
一百一十二、GenerationTask 安全回归
=========================

RenderJob：

不要关联并返回：

```text
GenerationTask
```

除非未来确实需要。

Phase 14：

Render 不是 GenerationTask。

不要为了统一：

创建假的 GenerationTask。

==================================================
一百一十三、Render 与 Asset 模型边界
=========================

必须保持：

```text
Asset
=
AI / Source Media
```

```text
RenderArtifact
=
Final Render Output
```

不要把 Render MP4：

强制伪装成：

```text
AI Generated Asset
```

除非现有项目模型已经明确支持这种关系。

==================================================
一百一十四、Render 版本关系
=================

一个 Timeline：

```text
version 5
```

可以有：

```text
RenderJob #1
RenderJob #2
RenderJob #3
```

全部都是：

```text
timelineVersion = 5
```

如果 Timeline version 变成：

```text
6
```

则新的 Render：

```text
timelineVersion = 6
```

旧 Render：

仍然保留。

==================================================
一百一十五、Render UI History
=======================

Episode Render 页面显示：

```text
Render History
```

例如：

```text
Version 5
Failed
2026-08-18 15:02

Version 5
Succeeded
2026-08-18 15:08

Version 6
Succeeded
2026-08-18 16:20
```

用户可以查看：

```text
status
timeline version
duration
created time
artifact
error
```

==================================================
一百一十六、最终 Demo
=============

Phase 14 最终必须至少完成一次真实 Demo：

```text
Episode 1
↓
Timeline
↓
LOCKED
↓
Create RenderJob
↓
Local Worker
↓
FFmpeg
↓
MP4
↓
RenderArtifact
↓
Web 播放
```

如果数据库当前没有：

```text
Script
Storyboard
Asset
```

则：

不要假装 Demo 完成。

明确说明：

```text
上游 Demo 数据不存在，需要先执行 Phase 13 seed。
```

==================================================
一百一十七、最终验收标准
============

只有满足以下条件，才宣布：

# Phase 14 COMPLETE

必须：

```text
[ ] RenderJob 完成
[ ] RenderJobEvent 完成
[ ] RenderArtifact 完成
[ ] Render Status 完成
[ ] Render State Machine 完成
[ ] Manifest Snapshot 完成
[ ] Timeline LOCKED 校验完成
[ ] Timeline STALE 校验完成
[ ] Local Render Worker 完成
[ ] FFmpeg Adapter 完成
[ ] FFmpeg Command Builder 完成
[ ] Video Render 完成
[ ] Image Render 完成
[ ] Audio Render 完成
[ ] Dialogue 完成
[ ] Music 完成
[ ] SFX 完成
[ ] Track Volume 完成
[ ] Clip Volume 完成
[ ] Track Muted 完成
[ ] Clip Disabled 完成
[ ] Render Progress 完成
[ ] Render Cancellation 完成
[ ] Render Retry 完成
[ ] Render Failure 完成
[ ] Render Artifact 完成
[ ] MP4 输出真实存在
[ ] FFprobe 校验完成
[ ] Project isolation 完成
[ ] Asset isolation 完成
[ ] API Key 无泄漏
[ ] GenerationTask 无泄漏
[ ] 内部 filesystem path 无泄漏
[ ] Web Render UI 完成
[ ] Render History 完成
[ ] Mobile 基础查看完成
[ ] Desktop 基础查看完成
[ ] Seed 完成
[ ] Tests 全部通过
[ ] lint 通过
[ ] typecheck 通过
[ ] prisma validate 通过
[ ] build 通过
[ ] API health 正常
[ ] 旧数据保留
[ ] 旧功能无回归
```

并且：

```text
[ ] 没有引入 Redis
[ ] 没有引入 BullMQ
[ ] 没有实现 Cloud Render
[ ] 没有实现 GPU Render
[ ] 没有新增 AI Provider
[ ] 没有新增 Render Provider
[ ] 没有实现自动补素材
[ ] 没有实现 AI 自动剪辑
[ ] 没有实现字幕
[ ] 没有实现高级 Audio Mastering
[ ] 没有实现平台发布
[ ] 没有实现 Billing
[ ] 没有实现 Login/JWT
```

最重要：

```text
[ ] 不是假 MP4
[ ] 不是假 Render
[ ] 不是假 Progress
[ ] 不是假 Worker
[ ] 不是假 Artifact
```

==================================================
一百一十八、最终输出报告
============

完成后严格输出：

# Phase 14 Complete

## 1. Database

新增 Model：

...

Migration：

...

## 2. Render Architecture

...

## 3. RenderJob

...

## 4. Manifest Snapshot

...

## 5. Render Worker

...

## 6. FFmpeg

...

## 7. Video Composition

...

## 8. Audio Composition

...

## 9. Progress

...

## 10. Cancellation

...

## 11. Retry

...

## 12. Artifact

...

## 13. API

...

## 14. Web

...

## 15. Mobile

...

## 16. Desktop

...

## 17. Seed

...

## 18. Tests

...

## 19. lint

...

## 20. typecheck

...

## 21. prisma validate

...

## 22. build

...

## 23. API health

...

## 24. Existing Data

...

## 25. Existing Provider

...

## 26. Existing GenerationTask

...

## 27. Real Render Verification

必须明确：

```text
FFmpeg:
FOUND / NOT_FOUND

FFprobe:
FOUND / NOT_FOUND

RenderJob:
...

Timeline Version:
...

Output:
...

MP4:
FOUND / NOT_FOUND

File Size:
...

Duration:
...

Resolution:
...

FPS:
...

Video Codec:
...

Audio Codec:
...
```

如果没有真实 Render：

必须明确写：

```text
REAL RENDER NOT VERIFIED
```

绝对不能写：

```text
Render succeeded
```

## 28. Remaining Limitations

...

## 29. Explicitly NOT Implemented

...

## 30. Recommended Next Phase

...

==================================================
一百一十九、Phase 14 最终边界
===================

本阶段最终做到：

```text
Storyboard / Script / Assets
        ↓
Episode Timeline
        ↓
Composition Manifest
        ↓
LOCKED Timeline
        ↓
RenderJob
        ↓
Manifest Snapshot
        ↓
Local Render Worker
        ↓
FFmpeg
        ↓
Real Episode MP4
        ↓
RenderArtifact
```

停止。

不要继续实现：

```text
Subtitle Engine
Advanced Audio Mastering
AI Intelligent Editing
Batch Production
Cloud Render
Redis
BullMQ
Publishing
Billing
```

留到后续 Phase。

==================================================
一百二十、未来路线
=========

Phase 14：

```text
Local Render Engine
```

Phase 15：

```text
Subtitle / Caption Engine
```

Phase 16：

```text
Advanced Audio Mixing / Mastering
```

Phase 17：

```text
Batch Generation / Production Pipeline
```

Phase 18：

```text
AI Intelligent Editing / Agent
```

Phase 19：

```text
User Account / BYOK / Billing
```

Phase 20：

```text
Publishing
YouTube
TikTok
Bilibili
Instagram
```

未来最终链路：

```text
Story Outline
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
Images
↓
Videos
↓
Dialogue
↓
Music
↓
SFX
↓
Timeline
↓
Preview
↓
LOCK
↓
Render
↓
MP4
↓
Subtitles
↓
Final Episode
↓
Publish
```

==================================================
一百二十一、执行纪律
==========

你现在处于 Cursor 自动执行模式。

因此：

不要频繁询问我是否继续。

不要因为一个普通实现问题暂停整个 Phase。

遇到普通：

```text
TypeScript error
Prisma error
Test failure
Lint error
Build error
API integration issue
```

自行检查、定位并修复。

遇到数据库问题：

优先使用：

```text
新的增量 migration
```

绝对不要：

```text
reset
DROP
删除历史 migration
```

遇到 API 端口占用：

不要杀进程。

检查：

```text
/health
```

必要时：

提示用户手动重启 API。

遇到 Windows Prisma DLL EPERM：

不要删除占用文件。

不要暴力杀 API。

按照当前项目已有纪律处理。

==================================================
一百二十二、重要实现原则
============

不要假设当前代码结构。

首先检查实际项目：

```text
apps/
packages/
prisma/
api-client/
core/
storage/
timeline/
```

以及 Phase 13 实际已经创建的：

```text
EpisodeTimeline
TimelineTrack
TimelineClip
TimelineService
TimelineBuilderService
TimelineTimingService
TimelineContinuityService
CompositionService
CompositionManifest
AssetStorageService
```

不要重复创建已有模块。

不要重新设计 Phase 13。

优先复用：

```text
Timeline
Composition
Asset
Storage
Project isolation
API client
Core types
```

==================================================
一百二十三、最终要求
==========

现在开始：

# Phase 14 — Local Render Engine / FFmpeg Episode MP4 v1

请直接检查当前项目实际代码状态。

不要假设文件存在。

不要重建已有模块。

不要删除已有功能。

不要 reset 数据库。

不要修改历史 migration。

不要引入 Redis。

不要引入 BullMQ。

不要引入 AI Provider。

不要实现 Cloud Render。

不要实现发布。

不要实现 Billing。

不要创建假 MP4。

不要创建假 Render。

不要创建假 Worker。

不要创建假 Progress。

必须真正使用 FFmpeg 完成：

```text
LOCKED Timeline
↓
RenderJob
↓
Manifest Snapshot
↓
Local Worker
↓
FFmpeg
↓
真实 MP4
↓
RenderArtifact
```

完成全部：

```text
implementation
+
migration
+
tests
+
lint
+
typecheck
+
prisma validate
+
build
+
API health
+
real render verification
```

之后再输出完整：

# Phase 14 Complete

报告。

现在开始执行。
