你现在开始执行：

# Phase 11 — TTS / Voice Engine v1

目标：

在现有 AI Drama Studio 架构基础上，实现第一版 TTS / Voice Engine。

本阶段必须保持现有 Monorepo、World、Character、Story Bible、Season、Episode、Script、Storyboard、Image、Video、Provider、Capability 架构不变。

禁止重建项目、禁止 reset 数据库、禁止删除历史 migration、禁止 DROP 现有业务表、禁止更换技术栈。

==================================================
一、核心产品目标
==================================================

建立：

ScriptBlock / StoryboardShot Dialogue
        ↓
Voice / TTS 配置
        ↓
ProjectAiConfig(TTS)
        ↓
ProviderResolver
        ↓
TTS Provider Adapter
        ↓
Preview
        ↓
Apply
        ↓
Asset(type=AUDIO)
        +
Dialogue / Shot 音频关联

最终形成：

Story
 ↓
Script
 ↓
Storyboard
 ↓
Image
 ↓
Video

以及：

Script Dialogue
 ↓
TTS
 ↓
Audio Asset

本阶段只负责：

「文字 → 语音」

不负责：

- Timeline
- FFmpeg
- 字幕烧录
- BGM
- 音效混音
- Episode Render
- 最终 MP4
- 自动发布
- Voice Clone
- Lip Sync
- 多轨音频编辑

这些留到后续阶段。

==================================================
二、非常重要的架构原则
==================================================

必须继续使用现有：

Capability
Provider
Model
ProjectAiConfig
ProviderResolver
GenerationTask
Asset
Preview → Apply
Transaction

禁止重新设计一套 AI Provider 系统。

本阶段新增能力：

AiCapability.TTS

如果 VOICE_CLONE 已经存在于枚举，则继续保留，但：

本阶段不得实现 Voice Clone。

如果 TTS 枚举已经存在，则直接复用，不重复创建。

AI Provider 必须通过：

resolveForCapability(projectId, AiCapability.TTS)

获取。

绝对禁止：

- 直接读取 AI_API_KEY
- 直接读取 AI_MODEL
- 直接读取 AI_BASE_URL
- 在业务 Service 中写死 DeepSeek
- TTS fallback 到 CHAT
- TTS fallback 到 STRUCTURED_OUTPUT
- TTS fallback 到 IMAGE
- TTS fallback 到 VIDEO

如果没有 TTS Provider：

返回明确：

TTS_PROVIDER_NOT_CONFIGURED

如果 Provider 存在但不支持 TTS：

返回：

CAPABILITY_NOT_SUPPORTED

==================================================
三、Provider Architecture
==================================================

扩展现有 AiProvider 接口。

新增：

generateSpeech()

建议抽象为：

generateSpeech(input): Promise<GeneratedAudio>

input 至少支持：

- text
- voice
- language
- format
- speed
- pitch
- provider/model specific options

output 至少包含：

- url 或 base64
- mimeType
- format
- durationSeconds（如果 Provider 能提供）
- provider
- model
- metadata

不要要求所有 Provider 都必须返回 duration。

如果 Provider 不返回 duration：

允许为 null。

==================================================
四、OpenAI Compatible TTS Adapter
==================================================

在现有：

OpenAiCompatibleProvider

中增加：

generateSpeech()

默认协议：

POST {baseUrl}/audio/speech

请求结构按照 OpenAI-compatible TTS 的常见形式设计：

{
  model,
  input,
  voice,
  response_format,
  speed
}

但必须注意：

不同 Provider 的 TTS API 并不完全兼容。

因此：

不要假设所有 OpenAI Compatible Provider 都支持 TTS。

Provider 不支持：

返回 CAPABILITY_NOT_SUPPORTED。

HTTP 404：

映射为：

CAPABILITY_NOT_SUPPORTED

不要返回 500。

API Key：

只能存在服务端。

日志：

禁止打印 API Key。

错误：

必须经过现有 secret sanitization。

==================================================
五、TTS 数据模型设计
==================================================

不要创建：

TtsAsset
VoiceAsset
AudioAsset

