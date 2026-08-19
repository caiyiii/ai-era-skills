# Phase 15 — Episode Production Pipeline & E2E Orchestration v1

你现在开始执行：

# Phase 15 — Episode Production Pipeline & E2E Orchestration v1

这是 AI Drama Studio 在 Phase 14 / Phase 14.5 / Phase 14.6 之后的下一阶段。

==================================================
一、项目当前真实状态
==================================================

这是一个已经连续完成多个阶段的 AI 漫剧创作平台。

当前核心能力已经存在：

Project
World
Characters
Story Bible
Season
Episode
Script
Storyboard
Image Generation
Video Generation
TTS / Voice
Music
SFX
Timeline / Composition Preview
Local FFmpeg Render
RenderJob
RenderArtifact

当前产品目标不是继续无止境增加独立 Engine。

本阶段的目标是：

# 把已有 Engine 真正串成一条完整的 Episode Production Pipeline。

==================================================
二、必须先理解当前产品架构
==================================================

用户不是在使用：

World Engine
Character Engine
Script Engine
Storyboard Engine
Image Engine
Video Engine
TTS Engine
Music Engine
SFX Engine
Timeline Engine
Render Engine

用户是在：

“制作一集漫剧”。

因此：

Episode 是生产中心。

必须建立：

Project
 ↓
Season
 ↓
Episode
 ↓
Episode Production Workspace
 ↓
Episode Plan
 ↓
Script
 ↓
Storyboard
 ↓
Assets
 ↓
Timeline
 ↓
Render
 ↓
Episode MP4

==================================================
三、Phase 14.5 / 14.6 已经解决的问题
==================================================

Phase 14.5 / 14.6 已经明确：

Season：

“这一季讲什么”

Episode：

“这一集讲什么”

Script：

“这一集具体怎么讲”

Storyboard：

“这一集具体怎么拍”

Assets：

“这一集需要什么画面和声音素材”

Timeline：

“把已经生成的素材编排起来”

Render：

“把 LOCKED Timeline 输出为成片”

本阶段不得重新定义这些概念。

==================================================
四、本阶段核心问题
==================================================

虽然上述 Engine 已经存在，

但当前系统仍然存在一个更深层的问题：

用户可能需要：

进入 Episode
↓
手动寻找 Script
↓
手动寻找 Storyboard
↓
手动寻找 Image
↓
手动寻找 Video
↓
手动寻找 Voice
↓
手动寻找 Music
↓
手动寻找 SFX
↓
手动进入 Timeline
↓
手动 Lock
↓
手动 Render

这仍然不是完整的生产工作流。

本阶段必须让：

Episode Workspace

成为整个生产过程的控制中心。

==================================================
五、核心目标
==================================================

最终用户进入：

E01 · 星门初现

之后：

不需要理解后端 Engine。

只需要按照：

① 剧集规划
② 剧本
③ 分镜
④ 视觉素材
⑤ 音频素材
⑥ 合成
⑦ 成片

逐步完成。

==================================================
六、Episode Production State
==================================================

需要建立一个：

“生产阶段计算 / 展示机制”。

优先检查当前 Episode / Script / Storyboard / Timeline / Render 已经存在的状态。

不要重复创建已有状态。

推荐 UI 层阶段：

PLANNING

SCRIPTING

STORYBOARDING

VISUAL_ASSETS

AUDIO_ASSETS

COMPOSING

READY_TO_RENDER

RENDERING

COMPLETED

如果当前数据库已有状态：

优先复用已有状态。

可以通过：

ProductionStageResolver

或：

EpisodeProductionService

计算当前阶段。

不要为了 UI 再创建一套互相冲突的数据库状态。

==================================================
七、Production Stage 必须是“计算结果”
==================================================

不要让用户手动选择：

“当前阶段”。

系统应该根据真实数据计算。

例如：

没有 Episode Plan：

PLANNING

Episode Plan 已完成：

SCRIPTING

Script 已确认：

STORYBOARDING

Storyboard 已确认：

VISUAL_ASSETS

视觉素材完成：

AUDIO_ASSETS

音频素材完成：

COMPOSING

Timeline 存在且 READY：

READY_TO_RENDER

