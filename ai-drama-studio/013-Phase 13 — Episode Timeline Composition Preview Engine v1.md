你现在开始执行：

# Phase 13 — Episode Timeline / Composition Preview Engine v1

这是 AI Drama Studio 的第 13 阶段。

==================================================
一、最高优先级目标
==================================================

本阶段目标：

把前面已经完成的：

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

第一次统一进入：

Episode Timeline / Composition

形成：

Episode
  ↓
Timeline
  ├── Visual Tracks
  │   ├── Shot Video
  │   └── Shot Image
  │
  ├── Dialogue Track
  │   └── ScriptBlock TTS
  │
  ├── Music Track
  │   └── Episode Music
  │
  └── SFX Track
      └── Episode / Scene / Shot SFX

最终得到：

Episode Timeline
      ↓
Composition Preview
      ↓
浏览器内可播放的“整集预览”

注意：

本阶段只实现：

Timeline 数据模型
+
Composition Engine
+
Preview

绝对不要实现：

FFmpeg
视频导出
最终 MP4
字幕烧录
BGM 混音导出
平台发布
YouTube
TikTok
Bilibili
自动发布
Billing
Login
JWT
Subscription
Redis
BullMQ
Worker

这些全部留到后续阶段。

==================================================
二、必须严格遵守现有架构
==================================================

绝对不要重建 Monorepo。

不要换技术栈。

继续使用：

pnpm
Turborepo
Nuxt 3
Vue 3
NestJS
Ionic Vue
Tauri
Prisma
PostgreSQL

不要删除历史 migration。

不要修改历史 migration。

只允许新增 migration。

绝对禁止：

prisma migrate reset
DROP TABLE
删除现有数据
重新初始化数据库

必须保证：

pnpm lint
pnpm typecheck
pnpm test
pnpm prisma validate
pnpm build

最终全部通过。

==================================================
三、核心架构原则
==================================================

本阶段必须明确：

Timeline 是“编排层”。

它不是：

Script
Storyboard
Asset

的替代品。

正确关系：

Script
  ↓
Storyboard
  ↓
Assets
  ↓
Timeline
  ↓
Composition Preview
  ↓
未来 Render
  ↓
未来 Publish

Timeline 只引用已有实体。

不要把：

imagePrompt
videoPrompt
dialogueText
musicPrompt
sfxPrompt

重新复制进 Timeline。

Timeline 应该主要存：

source reference
+
时间位置
+
持续时间
+
轨道
+
层级
+
播放参数

==================================================
四、Timeline 数据模型
==================================================

新增：

EpisodeTimeline

关系：

Episode 1 ── 1 EpisodeTimeline

建议字段：

EpisodeTimeline

- id
- projectId
- episodeId
- version
- status
- durationSeconds
- fps
- resolution
- aspectRatio
- metadata
- createdAt
- updatedAt

Timeline Status：

DRAFT
PREVIEW_READY
STALE
LOCKED

注意：

本阶段不要实现真正 Render Status。

不要加入：

RENDERING
RENDERED
EXPORTING
EXPORTED

这些属于后续 Render Engine。

==================================================
五、Timeline Track
==================================================

新增：

TimelineTrack

关系：

EpisodeTimeline 1 ── N TimelineTrack

字段：

- id
- timelineId
- type
- name
- order
- enabled
- muted
- volume
- metadata
- createdAt
- updatedAt

Track Type：

VIDEO
IMAGE
DIALOGUE
MUSIC
SFX

注意：

本阶段允许：

VIDEO Track
IMAGE Track

但默认 Composition 可以优先使用 VIDEO。

如果某个 Shot 没有 Final Video：

可以 fallback：

FINAL IMAGE

但是：

不能 fallback 到：

Script
Character
Storyboard Prompt

视觉 Timeline 的来源只能是：

StoryboardShotAsset
+
Asset

==================================================
六、Timeline Clip
==================================================

新增：

TimelineClip

关系：

TimelineTrack 1 ── N TimelineClip

字段：

- id
- trackId
- type
- sourceType
- sourceId
- assetId
- startTime
- duration
- sourceStartTime
- sourceDuration
- zIndex
- volume
- speed
- opacity
- enabled
- metadata
- createdAt
- updatedAt

Clip Type：

VIDEO
IMAGE
AUDIO