等重复模型。

继续复用：

Asset

使用：

AssetType.AUDIO

如果 AssetType.AUDIO 已存在：

直接复用。

如果不存在：

增量增加 AUDIO。

==================================================
六、VoiceProfile
==================================================

Character 已经存在：

voiceProfile

本阶段不要把 voiceProfile 强行设计成完整 Provider 配置。

它应该继续作为：

「角色声音偏好 / 声音身份」

例如：

{
  "voiceId": "...",
  "provider": "...",
  "language": "zh-CN",
  "style": "calm",
  "speed": 1,
  "pitch": 0
}

但是：

API Key / Provider Secret

绝对不能写入 Character.voiceProfile。

不要在 Character 表中保存 Provider API Key。

==================================================
七、Character Voice Configuration
==================================================

新增一个非常轻量的 Voice 配置概念。

如果现有 Character.voiceProfile 已经足够：

优先复用，不新增重复表。

但是建议至少支持：

- voiceId
- language
- speed
- pitch
- style

角色 Voice Profile 的职责：

告诉 TTS：

「这个角色应该用什么声音」

不是保存真正的音频文件。

==================================================
八、Audio Asset 与业务关联
==================================================

需要建立：

TTS Generation
        ↓
Asset(type=AUDIO)
        ↓
关联 ScriptBlock / Dialogue

建议新增：

ScriptBlockAsset

或者如果项目已经有通用 Asset Relation 结构：

优先复用。

如果没有：

新增一个最小模型，例如：

ScriptBlockAsset

字段：

- id
- scriptBlockId
- assetId
- role
- isPrimary
- createdAt

role 建议：

GENERATED
FINAL
REFERENCE

不要创建过度复杂的 AudioTrack 模型。

因为真正 Audio Track / Timeline 属于后续 Editing 阶段。

==================================================
九、GenerationTask
==================================================

继续复用 GenerationTask。

新增：

GenerationTaskType.TTS

如果已经存在则直接复用。

capability：

AiCapability.TTS

usage：

继续使用现有 GenerationTask.usage。

至少记录：

{
  durationMs,
  characterCount
}

如果 Provider 能提供：

- audioDurationSeconds
- estimatedUnits

可以记录。

但是：

禁止伪造费用。

不要实现 Billing。

==================================================
十、TTS Generation API
==================================================

新增：

POST

/projects/:projectId/generations/tts

用于：

单条 ScriptBlock Dialogue → TTS

输入至少支持：

{
  episodeId,
  scriptBlockId,
  text?,
  characterId?,
  voiceId?,
  language?,
  speed?,
  pitch?,
  format?
}

设计原则：

默认从 ScriptBlock 获取 Dialogue。

不要要求前端重复提交完整 Script。

如果 scriptBlockId 存在：

服务端验证：

Project
→ Episode
→ Script
→ Scene
→ ScriptBlock

全部属于当前 project。

禁止跨项目引用。

==================================================
十一、TTS 输入来源
==================================================

优先：

ScriptBlock.type === DIALOGUE

如果不是 Dialogue：

返回：

TTS_SOURCE_NOT_DIALOGUE

如果 Dialogue 没有 characterId：

允许生成。

这种情况表示：

「旁白 / 未绑定角色对白」

但必须允许用户指定：

voiceId

如果 Dialogue 已关联 Character：

优先：

Character.voiceProfile.voiceId

如果用户显式传入 voiceId：

用户传入优先。

优先级：

Request voiceId
↓
Character.voiceProfile.voiceId
↓
Provider default voice
↓
没有则 TTS_VOICE_REQUIRED

==================================================
十二、文本处理
==================================================

不要直接把任意长文本丢给 Provider。

增加：

TTS input normalization。

至少：

- trim
- 空文本拒绝
- 最大字符长度限制
- 去除明显无意义控制字符
- 保留中文标点
- 不破坏对白内容

如果超过 Provider 单次限制：

本阶段不要自动拆分成多个音频。

直接返回：

TTS_TEXT_TOO_LONG

未来再实现语音分段。