Timeline LOCKED：

READY_TO_RENDER

Render Job：

RENDERING

RenderArtifact 成功：

COMPLETED

注意：

不要要求所有内容必须存在才能显示下一阶段。

必须根据项目实际数据做合理判断。

==================================================
八、Episode Workspace 成为 Production Control Center
==================================================

进入：

/projects/:projectId/episodes/:episodeId

页面必须成为：

Episode Production Workspace。

顶部：

← 第一季

E01 · 星门初现

状态：

制作中

当前阶段：

剧本

--------------------------------------------

Production Steps

✓ 剧集规划
● 剧本
○ 分镜
○ 视觉素材
○ 音频素材
○ 合成
○ 成片

--------------------------------------------

Current Task

剧本已准备好。

下一步：

确认剧本后生成分镜。

[查看剧本]
[确认剧本]

==================================================
九、每个阶段必须有明确 Action
==================================================

不要只有：

“已完成”。

必须告诉用户：

“下一步做什么”。

--------------------------------------------

Episode Plan：

[编辑剧集规划]

下一步：

[生成剧本]

--------------------------------------------

Script：

[查看剧本]

如果未确认：

[确认剧本]

确认后：

[生成分镜]

--------------------------------------------

Storyboard：

[查看分镜]

确认后：

[生成视觉素材]

--------------------------------------------

Visual Assets：

[查看素材]

下一步：

[生成音频素材]

--------------------------------------------

Audio Assets：

[查看音频]

下一步：

[进入合成]

--------------------------------------------

Timeline：

[打开合成]

如果未 LOCKED：

[锁定时间线]

如果 LOCKED：

[Render Episode]

--------------------------------------------

Render：

[查看成片]

==================================================
十、不要实现“假一键生成”
==================================================

本阶段虽然是 Pipeline，

但绝对不要：

点击：

“一键生成整集”

然后后台偷偷：

生成剧本
生成分镜
生成图片
生成视频
生成配音
生成音乐
生成音效
生成 Timeline
Render

这种行为暂时不要实现。

原因：

1. AI 成本不可控
2. 中间结果不可审核
3. 某一步失败无法处理
4. 用户无法修改
5. 不符合当前项目：
   Generate → Preview → Apply → Confirm
   原则

本阶段是：

“引导式 Pipeline”。

不是：

“黑盒一键生成”。

==================================================
十一、 Pipeline Action 必须与已有 Engine 对接
==================================================

不要重新实现 AI Engine。

必须复用现有：

ProviderResolver

AiCapability

ProjectAiConfig

GenerationTask

Preview

Apply

Asset

Storage

Timeline

Render

==================================================
十二、 Script Pipeline
==================================================

流程：

Episode Plan
 ↓
Generate Script
 ↓
Preview
 ↓
Apply
 ↓
用户编辑
 ↓
Confirm Script

如果 Script 已存在：

显示：

[查看]
[编辑]
[重新生成]

不要直接禁止。

如果重新生成：

必须提示：

“重新生成剧本可能导致现有分镜失效。”

如果已有 Storyboard：

必须产生合理的：

STALE

或使用当前已有 Stale 机制。

不要自动删除 Storyboard。

==================================================
十三、 Storyboard Pipeline
==================================================

流程：

Confirmed Script
 ↓
Generate Storyboard
 ↓
Preview
 ↓
Apply
 ↓
用户编辑
 ↓
Confirm Storyboard

如果 Script 没有确认：

不能生成正式 Storyboard。

UI 提示：

“请先确认剧本。”

如果 Storyboard 已存在：

显示：

[查看分镜]
[编辑分镜]
[重新生成分镜]

不要因为已经存在 Storyboard 就禁止操作。

==================================================
十四、 Visual Asset Pipeline
==================================================

Storyboard Shot
 ↓
Image Generation
 ↓
Preview
 ↓
Apply
 ↓
Image Asset

然后：

Image Asset
或
Storyboard Shot
 ↓
Video Generation
 ↓
Preview
 ↓
Apply
 ↓
Video Asset

必须保留现有：

GenerationTask

和：

Preview → Apply

原则。

不要生成成功后直接伪造正式 Asset。