SourceType：

STORYBOARD_SHOT
SCRIPT_BLOCK
EPISODE_AUDIO
ASSET

重要：

不要把 sourceId 当成唯一来源。

必须同时保留：

sourceType
+
sourceId

例如：

STORYBOARD_SHOT
shotId

SCRIPT_BLOCK
blockId

EPISODE_AUDIO
episodeAudioAssetId

ASSET
assetId

==================================================
七、不要重复 Asset 数据
==================================================

TimelineClip 不允许复制：

文件 URL
mimeType
storageKey
width
height
durationSeconds
provider
model

这些全部继续从：

Asset

读取。

Timeline 只负责：

“什么时候播放什么”。

==================================================
八、视觉 Timeline 自动构建规则
==================================================

新增：

TimelineBuilderService

核心方法：

buildEpisodeTimeline(projectId, episodeId)

构建顺序：

Episode
 ↓
Storyboard
 ↓
StoryboardShot
 ↓
StoryboardShotAsset
 ↓
Final Video / Final Image
 ↓
Timeline VIDEO / IMAGE Track

规则：

1. 按 StoryboardShot 顺序排列。

2. Shot 的时间来自：

StoryboardShot.durationSeconds

3. 如果存在：

FINAL Video

优先使用 Video。

4. 如果不存在 FINAL Video：

使用 FINAL Image。

5. 如果两者都不存在：

该 Shot 不生成 Visual Clip。

但必须记录：

missingVisualAsset

6. 不允许 AI 自动生成缺失视频。

7. 不允许 AI 自动生成缺失图片。

Timeline Builder 只是编排。

==================================================
九、Dialogue Timeline
==================================================

Dialogue 来源：

Script
→ Scene
→ ScriptBlock
→ ScriptBlockAsset
→ Asset

只处理：

ScriptBlock.type === DIALOGUE

如果 ScriptBlock 有：

FINAL AUDIO

则创建：

DIALOGUE TimelineClip

位置必须根据：

Timeline / Scene / Shot

推导。

但是不要假设每个 Dialogue 天然有准确时间。

因此需要建立：

Dialogue Timing Strategy

v1：

优先使用：

ScriptBlockAsset Asset.durationSeconds

如果不存在 duration：

根据音频实际 duration。

如果没有 Audio：

不要自动调用 TTS。

记录：

missingDialogueAudio

==================================================
十、Music Timeline
==================================================

Music 来源：

EpisodeAudioAsset

role = MUSIC

选择：

isPrimary === true

如果存在：

加入 MUSIC Track。

如果没有：

不创建 Music Clip。

不要自动生成 Music。

Music 默认：

startTime = 0

duration：

使用 Asset.durationSeconds

如果 Music 长于 Episode：

clip duration 截断到 Timeline duration。

如果 Music 短于 Episode：

不要自动循环。

本阶段不实现 Loop。

==================================================
十一、SFX Timeline
==================================================

SFX 来源：

EpisodeAudioAsset

role = SFX

根据：

sceneId
shotId

metadata

或者现有关系确定位置。

如果有：

shotId

则放在对应 Shot 时间范围内。

如果有：

sceneId

则放在对应 Scene 时间范围内。

如果两者都没有：

允许作为 Episode-level SFX。

默认：

Episode-level SFX 从 0 秒开始。

本阶段不实现复杂自动音效编排。

==================================================
十二、时间轴时间计算
==================================================

新增：

TimelineTimingService

负责：

Shot → Timeline Time
Scene → Timeline Time
ScriptBlock → Timeline Time
Audio → Timeline Time

Shot 时间：

shot 1:
0 → duration

shot 2:
shot1.end → shot1.end + duration

依次累加。

Scene 时间：

Scene 的开始时间 = 第一个 Shot start

Scene 的结束时间 = 最后一个 Shot end

Dialogue：

优先尝试匹配 ScriptBlock → StoryboardShot。

如果 ScriptBlock 对应多个 Shot：

使用 sourceScriptBlockIds。

Dialogue 放置到：

第一个相关 Shot 的开始时间。

如果音频超过 Shot：

允许跨越 Shot。

不要强制截断 Dialogue。

==================================================
十三、Timeline Version
==================================================

Timeline 必须记录：