==================================================
十三、Preview / Apply
==================================================

严格保持：

Generate
↓
GenerationTask
↓
Preview
↓
Apply

Generate 阶段：

不创建正式 Asset。

GenerationTask.output 保存：

{
  previewUrl / temporaryUrl,
  mimeType,
  format,
  durationSeconds,
  voice,
  model,
  provider
}

注意：

如果 Provider 返回临时 URL：

必须考虑 URL 过期。

不要把临时 URL 当永久 Asset URL。

Apply 时：

下载 / 获取 Provider 返回的音频
↓
AssetStorageService
↓
Asset(type=AUDIO)
↓
ScriptBlockAsset
↓
GenerationTask.appliedAt

正式 Asset 必须使用项目自己的 storageKey。

==================================================
十四、Storage
==================================================

继续复用：

AssetStorageService

禁止业务层直接 fs。

目录：

storage/assets/{projectId}/{assetId}/audio.*

MIME：

audio/mpeg
audio/wav
audio/ogg
audio/mp4
audio/aac

根据实际 Provider 返回类型确定。

不要伪造 MIME。

==================================================
十五、Asset Metadata
==================================================

Asset.metadata 可以保存：

{
  source: "tts",
  scriptBlockId,
  characterId,
  voiceId,
  language,
  provider,
  model,
  format,
  generationTaskId
}

不要保存：

apiKey
encryptedApiKey
完整 Provider credentials

==================================================
十六、重新生成
==================================================

重新生成：

必须创建新的 GenerationTask。

Apply 后：

旧 Audio Asset：

保留历史。

旧 ScriptBlockAsset：

isPrimary = false

新：

isPrimary = true

不要删除历史音频。

用户可以查看：

v1
v2
v3

与 Image / Video 保持一致。

==================================================
十七、Primary Audio
==================================================

一个 ScriptBlock：

最多一个：

Primary Final Audio

提供：

GET：

/projects/:projectId/script-blocks/:scriptBlockId/assets

以及：

POST：

/projects/:projectId/script-blocks/:scriptBlockId/assets/:assetId/primary

setPrimary：

不调用 AI。

只修改关联关系。

==================================================
十八、TTS Provider Capability
==================================================

更新：

GET /ai/capabilities

必须包含：

TTS

前端 Provider 管理：

允许 Provider 勾选：

Chat
Structured Output
Image
Video
Image to Video
TTS
...

实际是否能调用：

由 Provider capability + Model capability 决定。

如果一个 Provider：

支持 Chat
支持 Structured Output
不支持 TTS

那么：

TTS 不允许配置。

==================================================
十九、AiModel
==================================================

保持现有：

AiModel.capabilities

TTS Model 必须明确声明：

TTS

例如：

Provider:

OpenAI Compatible Provider

Models:

text-model
capabilities:
CHAT
STRUCTURED_OUTPUT

tts-model
capabilities:
TTS

不要允许：

使用 text-model 生成 TTS。

Resolver / Config Service 必须验证：

model supports TTS。

==================================================
二十、Project AI Settings
==================================================

在：

/projects/:id/settings

增加：

TTS

显示：

TTS Provider
TTS Model
Voice
Status

例如：

TTS
----------------
Provider:
未配置

Model:
未配置

[配置 TTS]

或者：

Provider:
xxx

Model:
xxx

Status:
Ready

注意：

不要自动使用项目 IMAGE / VIDEO Provider。

每个 Capability 独立。

==================================================
二十一、Provider 表单
==================================================

/ai-providers

Provider Capability：

TTS

Model Capability：

TTS

需要清楚区分：

Provider 支持 TTS

和：

Model 支持 TTS。

UI 不允许用户选择一个没有 TTS capability 的 Model。

==================================================
二十二、Web TTS UI
==================================================

在 ScriptBlock / Dialogue 编辑区域增加：

「生成语音」

点击：

打开：

TTS Generation Modal

显示：

角色：

沈星河

文本：

「你是谁？」

Voice：

xxx

Language：

中文

Speed：

1.0

Pitch：