==================================================
十五、 Image / Video 生成必须以 Shot 为中心
==================================================

Episode Workspace 中：

视觉素材页面：

Scene 01

Shot 001

[Storyboard Preview]

Image：

未生成

[生成图片]

Video：

未生成

[生成视频]

Shot 002

...

用户必须能够明确知道：

这个图片 / 视频属于：

E01
→ Scene 01
→ Shot 001

而不是一个没有上下文的：

“Image Asset”。

==================================================
十六、 Audio Pipeline
==================================================

音频分为：

Dialogue
Music
SFX

--------------------------------------------

Dialogue：

ScriptBlock
 ↓
TTS
 ↓
Audio Asset

必须能够显示：

Scene
Script Block
角色
文本
Voice
Audio

--------------------------------------------

Music：

Episode
 ↓
Music Generation
 ↓
Music Asset

--------------------------------------------

SFX：

Shot / Scene / Episode
 ↓
SFX
 ↓
Audio Asset

继续使用 Phase 11 / Phase 12 已有 Engine。

不要重新实现 Provider。

==================================================
十七、 Audio 页面
==================================================

Episode Workspace：

音频素材

分成：

对白
音乐
音效

例如：

对白：

林玄：
“这是什么力量？”

[已生成]

艾琳：
“检测到未知能量。”

[未生成]

[生成缺失对白]

音乐：

主音乐

[已生成]

音效：

Shot 001
空间裂缝

[已生成]

Shot 004
能量爆炸

[未生成]

[生成缺失音效]

==================================================
十八、 不要自动无限生成
==================================================

本阶段可以提供：

[生成缺失素材]

但是：

必须生成 Preview / Task。

不能：

直接批量创建正式 Asset。

用户必须可以：

查看
确认
Apply

如果项目已有批量 Generate API：

优先复用。

==================================================
十九、 Asset Readiness
==================================================

Episode Workspace 必须显示：

视觉素材：

3 / 5

对白：

8 / 8

音乐：

1 / 1

音效：

5 / 7

不要只显示：

“素材已完成”。

必须让用户知道：

缺了什么。

==================================================
二十、 Timeline Readiness
==================================================

当用户进入 Timeline：

Timeline Builder 继续按照 Phase 13 逻辑工作。

不要重写 Timeline。

Timeline：

Script
Storyboard
Assets
 ↓
Builder
 ↓
Timeline
 ↓
Preview

如果存在缺失：

显示：

缺失视觉素材：

Shot 003

缺失对白：

ScriptBlock 08

不要自动调用 AI。

==================================================
二十一、 Timeline Lock
==================================================

Render 前必须：

Timeline LOCKED

继续使用 Phase 14：

LOCKED
→ Render

如果：

STALE

禁止 Render。

UI：

“时间线已过期，请重新构建或检查时间线。”

如果：

DRAFT

显示：

[继续编辑]

[锁定时间线]

==================================================
二十二、 Render Pipeline
==================================================

继续使用 Phase 14 已实现架构：

LOCKED Timeline
 ↓
RenderService
 ↓
Immutable Manifest Snapshot
 ↓
RenderJob
 ↓
RenderWorker
 ↓
FFmpeg
 ↓
FFprobe
 ↓
RenderArtifact

不要重新实现 Render Engine。

==================================================
二十三、 Render 前置检查
==================================================

Episode Workspace 点击：

[Render Episode]

必须先检查：

1. Timeline exists
2. Timeline LOCKED
3. Timeline not STALE
4. no missing visual asset
5. no missing required dialogue audio
6. Render prerequisites valid

如果失败：

不要创建 RenderJob。

明确告诉用户：

为什么不能 Render。

例如：

“Shot 003 缺少视频或图片素材。”

“ScriptBlock 08 缺少对白音频。”

“Timeline 尚未锁定。”

==================================================
二十四、 Render 完成后的 Episode
==================================================

Render SUCCEEDED：

Episode Workspace 状态：

COMPLETED

显示：

Episode MP4

播放器。

信息：

Duration
Resolution
FPS
Render Time

按钮：

[再次渲染]

[查看 Render History]

不要创建假 MP4。

继续使用真实：

