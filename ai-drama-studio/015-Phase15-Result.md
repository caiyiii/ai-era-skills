Phase 15 Complete
Episode Workspace 已成为引导式生产控制中心。阶段与下一步由 Core 纯函数根据真实数据计算，不写死在 Vue，也没有一键黑盒生成。

1. Production Architecture
Project → Season → Episode
  → Episode Production Workspace
  → Episode Plan → Script → Storyboard
  → Visual / Audio Assets
  → Timeline → LOCK → Render → Episode MP4
用户始终在「制作一集」，而不是在各个 Engine 之间迷路。已有 Provider / GenerationTask / Preview→Apply / Timeline / FFmpeg Render 全部复用，没有重写。

2. Production Stage
计算结果，不是用户手选，也不是新的数据库字段：

Stage	含义
PLANNING
还没有可用的剧集规划
SCRIPTING
规划已就绪，剧本未确认
STORYBOARDING
剧本已确认，分镜未确认
VISUAL_ASSETS
分镜已确认，视觉素材未齐
AUDIO_ASSETS
视觉已齐，对白/音乐未齐
COMPOSING
素材可合成，时间线未锁定
READY_TO_RENDER
时间线已锁定且未过期
RENDERING
RenderJob 进行中
COMPLETED
Render 成功，可播放成片
允许部分完成。某个 Shot 生成失败不会把整集标成失败。

3. Next Action Resolver
resolveEpisodeNextAction() 根据当前 Stage + 实体状态给出唯一主按钮：

无规划 → EDIT_PLAN
有规划无剧本 → GENERATE_SCRIPT
剧本未确认 → CONFIRM_SCRIPT
剧本已确认 → GENERATE_STORYBOARD
分镜未确认 → CONFIRM_STORYBOARD
视觉缺失 → GENERATE_MISSING_VISUAL_ASSETS
音频缺失 → GENERATE_MISSING_AUDIO_ASSETS
时间线未建/过期 → OPEN_TIMELINE
时间线草稿 → LOCK_TIMELINE
已锁定 → RENDER_EPISODE（失败则为 RETRY_RENDER）
渲染中 → VIEW_RENDER_JOB
成功 → VIEW_EPISODE
4. Episode Workspace
进入 /projects/:projectId/episodes/:episodeId（季内路由会落到同一工作台）：

状态：制作中 / 已完成
当前阶段
Production Steps（✓ / ● / ○）
Current Task + 下一步按钮
Asset Readiness：4 / 5 这种计数，而不是「处理中」
Production Checklist（由真实数据计算）
缺失 Shot / ScriptBlock
Stale 提示
Render History + Episode MP4 播放器
最近活动来自 GenerationTask / RenderJob
打开页面不会自动调用 AI。

5. Script Integration
规划 → 生成 Preview → Apply → 编辑 → 确认剧本 → 生成分镜。

已有剧本仍可查看 / 编辑 / 重新生成。重新生成会提示：可能导致现有分镜 STALE，不会自动删除分镜。

6. Storyboard Integration
未确认剧本不能生成正式分镜，UI 提示「请先确认剧本」。确认分镜后下一步是生成视觉素材。重新生成分镜会提示 Timeline 将标记为 STALE。

7. Visual Assets
按 Scene / Shot 列出 Image / Video 是否已生成。缺失走现有 Shot 生成（Preview → Apply），不伪造正式 Asset。

8. Audio Assets
对白按 ScriptBlock（角色 + 文本 + 已生成/未生成），音乐 / 音效按本集绑定。提供「生成缺失对白 / 视觉 / 音效」入口，仍然进入已有生成流程，不批量 Apply 正式 Asset。

9. Timeline
继续使用 Phase 13 Builder。缺失会列出 Shot 003、ScriptBlock 08。STALE 显示「上游内容发生变化，当前时间线需要重新检查。」DRAFT 可继续编辑并锁定；LOCKED 才出现 Render Episode。不会自动调 AI。

10. Render
继续使用 Phase 14：LOCKED → RenderService → Manifest Snapshot → RenderJob → Worker → FFmpeg。

Workspace / Render 页点击 Render 前会检查：时间线存在、LOCKED、非 STALE、视觉齐、必要对白齐。失败不创建 Job，并给出具体原因。

11. Partial Production
视觉 4/5、对白 8/8、音乐 1/1、音效 5/7 都可以同时存在。用户可继续补缺失。Timeline / Render 按各自约束判断。

12. Missing Asset
Overview 与素材页显示计数和具体缺失项。Render 被阻塞时会指出哪个 Shot / ScriptBlock。

13. Stale
Script 变更 → Storyboard STALE
Storyboard 变更 → Timeline STALE
Timeline STALE → 禁止 Render
只提示，不自动重新生成。