sourceStoryboardVersion
sourceScriptVersion
sourceAssetVersionSummary

用于判断：

Timeline 是否过期。

例如：

Storyboard version = 3
Timeline sourceStoryboardVersion = 2

则：

STALE

Script version 改变：

Timeline 也应该认为可能过期。

但是：

GET 时只计算状态。

不要在 GET 时写数据库。

==================================================
十四、Timeline Stale
==================================================

新增：

TimelineContinuityService

负责：

validateTimelineContinuity()

检查：

Episode 是否属于 Project

Timeline 是否属于 Episode

Storyboard 是否属于 Episode

Shot 是否属于 Storyboard

Scene 是否属于 Script

ScriptBlock 是否属于 Scene

Asset 是否属于 Project

StoryboardShotAsset 是否属于 Shot

ScriptBlockAsset 是否属于 ScriptBlock

EpisodeAudioAsset 是否属于 Episode

任何跨 Project 引用：

必须拒绝。

==================================================
十五、Timeline Build Preview
==================================================

新增 API：

POST

/projects/:projectId/episodes/:episodeId/timeline/build

作用：

根据当前：

Storyboard
Script
Assets
Episode Audio

构建 Timeline。

但是：

本 API 不应该调用 AI。

Timeline 是确定性计算。

不需要：

ProviderResolver
AI
GenerationTask

==================================================
十六、Build 行为
==================================================

build API：

1. 查询 Episode

2. 查询 Storyboard

3. 查询 Script

4. 查询 Shots

5. 查询 Final Visual Assets

6. 查询 Dialogue Assets

7. 查询 Music

8. 查询 SFX

9. 计算时间

10. 创建或更新 Timeline

11. 创建 Tracks

12. 创建 Clips

13. 保存：

sourceStoryboardVersion
sourceScriptVersion

14. 返回 Timeline

必须使用：

prisma.$transaction

==================================================
十七、Build 是否覆盖已有 Timeline
==================================================

如果 Timeline 不存在：

创建。

如果 Timeline 已存在：

必须返回：

TIMELINE_ALREADY_EXISTS

除非请求显式：

rebuild=true

只有：

rebuild=true

才允许重新构建。

重新构建：

version + 1

删除旧 Tracks / Clips

重新创建。

但是：

绝对不要删除：

Asset
Script
Storyboard
Shot

Timeline 只是引用层。

==================================================
十八、手工编辑 Timeline
==================================================

提供基础 CRUD。

Timeline Track：

GET
POST
PATCH
DELETE

Timeline Clip：

GET
POST
PATCH
DELETE

Clip 支持修改：

startTime
duration
sourceStartTime
sourceDuration
volume
speed
opacity
enabled
zIndex

但是：

不能修改：

sourceType
sourceId
assetId

如果需要换素材：

删除旧 Clip
+
创建新 Clip

这样避免隐藏数据变更。

==================================================
十九、禁止 Timeline 直接修改上游
==================================================

Timeline 编辑：

不能修改：

Script
Storyboard
Shot
Character
World
Asset

例如：

用户把 Clip 拖到 10 秒。

只能改变：

TimelineClip.startTime

不能修改：

StoryboardShot.durationSeconds

==================================================
二十、Composition Engine
==================================================

新增：

CompositionEngine

核心：

composeEpisode(projectId, episodeId)

输入：

EpisodeTimeline

输出：

CompositionManifest

不要输出 MP4。

不要使用 FFmpeg。

不要写文件。

CompositionManifest 应类似：

{
  episodeId,
  durationSeconds,
  fps,
  resolution,
  aspectRatio,
  tracks: [
    {
      type,
      clips: [...]
    }
  ]
}

==================================================
二十一、Composition Manifest
==================================================

必须是纯 JSON。

可以作为未来：

FFmpeg
Remotion
WebCodecs
Cloud Render
GPU Render

的输入。

本阶段不绑定具体 Render 技术。

==================================================
二十二、Preview API
==================================================

新增：

GET

/projects/:projectId/episodes/:episodeId/timeline

GET

/projects/:projectId/episodes/:episodeId/timeline/manifest

GET

/projects/:projectId/episodes/:episodeId/timeline/preview

其中：

timeline：

返回数据库 Timeline。

manifest：

返回 CompositionManifest。

preview：

返回用于 Web Preview 的结构化数据。