RenderArtifact。

==================================================
二十五、 Render History
==================================================

Episode Workspace 中显示：

Render History

例如：

Render #3
成功
Timeline v4
2026-08-19

Render #2
失败
Timeline v3

Render #1
成功
Timeline v2

每个 Render：

[查看]
[播放]

FAILED：

[Retry]

继续使用 Phase 14 已有 RenderJob。

==================================================
二十六、 Episode Workspace Overview API
==================================================

检查现有：

GET

/projects/:projectId/episodes/:episodeId/overview

如果已有：

优先扩展。

不要创建重复 API。

返回：

episode
season
productionStage

plan
script
storyboard

visualAssets
audioAssets

timeline
render

示例：

{
  episode,
  season,
  productionStage,

  plan: {
    exists,
    status
  },

  script: {
    exists,
    status,
    version
  },

  storyboard: {
    exists,
    status,
    version,
    sceneCount,
    shotCount
  },

  assets: {
    images: {
      total,
      ready,
      missing
    },

    videos: {
      total,
      ready,
      missing
    },

    voices: {
      total,
      ready,
      missing
    },

    music: {
      total,
      ready,
      missing
    },

    sfx: {
      total,
      ready,
      missing
    }
  },

  timeline: {
    exists,
    status,
    version,
    durationSeconds
  },

  render: {
    latestJob,
    latestArtifact,
    status
  }
}

不要返回：

API Key
encryptedApiKey
Provider secrets
GenerationTask secret fields
internal filesystem paths

==================================================
二十七、 Production Readiness Resolver
==================================================

建议在：

packages/core

新增纯函数：

resolveEpisodeProductionStage()

或：

resolveEpisodeProductionState()

必须：

纯函数
可单测
不访问数据库

输入：

Episode production snapshot

输出：

ProductionStage

以及：

nextAction

例如：

{
  stage: "SCRIPTING",

  nextAction: {
    type: "CONFIRM_SCRIPT",
    label: "确认剧本"
  }
}

或：

{
  stage: "STORYBOARDING",

  nextAction: {
    type: "GENERATE_STORYBOARD",
    label: "生成分镜"
  }
}

不要把复杂判断全部写在 Vue 页面。

==================================================
二十八、 Next Action Resolver
==================================================

建议：

resolveEpisodeNextAction()

统一处理：

当前阶段
下一步
按钮
原因

例如：

PLANNING：

next:
GENERATE_SCRIPT

SCRIPTING：

如果未确认：
CONFIRM_SCRIPT

如果已确认：
GENERATE_STORYBOARD

STORYBOARDING：

CONFIRM_STORYBOARD

VISUAL_ASSETS：

GENERATE_MISSING_VISUAL_ASSETS

AUDIO_ASSETS：

GENERATE_MISSING_AUDIO_ASSETS

COMPOSING：

OPEN_TIMELINE

READY_TO_RENDER：

RENDER_EPISODE

RENDERING：

VIEW_RENDER_JOB

COMPLETED：

VIEW_EPISODE

==================================================
二十九、 不允许把业务逻辑写死在前端
==================================================

Web 不应该：

if (!script)
  ...

然后自己判断整个生产流程。

应该：

API
→ Episode Production Snapshot
→ Core Resolver
→ UI

Mobile / Desktop 也复用相同逻辑。

==================================================
三十、 Web
==================================================

Web 是完整生产端。

Episode Workspace 必须完整支持：

Episode Plan
Script
Storyboard
Visual Assets
Audio Assets
Timeline
Render

==================================================
三十一、 Mobile
==================================================

Mobile：

支持：

查看生产阶段
查看下一步
查看 Script
查看 Storyboard
查看 Assets
查看 Render
播放 Timeline Preview

复杂编辑：

继续提示：

“请在 Web 端编辑。”

不要实现复杂 NLE。

==================================================
三十二、 Desktop
==================================================

Desktop：

复用：

Episode Workspace
Timeline
Render

不实现：

专业 NLE。

==================================================
三十三、 全局导航
==================================================

Phase 14.5 / 14.6 已经确定：

不要把：

Images
Videos
Voices
Music
SFX

当成主要生产导航。