0

Provider：

xxx

Model：

xxx

按钮：

[生成语音]

==================================================
二十三、Preview Modal
==================================================

生成完成后：

显示：

Audio Player

播放 / 暂停

显示：

Provider
Model
Voice
Duration

按钮：

[重新生成]

[应用为最终语音]

不要直接写 Asset。

==================================================
二十四、角色声音
==================================================

Character Detail：

增加：

「声音配置」

显示：

Voice ID
Language
Speed
Pitch
Style

允许编辑。

但是：

本阶段不要实现：

Voice Clone
上传声音样本
声音训练
Voice Embedding

这些留到后续。

==================================================
二十五、项目 Audio 页面
==================================================

新增：

/projects/:id/audio

显示：

Audio Assets

筛选：

- Dialogue
- Narration
- Character
- Episode

显示：

播放器

Provider
Model
Voice
Duration
Created At

支持：

设为最终

但不要做：

Timeline
剪辑
拼接
混音

==================================================
二十六、Episode Audio 概念
==================================================

本阶段不要创建：

EpisodeAudioTrack

不要把多个 Dialogue 音频合并成 Episode Audio。

现在只保存：

「独立 Dialogue Audio Assets」

未来：

Timeline / Editing 阶段

再把：

Dialogue Audio
+
Video
+
BGM
+
SFX
+
Subtitle

组合。

==================================================
二十七、Context Builder
==================================================

新增：

buildTtsContext()

但必须保持轻量。

输入：

Project
Episode
Scene
ScriptBlock
Character voiceProfile

不要把：

World 全量
Story Bible 全量
GenerationTask 全量
ImageProfile
Voice secret

塞进 Prompt。

TTS 主要需要：

text
character
voice preferences
language

==================================================
二十八、Continuity
==================================================

TTS 生成前检查：

Project
Episode
Script
Scene
ScriptBlock
Character

归属关系。

如果 characterId 存在：

必须属于当前 Project。

如果：

Character.voiceProfile.voiceId

不存在：

允许用户手动选择 voiceId。

如果两者都没有：

TTS_VOICE_REQUIRED。

==================================================
二十九、Generation Apply Transaction
==================================================

Apply 必须：

prisma.$transaction

流程：

1. 获取 GenerationTask
2. 验证 task.type === TTS
3. 验证 projectId
4. 检查 appliedAt
5. 获取 Provider output
6. 获取音频文件
7. 保存 Asset
8. 创建 ScriptBlockAsset
9. 旧 Primary → false
10. 新 Asset → FINAL + primary
11. GenerationTask.appliedAt
12. commit

任意一步失败：

rollback 数据库。

如果文件已经写入：

必须执行补偿删除。

不能留下孤儿文件。

==================================================
三十、错误码
==================================================

至少支持：

TTS_PROVIDER_NOT_CONFIGURED
TTS_PROVIDER_DISABLED
TTS_MODEL_NOT_SUPPORTED
TTS_VOICE_REQUIRED
TTS_SOURCE_NOT_DIALOGUE
TTS_TEXT_EMPTY
TTS_TEXT_TOO_LONG
TTS_GENERATION_FAILED
TTS_APPLY_FAILED
TTS_ASSET_NOT_FOUND
TTS_ASSET_PROJECT_MISMATCH

复用已有：

CAPABILITY_NOT_SUPPORTED
GENERATION_ALREADY_APPLIED

错误信息：

禁止泄漏 API Key。

==================================================
三十一、不要实现假 Provider
==================================================

非常重要：

不要为了 Demo 创建假 TTS Provider。

如果当前 DeepSeek Provider 不支持 TTS：

页面必须明确：

「当前项目尚未配置 TTS 模型」

这是正确产品状态。

可以保留：

seed TTS demo

但：

只允许写本地 fixture 音频。

不得伪装成真实 AI 生成。

==================================================
三十二、Seed
==================================================

新增：

prisma/seed-tts-demo.ts

命令：

pnpm db:seed:tts-demo

要求：