不要生成 MP4。

==================================================
二十三、Web Timeline Editor
==================================================

新增：

/projects/:id/episodes/:episodeId/timeline

页面结构：

┌─────────────────────────────────────┐
│ Episode Header                      │
├─────────────────────────────────────┤
│ Preview Player                      │
│                                     │
│ ▶ 00:00 ─────────────── 05:00       │
├─────────────────────────────────────┤
│ Timeline                            │
│                                     │
│ VIDEO   █████ ██████ █████          │
│ IMAGE   ████  █████  ████           │
│ VOICE   ██ ██ ███ ██                │
│ MUSIC   █████████████████████       │
│ SFX       ██   █ █      ██          │
│                                     │
└─────────────────────────────────────┘

==================================================
二十四、Preview Player
==================================================

本阶段实现：

浏览器原生 Preview。

如果 Clip 是：

VIDEO：

使用 HTML5 video。

IMAGE：

使用 img。

AUDIO：

使用 audio / Web Audio。

Composition Player 根据：

currentTime

计算当前应该显示：

Visual Clip

Dialogue Audio

Music

SFX

==================================================
二十五、Preview 播放规则
==================================================

必须支持：

Play
Pause
Seek
Current Time
Duration

Timeline 播放：

currentTime
 ↓
找当前 Video Clip
 ↓
找当前 Image Clip
 ↓
找当前 Dialogue Clip
 ↓
找 Music
 ↓
找 SFX

然后同步播放。

本阶段不要求专业级音视频同步。

但数据模型必须为未来 Render 做准备。

==================================================
二十六、Visual Layer
==================================================

同时存在：

VIDEO
IMAGE

规则：

如果同一时间：

VIDEO + IMAGE

默认：

VIDEO 优先。

zIndex 更高者优先。

如果没有 Video：

显示 Image。

不要把 Image 转成 Video。

==================================================
二十七、Audio Mixer v1
==================================================

本阶段实现：

Dialogue
Music
SFX

基础音量控制。

TimelineClip.volume：

默认：

1.0

范围：

0.0 - 1.0

支持：

静音

Track muted：

不播放。

Track volume：

作为 Track × Clip 的最终音量。

不要实现：

Fade
Ducking
Compressor
EQ
Limiter
Reverb
Pan

这些属于未来 Audio Engine。

==================================================
二十八、Timeline API
==================================================

至少实现：

GET
/projects/:projectId/episodes/:episodeId/timeline

POST
/projects/:projectId/episodes/:episodeId/timeline/build

PATCH
/projects/:projectId/episodes/:episodeId/timeline

DELETE
/projects/:projectId/episodes/:episodeId/timeline

GET
/projects/:projectId/episodes/:episodeId/timeline/manifest

GET
/projects/:projectId/episodes/:episodeId/timeline/preview

Track：

GET
/projects/:projectId/timelines/:timelineId/tracks

POST
/projects/:projectId/timelines/:timelineId/tracks

PATCH
/projects/:projectId/timelines/:timelineId/tracks/:trackId

DELETE
/projects/:projectId/timelines/:timelineId/tracks/:trackId

Clip：

GET
/projects/:projectId/timelines/:timelineId/clips

POST
/projects/:projectId/timelines/:timelineId/clips

PATCH
/projects/:projectId/timelines/:timelineId/clips/:clipId

DELETE
/projects/:projectId/timelines/:timelineId/clips/:clipId

==================================================
二十九、Shared Types
==================================================

新增：

EpisodeTimeline
TimelineTrack
TimelineClip
TimelineTrackType
TimelineClipType
TimelineClipSourceType
TimelineStatus
CompositionManifest
CompositionTrack
CompositionClip
TimelineBuildResult
TimelineContinuityResult

不要在 Web / Mobile / Desktop 重复定义。

全部放：

packages/types

==================================================
三十、API Client
==================================================

新增：

getEpisodeTimeline()
buildEpisodeTimeline()
updateEpisodeTimeline()
deleteEpisodeTimeline()

getTimelineTracks()
createTimelineTrack()
updateTimelineTrack()
deleteTimelineTrack()

getTimelineClips()
createTimelineClip()
updateTimelineClip()
deleteTimelineClip()

getCompositionManifest()
getCompositionPreview()