14. API
扩展已有路径，没有另起 /production/episode/:id：

GET /projects/:projectId/episodes/:episodeId/overview（新增项目级）
GET /projects/:projectId/seasons/:seasonId/episodes/:episodeId/overview（继续可用）
返回 episode / season / productionStage / nextAction / plan / script / storyboard / assets(total,ready,missing) / timeline / render / progress / checklist / readiness / activity。

不返回 API Key、encryptedApiKey、内部路径、GenerationTask 密钥字段。

15. Core
packages/core/src/episode-production.ts（无数据库 / 无 HTTP / 无 Prisma）：

resolveEpisodeProductionStage()
resolveEpisodeNextAction()
resolveEpisodeProductionProgress()
resolveEpisodeReadiness()
resolveEpisodeProductionChecklist()
resolveEpisodeNextActionRoute()
getEpisodeProductionStageLabel()
16. Tests
API 全量：463 passed，1 skipped。

本阶段覆盖包括：Empty→PLANNING、Plan→SCRIPTING、Draft→CONFIRM_SCRIPT、Confirmed→STORYBOARDING、分镜确认→VISUAL、部分视觉/音频、Timeline Draft/Locked、Render Queued/Success/Failure、Stale 阻断 Render、缺失视觉/对白阻断 Render、Overview 项目隔离、密钥不泄漏。

Web 页面无直接 fetch，不直连 Provider。

17. lint
通过（pnpm lint）。

18. typecheck
通过（pnpm typecheck，9 个包）。

19. prisma validate
通过。schema 有效。未 migrate reset，未改历史 migration，未新增 productionStage 字段。

20. build
通过（pnpm build，9 个包，含 web / api / mobile / desktop）。

21. E2E Verification
对现有 Demo 项目「星河碰撞」做了真实 API 探测，没有伪造成功：

步骤	结果
World / Character / Season / Episode
已存在。E01 demo-ep-01 标题「星系碰撞」，有 synopsis + outline
Episode Workspace Web
http://localhost:3010/projects/demo-xinghe/episodes/demo-ep-01 → HTTP 200
Overview API
productionStage=SCRIPTING，下一步 GENERATE_SCRIPT，与「有规划、无剧本」一致
项目隔离
其他 projectId 访问该 overview → 400
密钥泄漏
Overview JSON 未泄漏 encryptedApiKey / apiKey
Script
当前库中 E01 没有 Script（404）
Storyboard
没有（404）
Assets
因无分镜/剧本，计数为 0
Timeline
没有（404）
Render Job
[]
真实 AI Generate Script→…→Render
未走通。打开 Workspace 不会自动生成；本环境也未配置可用 Provider 去跑完整生成链
结论：控制中心、阶段计算、Overview、隔离与安全已在真实数据上验证。完整「生成剧本 → 确认 → 分镜 → 素材 → 锁定 → 成片」取决于后续人工点击 + Provider + FFmpeg，当前环境不具备后两步。

22. Real MP4 Verification
FFmpeg: NOT_FOUND
FFprobe: NOT_FOUND
Render: NOT VERIFIED
MP4: NOT FOUND
没有安装系统软件，没有伪造 MP4。

23. Remaining Limitations
当前 Demo 库里的 E01 只有规划，没有 Script / Storyboard / Timeline / Render 记录，所以无法在本机把成片链路跑完。
音效 expected 数量没有从分镜 cue 推导；无音效需求时不阻塞音频阶段（避免假 5/7）。
本机 API watch 进程曾在编译中退出；Overview 仍能打到一个仍在监听 3011 的实例，但不保证它会随源码热更新。
外部 AI Provider / API Key 未在本阶段做真实生成验收。
24. Explicitly NOT Implemented
字幕、Caption、Ducking/EQ/Fade/转场、Redis/BullMQ、Cloud/GPU Render、AI Agent、自动剪辑/补镜/补声、Batch Production、YouTube/TikTok/Bilibili/Instagram、Billing、Login/JWT、Subscription、协作、一键生成整集。

25. Recommended Next Phase
在 FFmpeg 与 Provider 配好后，用现有 E01 Workspace 人工走完 Generate → Preview → Apply → Confirm → LOCK → Render，做一次真实 MP4 验收。再考虑字幕或发布，而不是继续加新 Engine。

API_OK 200
API 热重启失败了：3011 已被占用（EADDRINUSE），Nest 随后去杀一个已经不存在的进程，watch 退出。

当前 http://localhost:3011 仍在正常响应（/projects 返回 200），本地开发可以继续用这个已在跑的实例。若要干净地重新 watch，先结束占用 3011 的旧进程，再执行 pnpm --filter @ai-drama-studio/api dev。