- 不调用真实 AI
- 使用本地 fixture 音频
- 创建 GenerationTask(type=TTS)
- 创建 Asset(type=AUDIO)
- 创建 ScriptBlockAsset
- 标记 demo metadata

依赖：

星河碰撞
Season
Episode
Script
ScriptBlock

如果缺少：

明确提示先执行：

pnpm db:seed:script-demo

不要自动 seed。

==================================================
三十三、API Client
==================================================

所有 Web API：

必须经过：

packages/api-client

新增：

createTtsGeneration()
getTtsGeneration()
applyTtsGeneration()

以及：

getScriptBlockAssets()
setPrimaryScriptBlockAsset()

不要在 Web 页面直接 fetch。

==================================================
三十四、Shared Types
==================================================

新增：

TtsGenerationInput

TtsGenerationResult

GeneratedAudio

VoiceProfile

ScriptBlockAudioAsset

TtsGenerationOptions

不要在：

Web
Mobile
Desktop

重复定义类型。

==================================================
三十五、Mobile
==================================================

增加：

Script → Dialogue → Audio

可以：

播放已有 Audio

查看 Voice

查看状态

基础编辑 Voice Profile。

复杂：

TTS Generation

引导 Web。

不要复制完整 TTS 生成工作台。

==================================================
三十六、Desktop
==================================================

保持现有 Desktop App Shell。

支持：

查看 Dialogue Audio

播放

查看 Voice

基础管理。

复杂 AI 生成引导 Web。

不要提前实现 Timeline。

==================================================
三十七、未来架构预留
==================================================

本阶段必须为未来保留：

VOICE_CLONE
MUSIC
SFX

但不要实现。

未来最终音频生产链：

Dialogue
 ↓
TTS
 ↓
Voice Audio

Music
 ↓
Music Asset

SFX
 ↓
SFX Asset

然后：

Timeline / Editing Engine

统一组合：

Video
+
Dialogue Audio
+
Music
+
SFX
+
Subtitle

最终：

Episode Render
↓
Final Video
↓
Publish

本阶段不要提前实现这些模块。

==================================================
三十八、重要：成本模型
==================================================

项目最终采用：

BYOK / User Pays

平台默认 Provider：

仅用于：

Demo
Development
测试

绝对不能设计成：

平台永久替用户支付 TTS 成本。

UI 文案明确：

「语音生成费用由当前项目配置的 Provider 账户承担。」

未来：

User
 ↓
Provider
 ↓
Capability
 ↓
Model

用户自己承担：

文本
图片
视频
TTS
Music

等 AI 成本。

不要实现 Billing。

==================================================
三十九、测试
==================================================

必须增加完整测试。

至少覆盖：

1. TTS Capability
2. Resolver TTS
3. Provider TTS
4. Model TTS capability validation
5. ProjectAiConfig TTS
6. ScriptBlock project isolation
7. 非 Dialogue 拒绝
8. 空文本拒绝
9. 超长文本拒绝
10. Voice required
11. Character voiceProfile
12. Preview 不创建 Asset
13. GenerationTask SUCCESS
14. Provider failure
15. Schema / output validation
16. Apply 创建 Asset
17. Apply 创建 ScriptBlockAsset
18. Primary Audio
19. Re-generation 保留历史
20. Duplicate Apply
21. Transaction rollback
22. 文件补偿删除
23. API Key 不泄漏
24. 跨项目 Asset 拒绝
25. TTS 不 fallback 到其他 Capability
26. Seed 不调用真实 AI

测试必须全部通过。

==================================================
四十、质量检查
==================================================

完成后必须执行：

pnpm prisma validate

pnpm lint

pnpm typecheck

pnpm test

如果项目已有：

pnpm build

也执行。

所有失败必须修复。

不要因为测试失败而删除测试。

不要通过修改测试断言来掩盖实现问题。

==================================================
四十一、数据库安全
==================================================

必须：

增量 migration。

禁止：

prisma migrate reset

禁止：

DROP TABLE

禁止：

删除历史 migration

禁止：

重建数据库

必须确认：