页面禁止直接 fetch API。

==================================================
三十一、Core Package
==================================================

新增：

packages/core/src/timeline.ts

至少包含：

calculateShotTimeline()
calculateSceneTimeline()
calculateDialogueTimeline()
buildTimelineManifest()
resolveVisualClip()
resolveAudioClip()
calculateTimelineDuration()
detectTimelineStale()

核心逻辑必须：

纯函数优先。

不要把 Prisma 查询写进 Core。

==================================================
三十二、API Service 拆分
==================================================

不要创建一个巨大的 TimelineService。

至少拆成：

TimelineService
TimelineBuilderService
TimelineTimingService
TimelineContinuityService
CompositionService

职责：

TimelineService
→ CRUD

TimelineBuilderService
→ 自动构建 Timeline

TimelineTimingService
→ 时间计算

TimelineContinuityService
→ 数据一致性

CompositionService
→ Timeline → Manifest

==================================================
三十三、Web 页面
==================================================

新增：

/projects/:id/timeline

如果项目级 Timeline 页面不合理，可以 redirect 到：

/projects/:id/episodes

新增：

/projects/:id/episodes/:episodeId/timeline

Episode 页面增加：

Timeline

入口。

同时：

Storyboard
Images
Videos
Voices
Music
SFX

全部可以跳转 Timeline。

==================================================
三十四、Episode Workspace
==================================================

Episode Workspace 应形成：

Overview
Story
Script
Storyboard
Images
Videos
Voices
Music
SFX
Timeline

其中：

Timeline 是：

Composition Layer

而不是新的内容生产模块。

==================================================
三十五、Mobile
==================================================

Mobile：

可以：

查看 Timeline
播放 Preview
查看 Track
播放 Audio
查看 Video

可以做：

基础 Clip 编辑

但不要做复杂拖拽时间轴。

复杂 Timeline 编辑：

引导 Web。

==================================================
三十六、Desktop
==================================================

Desktop：

可以：

查看 Timeline
Preview
基础编辑

但本阶段不要做：

专业 NLE

不要做：

剪映级编辑器
Premiere 级编辑器
Final Cut 级编辑器

Timeline v1 只是：

Composition Preview。

==================================================
三十七、Asset URL
==================================================

继续使用：

AssetStorageService

禁止：

Web / API 直接读取 storage filesystem。

所有 Asset：

通过项目隔离后的 Asset API 获取。

必须检查：

projectId

assetId

ownership

禁止跨项目读取。

==================================================
三十八、安全
==================================================

必须继续：

API Key 不出现在 API response。

Timeline 不允许包含：

apiKey
encryptedApiKey

GenerationTask 不进入 CompositionManifest。

日志继续：

sanitizeSecret

==================================================
三十九、不要引入 AI
==================================================

这是本阶段一个非常重要的要求。

Phase 13：

不要增加新的 AI Generation。

Timeline Build 是确定性程序。

不要：

AI 自动剪辑
AI 自动改节奏
AI 自动配乐
AI 自动补镜头
AI 自动补声音

这些属于未来 Agent / Intelligent Editing。

==================================================
四十、不要引入新的 Provider
==================================================

不要新增 Provider。

不要新增：

Render Provider
FFmpeg Provider
Cloud Render Provider

本阶段只建立：

Composition 抽象。

未来 Render Engine 再实现。

==================================================
四十一、错误码
==================================================

至少增加：

TIMELINE_NOT_FOUND
TIMELINE_ALREADY_EXISTS
TIMELINE_ALREADY_LOCKED
TIMELINE_STALE
TIMELINE_PROJECT_MISMATCH
TIMELINE_EPISODE_MISMATCH

TIMELINE_TRACK_NOT_FOUND
TIMELINE_CLIP_NOT_FOUND

TIMELINE_INVALID_SOURCE
TIMELINE_ASSET_NOT_FOUND
TIMELINE_ASSET_PROJECT_MISMATCH

TIMELINE_INVALID_TIME_RANGE
TIMELINE_INVALID_DURATION

TIMELINE_BUILD_FAILED
COMPOSITION_BUILD_FAILED

MISSING_VISUAL_ASSET
MISSING_DIALOGUE_AUDIO
MISSING_MUSIC_ASSET
MISSING_SFX_ASSET