Episode Workspace 是主入口。

Project-level Asset Library 如果已经存在：

保留。

但主生产流程必须：

Episode
→ Assets

==================================================
三十四、 AI Provider
==================================================

本阶段：

不新增 Provider。

继续：

ProjectAiConfig
→ ProviderResolver
→ Capability
→ Provider

不要写死：

DeepSeek

不要让 Web 直接调用 AI。

==================================================
三十五、 GenerationTask
==================================================

继续使用现有：

GenerationTask

不要为：

EpisodeProduction

重新设计另一套 AI Task 系统。

所有 AI 生成仍然：

Generate
→ GenerationTask
→ Preview
→ Apply

==================================================
三十六、 错误处理
==================================================

Pipeline 中任何一步失败：

不能导致整个 Episode 状态变成：

“失败”。

例如：

Image Generation Failed

应该显示：

视觉素材：

4 / 5

Shot 003：

Generation Failed

[Retry]

其他素材继续可用。

不要：

一个 AI Task 失败
→ 整集失败。

==================================================
三十七、 Partial Production
==================================================

允许：

部分完成。

例如：

E01：

Script ✓
Storyboard ✓
Images 4/5
Videos 2/5
Voice 8/8
Music 1/1
SFX 5/7

用户仍然可以：

继续生成缺失素材。

但是：

Timeline / Render 必须按照各自真实约束判断。

==================================================
三十八、 Stale Propagation
==================================================

必须检查现有 Stale 系统。

至少保证：

Script 修改
→ Storyboard STALE

Storyboard 修改
→ Timeline STALE

Asset 变化：

如果影响 Timeline：

Timeline STALE

Timeline STALE：

不能 Render。

不要自动重新生成。

只提示：

“上游内容发生变化，当前时间线需要重新检查。”

==================================================
三十九、 Production Activity
==================================================

如果当前项目已有：

GenerationTask

不要新增复杂 Event 系统。

Episode Workspace 可以展示：

最近活动：

剧本生成完成
分镜生成完成
Shot 003 图片生成失败
TTS 生成完成
Timeline 已锁定
Render 成功

如果实现成本很低：

优先通过已有 GenerationTask / RenderJob 计算。

不要为了 UI 创建大量重复事件表。

==================================================
四十、 用户不能迷路
==================================================

Episode Workspace 任意页面：

必须显示：

项目
/
Season
/
Episode
/
当前阶段

例如：

星河碰撞
/
第一季
/
E01 · 星门初现
/
分镜

并且：

[← 返回 E01]

==================================================
四十一、 不要新增“流程大师”式复杂 UI
==================================================

不要：

巨大的流程图占满屏幕。

不要：

几十个步骤。

用户真正需要：

当前步骤
下一步
完成度
缺失内容

保持简单。

==================================================
四十二、 不要自动调用 AI
==================================================

特别重要：

打开 Episode Workspace

不能自动：

生成 Script
生成 Storyboard
生成 Image
生成 Video
生成 TTS
生成 Music
生成 SFX
Build Timeline
Render

只有用户明确点击：

[生成剧本]
[生成分镜]
[生成图片]
[生成视频]
[生成配音]
[生成音乐]
[生成音效]
[构建时间线]
[Render Episode]

才允许执行。

==================================================
四十三、 AI 成本必须透明
==================================================

每个 AI 操作：

如果当前系统已经支持：

Provider / Model

可以显示：

使用模型：

DeepSeek

或者：

当前 Project Provider

不要让用户误以为：

整个流程免费。

不要偷偷重复生成。

==================================================
四十四、 Episode Production Checklist
==================================================

Episode Workspace 提供：

Production Checklist

例如：

☑ Episode Plan
☑ Script
☑ Storyboard
☑ Visual Assets
☑ Dialogue
☑ Music
☑ SFX
☑ Timeline
☑ Timeline Locked
☑ Render

但：

Checklist 必须根据真实数据。

不要简单手动 checkbox。

==================================================
四十五、 不要改变数据库核心结构
==================================================

优先：

Core Resolver
API Snapshot
UI Workflow

如果不需要数据库：

不要新增数据库 Model。

如果确实需要：