Project
World
Character
StoryBible
Season
Episode
Script
Storyboard
Asset
AiProvider
GenerationTask

历史数据全部保留。

==================================================
四十二、兼容现有系统
==================================================

必须保证：

World Generation

Character Generation

Story Bible Generation

Season Generation

Episode Generation

Script Generation

Storyboard Generation

Image Generation

Video Generation

全部继续工作。

不能因为 TTS 修改：

GenerationTask
Provider
Resolver
Asset

导致旧能力失效。

==================================================
四十三、最终产品链路验证
==================================================

完成后必须验证：

项目：

「星河碰撞」

至少存在：

Season 1
Episode 1
Script
Scene
Dialogue ScriptBlock

然后：

1. 项目设置
2. 配置 TTS Provider
3. 配置 TTS Model
4. 确认 Capability = TTS
5. 打开 Episode 1
6. 打开 Script
7. 找到 Dialogue
8. 点击「生成语音」
9. Provider Resolver 选择 TTS
10. 调用 TTS Provider
11. GenerationTask = TTS
12. Preview
13. 播放音频
14. Apply
15. 创建 Asset(type=AUDIO)
16. 创建 ScriptBlockAsset
17. 设置 Primary
18. Audio 页面可以看到
19. ScriptBlock 可以播放最终语音

如果没有真实 TTS Provider：

不要伪造成功。

应明确显示：

TTS_PROVIDER_NOT_CONFIGURED

或：

CAPABILITY_NOT_SUPPORTED

==================================================
四十四、文档
==================================================

新增：

docs/architecture/tts-generation.md

说明：

- TTS Architecture
- Provider Resolver
- Capability
- Voice Profile
- GenerationTask
- Asset
- ScriptBlockAsset
- Preview / Apply
- Storage
- BYOK
- Error handling
- Future Voice Clone
- Future Timeline

==================================================
四十五、完成报告
==================================================

完成后输出：

1. 修改文件
2. 新增 Model
3. 新增 Enum
4. Migration 名称
5. Migration 是否执行
6. TTS Provider Architecture
7. Voice Profile 设计
8. TTS Generation API
9. Preview / Apply
10. Asset Storage
11. ScriptBlock Audio Relation
12. Web UI
13. Mobile
14. Desktop
15. Seed
16. 测试数量
17. lint
18. typecheck
19. prisma validate
20. build
21. API health
22. Web health
23. 现有数据是否保留
24. Provider 是否保留
25. GenerationTask 是否保留
26. 是否修改 Monorepo
27. 是否存在遗留问题
28. 本阶段明确没有实现什么
29. 下一阶段建议

==================================================
四十六、下一阶段边界
==================================================

完成 Phase 11 后：

不要自动开始 Phase 12。

下一阶段候选：

Phase 12 — Music / SFX Engine v1

然后：

Phase 13 — Asset Library / Media Management

Phase 14 — Editing / Timeline Engine

Phase 15 — Episode Render Engine

Phase 16 — Subtitle / Caption

Phase 17 — Batch Production / Async Worker

Phase 18 — User / Auth / BYOK Ownership

Phase 19 — Billing / Usage

Phase 20 — Publishing / Platform Integration

但是本阶段结束后必须停止。

==================================================
四十七、最终架构目标
==================================================

必须保持整个项目最终形成：

                    Story Bible
                         │
                         ▼
World ───────────────► Script
 │                      │
Character ──────────────┤
 │                      ▼
Voice Profile       Storyboard
 │                      │
 ▼                      ├────► Image
TTS                      │
 │                       └────► Video
 ▼
Audio Asset

最终进入：

             ┌──────── Video
             │
             ├──────── Dialogue Audio
             │
             ├──────── Music
             │
             ├──────── SFX
             │
             └──────── Subtitle
                       │
                       ▼
                Timeline Engine
                       │
                       ▼
                 Episode Render
                       │
                       ▼
                 Final Video
                       │
                       ▼
                    Publish

本阶段只完成：

Dialogue → TTS → Audio Asset

不要越界实现 Timeline / Render / Publish。

现在开始执行 Phase 11。