==================================================
四十二、Validation
==================================================

Clip：

startTime >= 0

duration > 0

sourceStartTime >= 0

sourceDuration > 0

不能：

startTime < 0

duration <= 0

volume < 0

volume > 1

speed <= 0

opacity < 0

opacity > 1

Timeline：

durationSeconds >= 0

fps > 0

==================================================
四十三、锁定机制
==================================================

TimelineStatus：

DRAFT
PREVIEW_READY
STALE
LOCKED

LOCKED Timeline：

禁止：

修改 Track
修改 Clip
删除 Timeline

需要：

unlock

本阶段可以提供：

POST /timeline/unlock

但不要实现复杂权限系统。

==================================================
四十四、Timeline Lock 的意义
==================================================

未来：

Timeline LOCKED
 ↓
Render Engine
 ↓
Final Video
 ↓
Publish

所以：

LOCKED 是未来 Render 的输入状态。

但是本阶段：

LOCKED 不产生任何视频。

==================================================
四十五、GenerationTask
==================================================

本阶段：

不新增 Timeline GenerationTask。

Timeline Build 不调用 AI。

不要为了 Timeline 创建假的 GenerationTask。

==================================================
四十六、Database Migration
==================================================

新增：

EpisodeTimeline
TimelineTrack
TimelineClip

migration 名称：

20260818110000_episode_timeline_composition

如果实际执行时间略有不同，以当前 migration naming convention 为准。

必须：

pnpm exec prisma migrate deploy

然后：

pnpm prisma validate

绝对不能 reset。

==================================================
四十七、Seed
==================================================

新增：

prisma/seed-timeline-demo.ts

命令：

pnpm db:seed:timeline-demo

不要自动执行。

Seed 必须依赖：

星河碰撞
Season 1
Episode 1
Script
Storyboard
Shot
Image Asset
Video Asset
TTS Asset
Music
SFX

如果依赖不存在：

明确提示：

请先执行：

pnpm db:seed:script-demo
pnpm db:seed:storyboard-demo
pnpm db:seed:image-demo
pnpm db:seed:video-demo
pnpm db:seed:tts-demo
pnpm db:seed:music-sfx-demo

不要自己偷偷 reset 或创建重复上游数据。

==================================================
四十八、Seed 内容
==================================================

Timeline Demo 至少包含：

VIDEO Track
IMAGE Track
DIALOGUE Track
MUSIC Track
SFX Track

至少：

10 个 Clips。

至少包含：

3 个视觉 Clip
3 个 Dialogue Clip
1 个 Music Clip
3 个 SFX Clip

必须真实引用现有 Asset。

不要创建假的 Asset URL。

==================================================
四十九、测试要求
==================================================

必须新增完整测试。

至少覆盖：

1. Timeline CRUD

2. Project isolation

3. Episode isolation

4. Track CRUD

5. Clip CRUD

6. Invalid time validation

7. Asset ownership

8. Storyboard ownership

9. ScriptBlock ownership

10. EpisodeAudioAsset ownership

11. Timeline Builder

12. Visual asset selection

13. Final Video priority

14. Final Image fallback

15. Missing visual asset

16. Dialogue timing

17. Missing dialogue audio

18. Music timing

19. SFX timing

20. Scene timing

21. Shot timing

22. Timeline duration

23. Version

24. Stale detection

25. Rebuild

26. Rebuild version increment

27. Manifest generation

28. Cross-project rejection

29. Locked Timeline

30. Track muted

31. Clip disabled

32. Volume validation

33. Composition Preview

34. API Key never exposed

35. GenerationTask never included in Manifest

目标：

所有测试必须通过。

==================================================
五十、测试数量
==================================================

不要求固定数字。

但 Phase 12：

60 files / 372 tests

本阶段新增合理测试。

最终：

pnpm test

必须全部通过。

==================================================
五十一、Build
==================================================

完成后必须运行：

pnpm prisma validate
pnpm lint
pnpm typecheck
pnpm test
pnpm build

全部通过后才能宣布：

Phase 13 COMPLETE

==================================================
五十二、API Health
==================================================

确认：

GET http://localhost:3011/health

返回：

{
  "status": "ok"
}

如果 3011 被旧进程占用：

不要杀进程。

不要暴力结束 Node。

先判断当前服务是否健康。