必须先检查现有模型。

不要为了：

productionStage

新增一个冗余 Episode 字段。

优先计算。

==================================================
四十六、 Seed
==================================================

本阶段必须检查现有 Seed：

World
Character
Season
Script
Storyboard
Image
Video
TTS
Music
SFX
Timeline
Render

不要创建重复 Seed。

如果项目当前没有完整 E01 Demo：

可以新增：

seed-episode-production-demo

但必须：

检查依赖
不重复创建
不 reset
不删除已有数据

Seed 只能用于：

开发验证。

不要创建假 RenderArtifact。

如果没有 FFmpeg：

明确：

REAL RENDER NOT VERIFIED。

==================================================
四十七、 Real E2E Verification
==================================================

本阶段必须真正尝试：

Project
→ Season
→ E01
→ Episode Workspace

然后验证：

Episode Plan
→ Script
→ Storyboard
→ Assets
→ Timeline
→ LOCK
→ Render

注意：

如果外部 Provider / API Key / FFmpeg 未配置：

不要伪造成功。

必须报告：

哪一步真实验证成功。

哪一步因为环境缺失无法验证。

==================================================
四十八、 FFmpeg
==================================================

Phase 14 已实现真实 FFmpeg Render。

本阶段：

不要重新实现 FFmpeg。

只验证：

Render Pipeline 能从 Episode Workspace 正确进入。

如果：

FFmpeg NOT_FOUND

不要安装系统软件。

不要伪造 MP4。

报告：

FFmpeg 未安装。

==================================================
四十九、 API Client
==================================================

继续：

packages/api-client

Web：

禁止直接 fetch。

Mobile：

禁止直接调用 AI Provider。

Desktop：

继续使用现有 API Client / API 层。

==================================================
五十、 Tests
==================================================

至少新增：

1. resolveEpisodeProductionStage

2. resolveEpisodeNextAction

3. Empty Episode → PLANNING

4. Episode Plan Ready → SCRIPTING

5. Script Draft → CONFIRM_SCRIPT

6. Script Confirmed → STORYBOARDING

7. Storyboard Confirmed → VISUAL_ASSETS

8. Partial Visual Assets → VISUAL_ASSETS

9. Visual Complete → AUDIO_ASSETS

10. Partial Audio → AUDIO_ASSETS

11. Timeline Draft → COMPOSING

12. Timeline Locked → READY_TO_RENDER

13. Render Queued → RENDERING

14. Render Success → COMPLETED

15. Render Failure → Retry

16. Script modification → Storyboard stale

17. Storyboard modification → Timeline stale

18. Timeline stale → Render blocked

19. Missing visual → Render blocked

20. Missing required dialogue → Render blocked

21. Episode Workspace Overview API

22. Project isolation

23. Episode ownership

24. API Key not leaked

25. GenerationTask secrets not leaked

26. No direct AI call from Web

==================================================
五十一、 UI 手动验收
==================================================

必须真实检查：

--------------------------------------------

1.

进入 Project。

--------------------------------------------

2.

进入 Season 1。

--------------------------------------------

3.

点击 E01。

必须进入：

Episode Workspace。

--------------------------------------------

4.

Episode Workspace：

能够看到：

当前阶段
生产进度
下一步

--------------------------------------------

5.

进入 Script。

能够：

查看
编辑
确认

--------------------------------------------

6.

确认 Script。

UI 必须告诉：

下一步：

生成分镜。

--------------------------------------------

7.

进入 Storyboard。

能够：

查看 Scene
查看 Shot

--------------------------------------------

8.

确认 Storyboard。

下一步：

生成视觉素材。

--------------------------------------------

9.

进入 Assets。

看到：

Images
Videos
Voices
Music
SFX

并且全部限定为当前 Episode。

--------------------------------------------

10.

存在缺失素材：

UI 明确：

4 / 5

而不是：

“素材处理中”。

--------------------------------------------

11.

进入 Timeline。

能够：

Preview

--------------------------------------------

12.

Timeline LOCKED。

显示：

Render Episode

--------------------------------------------

13.

点击 Render。

如果 FFmpeg 配置正确：

真实生成 MP4。

如果 FFmpeg 未配置：