如果必须加载新代码：

提示需要用户手动重启 API。

不要为了测试而破坏当前服务。

==================================================
五十三、数据完整性
==================================================

必须确认：

Project = 1
World = 1
Character = 3
Season = 1
Episode = 3
Provider = 1

并确认：

旧 GenerationTask 保留。

旧 Script 保留。

旧 Storyboard 保留。

旧 Asset 保留。

不能因为 Timeline migration 导致：

Script = 0
Storyboard = 0
Asset = 0

==================================================
五十四、现有功能不能被破坏
==================================================

必须验证：

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

仍然可以访问。

尤其：

IMAGE 不得 fallback 到 TEXT Provider。

VIDEO 不得 fallback 到 IMAGE Provider。

TTS 不得 fallback 到 CHAT。

MUSIC 不得 fallback 到 TTS。

SFX 不得 fallback 到 TTS。

Timeline 不调用任何 Provider。

==================================================
五十五、前端体验
==================================================

Timeline 页面必须告诉用户：

“这是合成预览，不是最终视频导出。”

如果存在缺失资产：

明确显示：

视觉素材缺失
对白音频缺失
音乐缺失
音效缺失

不要假装：

“正在生成”。

因为 Timeline 本身不生成素材。

==================================================
五十六、Timeline 自动构建后的提示
==================================================

如果：

Video 有
Image 有
Dialogue 有
Music 有
SFX 有

显示：

“时间线已就绪，可预览。”

如果存在缺失：

例如：

3 个 Shot 没有视频
1 个 Dialogue 没有配音

显示：

“时间线存在缺失素材。”

但允许 Preview。

==================================================
五十七、Preview 的降级策略
==================================================

如果：

VIDEO 缺失
但 IMAGE 存在：

使用 IMAGE。

如果：

VIDEO 和 IMAGE 都缺：

显示：

Visual Placeholder

例如：

“Shot 03 — Missing Visual”

绝对不要：

自动调用 Image AI。

绝对不要：

自动调用 Video AI。

==================================================
五十八、Audio 降级
==================================================

Dialogue 缺失：

不阻塞整个 Timeline。

显示：

Missing Dialogue Audio。

Music 缺失：

Timeline 仍可播放。

SFX 缺失：

Timeline 仍可播放。

Timeline Preview 必须是：

best-effort composition。

==================================================
五十九、未来 Render 兼容性
==================================================

CompositionManifest 必须设计为未来可直接被：

FFmpeg
Remotion
WebCodecs
Cloud Render

消费。

但本阶段：

不要实现这些 Render Engine。

不要引入依赖。

不要提前安装 FFmpeg。

==================================================
六十、未来完整产品路线
==================================================

在实现 Phase 13 时，请把未来架构考虑清楚，但不要实现：

Phase 14：

Render Engine

Timeline
 ↓
Render Job
 ↓
FFmpeg / Render Worker
 ↓
Episode MP4

Phase 15：

Subtitle / Caption Engine

Phase 16：

Audio Mixing / Mastering

Phase 17：

Batch Generation / Production Pipeline

Phase 18：

AI Intelligent Editing / Agent

Phase 19：

User Account / BYOK / Billing

Phase 20：

Publishing

YouTube
TikTok
Bilibili
Instagram
etc.

本阶段不要越界实现这些。

==================================================
六十一、非常重要：用户最终产品目标
==================================================

整个 AI Drama Studio 的最终目标是：

用户输入：

故事大纲

然后：

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
Render
↓
Subtitles
↓
Final Episode
↓
Publish

未来最终用户应该尽可能只需要：

1. 故事大纲
2. 每季大纲
3. 每集大纲
4. 风格 / 视觉 / 声音偏好
5. 确认关键 AI 生成结果

系统负责：

批量生成
+
检查
+
Preview
+
Apply
+
生产
+
最终发布

但：

本阶段绝对不要实现“一键生成整集”。

本阶段只是为未来“一键生产”建立可靠的 Timeline / Composition 基础。

==================================================
六十二、用户成本模型必须保持
==================================================

继续保持：

平台可以提供默认 Provider 作为 Demo / 开发体验。

但长期：

用户自己承担 AI 成本。

不同能力：

CHAT
STRUCTURED_OUTPUT
IMAGE
VIDEO
IMAGE_TO_VIDEO
TTS
MUSIC
SFX

必须允许：

不同 Provider
不同 Model

分别配置。

Timeline 不产生 AI 成本。

Composition Preview 不产生 AI 成本。

==================================================
六十三、BYOK 架构不能破坏
==================================================

继续使用：

ProjectAiConfig

ProviderResolver

AiCapability

不得重新引入：

AI_API_KEY
AI_MODEL
AI_BASE_URL

直接读取。

业务代码必须继续：

resolveForCapability()

==================================================
六十四、不要做“假功能”
==================================================

严格禁止：

假 Render
假 MP4
假 AI 生成
假费用
假取消
假 Provider
假云存储
假平台发布

如果当前能力没有真实 Provider：

明确显示：

NOT_CONFIGURED
或
NOT_SUPPORTED

==================================================
六十五、最终验收标准
==================================================

只有满足以下全部条件，才宣布：

# Phase 13 COMPLETE

条件：

[ ] EpisodeTimeline 完成
[ ] TimelineTrack 完成
[ ] TimelineClip 完成
[ ] Migration 完成
[ ] Timeline CRUD 完成
[ ] Track CRUD 完成
[ ] Clip CRUD 完成
[ ] Timeline Builder 完成
[ ] Timing Service 完成
[ ] Continuity 完成
[ ] Stale 检测完成
[ ] Rebuild 完成
[ ] Version 完成
[ ] CompositionManifest 完成
[ ] Preview 完成
[ ] Visual fallback 完成
[ ] Dialogue timing 完成
[ ] Music timing 完成
[ ] SFX timing 完成
[ ] Project isolation 完成
[ ] Asset isolation 完成
[ ] Web Timeline 完成
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
[ ] API Key 无泄漏
[ ] 没有引入 AI Provider
[ ] 没有实现 FFmpeg
[ ] 没有实现最终视频导出
[ ] 没有实现发布
[ ] 没有实现 Billing
[ ] 没有实现 Login/JWT
[ ] 没有实现 Redis/BullMQ

==================================================
六十六、完成后必须输出完整报告
==================================================

完成后，请严格按照以下格式汇报：

# Phase 13 Complete

## 1. Database
新增 Model：
...

Migration：
...

## 2. Timeline Architecture
...

## 3. Timeline Builder
...

## 4. Timing
...

## 5. Composition Manifest
...

## 6. Preview
...

## 7. API
...

## 8. Web
...

## 9. Mobile
...

## 10. Desktop
...

## 11. Seed
...

## 12. Tests
...

## 13. lint
...

## 14. typecheck
...

## 15. prisma validate
...

## 16. build
...

## 17. API health
...

## 18. Existing Data
...

## 19. Existing Provider
...

## 20. Existing GenerationTask
...

## 21. Remaining Limitations
...

## 22. Explicitly NOT Implemented
...

## 23. Recommended Next Phase
...

==================================================
六十七、执行纪律
==================================================

你现在处于 Cursor 自动执行模式。

因此：

不要频繁询问我是否继续。

不要因为发现一个小问题就暂停整个 Phase。

遇到普通实现问题：

自行判断并修复。

遇到类型错误：

自行修复。

遇到测试失败：

自行修复。

遇到 migration 问题：

优先使用新的增量 migration 修复。

绝对不要 reset 数据库。

遇到 API 端口占用：

不要杀现有进程。

检查 health。

必要时说明需要用户手动重启。

遇到 Windows Prisma DLL EPERM：

不要暴力删除占用文件。

不要破坏运行中的 API。

按照当前项目已有方式处理。

==================================================
六十八、最终边界
==================================================

本阶段只做到：

Storyboard / Script / Assets
        ↓
Episode Timeline
        ↓
Composition Manifest
        ↓
Browser Preview

停止。

不要继续做：

Render
FFmpeg
Export
Subtitle
Mixing
Publishing

等下一阶段。

现在开始执行：

# Phase 13 — Episode Timeline / Composition Preview Engine v1

请直接检查当前项目实际代码状态，在现有架构上增量实现。

不要假设文件存在。

不要重建已有模块。

不要删除现有功能。

不要 reset 数据库。

完成全部实现、测试、lint、typecheck、prisma validate、build 后，再输出完整 Phase 13 Complete 报告。