明确失败原因。

--------------------------------------------

14.

Render 成功：

Episode Workspace 显示：

COMPLETED

并能够播放：

Episode.mp4

==================================================
五十二、 Performance
==================================================

Episode Overview 不允许：

一次加载整个 Project 的所有 Asset。

只加载：

当前 Episode。

避免：

N+1 API。

如果需要：

优先增加聚合 API。

==================================================
五十三、 安全
==================================================

绝对不能泄漏：

AI_API_KEY
encryptedApiKey
Provider Secret
filesystem internal path
Render worker internal workspace
GenerationTask secret metadata

继续使用：

API Server

作为唯一 Provider / Render 控制边界。

==================================================
五十四、 不破坏已有功能
==================================================

绝对禁止：

重建 Monorepo

更换技术栈

删除历史 migration

修改历史 migration

prisma migrate reset

DROP TABLE

删除用户数据

删除 GenerationTask

删除 Asset

删除 Timeline

删除 RenderJob

删除 RenderArtifact

不要重新实现：

AI Provider
Timeline
FFmpeg
Render Worker

==================================================
五十五、 不实现的内容
==================================================

本阶段明确不实现：

字幕
Caption
Advanced Audio Mastering
Ducking
EQ
Fade
Transitions
Redis
BullMQ
Cloud Render
GPU Render
AI Agent
AI 自动剪辑
AI 自动补镜
AI 自动补声
Batch Production
Publishing
YouTube
TikTok
Bilibili
Instagram
Billing
Login
JWT
Subscription
Team Collaboration

这些留到后续 Phase。

==================================================
五十六、 实现顺序
==================================================

严格：

Phase A
Current Architecture Audit

Phase B
Production Stage Resolver

Phase C
Next Action Resolver

Phase D
Episode Overview API

Phase E
Episode Workspace Production Dashboard

Phase F
Script Integration

Phase G
Storyboard Integration

Phase H
Visual Asset Integration

Phase I
Audio Asset Integration

Phase J
Timeline Integration

Phase K
Render Integration

Phase L
Partial Production / Missing Asset UX

Phase M
Stale Propagation

Phase N
Tests

Phase O
E2E Verification

Phase P
lint

Phase Q
typecheck

Phase R
prisma validate

Phase S
build

==================================================
五十七、 当前代码扫描要求
==================================================

开始编码前必须先检查：

apps/web

apps/api

packages/core

packages/types

packages/api-client

packages/config

prisma/schema.prisma

并搜索：

Episode
Season
Script
Storyboard
Asset
GenerationTask
Timeline
RenderJob
RenderArtifact

重点检查：

已有 Episode Workspace

已有 Production UI

已有 Overview API

已有 Stage / Status

已有 Stale

已有 Generation APIs

已有 Timeline APIs

已有 Render APIs

不要重复创建。

==================================================
五十八、 API 路径原则
==================================================

Episode production API 必须保持：

/projects/:projectId/episodes/:episodeId/...

例如：

/overview
/script
/storyboard
/assets
/timeline
/render

不要创建：

/production/episode/:id

这种脱离现有资源结构的新体系。

==================================================
五十九、 Core
==================================================

建议：

packages/core/src/episode-production.ts

提供纯函数：

resolveEpisodeProductionStage()

resolveEpisodeNextAction()

resolveEpisodeProductionProgress()

resolveEpisodeReadiness()

必须：

无数据库
无 HTTP
无 Provider
无 Prisma

全部可单元测试。

==================================================
六十、 Production Progress
==================================================

建议 UI 显示：

例如：

4 / 7 stages completed

但不要伪造百分比。

推荐：

Planning
Script
Storyboard
Visual
Audio
Timeline
Render

按照真实完成状态计算。

==================================================
六十一、 Stage 与 Engine 不等价
==================================================

不要：

Script Engine = Script Stage

必须考虑：

Script 已存在但未确认。

所以：

Engine Status
≠
Production Stage

Production Stage 必须反映：

“用户现在应该做什么。”

==================================================
六十二、 用户行为优先
==================================================

任何一个 Episode Workspace 页面：

用户必须能回答：

1. 我在哪里？
2. 我在制作哪一集？
3. 当前阶段是什么？
4. 已完成什么？
5. 缺什么？
6. 下一步是什么？
7. 点击哪个按钮？

如果不能回答：

继续调整 UI。

==================================================
六十三、 最终产品流程
==================================================

最终用户应该体验：

创建项目

↓

AI 世界观

↓

AI 人物

↓

AI 故事圣经

↓

创建 Season

↓

AI 拆解 Episodes

↓

进入 E01

↓

Episode Plan

↓

生成 Script

↓

确认 Script

↓

生成 Storyboard

↓

确认 Storyboard

↓

生成 Image

↓

生成 Video

↓

生成 Voice

↓

生成 Music

↓

生成 SFX

↓

Timeline

↓

Preview

↓

LOCK

↓

Render

↓

Episode.mp4

用户始终知道：

“下一步是什么。”

==================================================
六十四、 重要：不要跳过人工确认
==================================================

默认：

AI Generate

↓

Preview

↓

Apply

↓

Edit

↓

Confirm

↓

Next Stage

不要默认：

AI Generate

↓

自动进入下一阶段。

==================================================
六十五、 最终完成标准
==================================================

必须实现：

[ ] Production Stage Resolver
[ ] Next Action Resolver
[ ] Episode Overview API
[ ] Episode Production Dashboard
[ ] Script Pipeline Integration
[ ] Storyboard Pipeline Integration
[ ] Visual Asset Pipeline Integration
[ ] Audio Asset Pipeline Integration
[ ] Timeline Integration
[ ] Render Integration
[ ] Partial Production
[ ] Missing Asset UX
[ ] Stale UX
[ ] Render Readiness
[ ] Production Progress
[ ] Mobile Read-only Workflow
[ ] Desktop Workflow
[ ] Tests
[ ] E2E Verification
[ ] lint
[ ] typecheck
[ ] prisma validate
[ ] build

==================================================
六十六、 最终报告
==================================================

完成后严格输出：

# Phase 15 Complete

## 1. Production Architecture

说明：

Episode
→ Episode Plan
→ Script
→ Storyboard
→ Assets
→ Timeline
→ Render

## 2. Production Stage

列出所有 Stage。

## 3. Next Action Resolver

说明如何判断下一步。

## 4. Episode Workspace

说明 UI。

## 5. Script Integration

## 6. Storyboard Integration

## 7. Visual Assets

## 8. Audio Assets

## 9. Timeline

## 10. Render

## 11. Partial Production

## 12. Missing Asset

## 13. Stale

## 14. API

列出新增 / 修改 API。

## 15. Core

列出新增纯函数。

## 16. Tests

列出测试数量。

## 17. lint

## 18. typecheck

## 19. prisma validate

## 20. build

## 21. E2E Verification

必须明确：

World
→ Character
→ Season
→ Episode
→ Script
→ Storyboard
→ Assets
→ Timeline
→ Render

哪些真实走通。

哪些因为环境 / Provider / FFmpeg 未配置无法走通。

绝对不能伪造成功。

## 22. Real MP4 Verification

明确：

FFmpeg:
FOUND / NOT_FOUND

FFprobe:
FOUND / NOT_FOUND

Render:
VERIFIED / NOT VERIFIED

MP4:
FOUND / NOT FOUND

## 23. Remaining Limitations

只写真实存在的问题。

## 24. Explicitly NOT Implemented

## 25. Recommended Next Phase

==================================================
六十七、 最终停止位置
==================================================

本阶段做到：

Episode
↓
Production Workspace
↓
Episode Plan
↓
Script
↓
Storyboard
↓
Visual Assets
↓
Audio Assets
↓
Timeline
↓
LOCK
↓
Render
↓
MP4

停止。

不要进入：

Subtitle
Advanced Audio
AI Agent
Batch Production
Cloud Render
Publishing

现在开始执行：

# Phase 15 — Episode Production Pipeline & E2E Orchestration v1

先扫描实际代码。

不要假设。

不要 reset。

不要破坏已有数据。

不要重复实现已有 Engine。

必须在现有架构上增量完成。

完成后进行真实 E2E 验收。

现在开始执行。