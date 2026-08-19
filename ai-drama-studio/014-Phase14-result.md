Phase 14 Complete
1. Database
新增 Model：

RenderJob — 渲染任务（status、manifestSnapshot、progress、currentStage、timelineVersion 等）
RenderJobEvent — 生命周期事件
RenderArtifact — 最终 MP4 成片
新增 Enum：

RenderJobStatus / RenderJobStage / RenderJobEventType / RenderArtifactType
Migration：

20260818140000_render_engine（已 migrate deploy，未 reset、未改历史 migration）
2. Render Architecture
LOCKED Timeline
  → RenderService.create()
  → CompositionService → RenderManifestSnapshot（深拷贝 + storageKey + missing）
  → RenderJob (QUEUED)
  → RenderWorkerService（本地轮询，concurrency=1，atomic claim）
  → LocalFfmpegRenderEngine
  → FFmpegService (spawn argv[])
  → FFprobe 校验
  → RenderArtifact (storage/renders/...)
Worker 只消费 snapshot，不重新读 Timeline / Storyboard / Script。

3. RenderJob
创建前校验：LOCKED、非 STALE、无缺失视觉/对白素材
同 projectId+episodeId+timelineId+timelineVersion 进行中任务幂等返回
FAILED 可 retry，复用原 snapshot
状态机：QUEUED → PREPARING → RENDERING → SUCCEEDED/FAILED/CANCELLED
4. Manifest Snapshot
RenderManifestService 基于 CompositionService.compose() + missingFromMetadata + Asset storageKey
stripSecretFields 去除敏感字段
Retry 不重读当前 Timeline
5. Render Worker
RenderWorkerService：OnModuleInit 轮询 QUEUED，updateMany claim
独立 workspace：tmp/render/<jobId>/{inputs,work,output,logs}
写入 inputs.json 供调试（不含密钥）
VITEST / RENDER_WORKER_DISABLED=true 时不启动轮询
6. FFmpeg
FFmpegService：spawn(ffmpegPath, args)，解析 -progress pipe:1
FFmpegCommandBuilder：可单测的 argv[] 构建（libx264 + AAC + filter_complex）
resolveFfmpegPath / resolveFfprobePath（FFMPEG_PATH / FFPROBE_PATH / PATH）
找不到 FFmpeg → FFMPEG_NOT_FOUND，不伪造成功
7. Video Composition
黑底 + scale/pad + overlay（按 zIndex；同层 IMAGE 在下、VIDEO 在上）
Image → loop 视频段；Video 直接 trim/scale
输出分辨率/fps 以 Manifest 为准
8. Audio Composition
Dialogue / Music / SFX：atrim + adelay + volume + amix
effectiveVolume = track.volume × clip.volume；muted/disabled 跳过
无音频时 -an 仍可出纯视频 MP4
9. Progress
QUEUED 0% / PREPARING 5% / RENDERING 5–95%（解析 out_time）/ FINALIZING 95–100%
无法解析时 progress = null，UI 显示 “Rendering…”
10. Cancellation
QUEUED → 直接 CANCELLED
PREPARING/RENDERING → CANCEL_REQUESTED + 真实 kill FFmpeg 进程
11. Retry
仅 FAILED；创建新 Job，复用原 manifestSnapshot + timelineVersion
12. Artifact
RenderArtifact + storageKey: renders/<projectId>/<episodeId>/<renderJobId>/episode.mp4
扩展 AssetStorageService.saveFromFile
安全访问：GET /projects/:projectId/render-artifacts/:artifactId/file
13. API
Method	Path
POST
/projects/:projectId/episodes/:episodeId/render
GET
/projects/:projectId/render-jobs
GET
/projects/:projectId/render-jobs/:renderJobId
POST
/projects/:projectId/render-jobs/:renderJobId/cancel
POST
/projects/:projectId/render-jobs/:renderJobId/retry
GET
/projects/:projectId/render-jobs/:renderJobId/artifact
GET
/projects/:projectId/render-artifacts/:artifactId
GET
/projects/:projectId/render-artifacts/:artifactId/file
14. Web
/projects/:id/episodes/:episodeId/render — Render 控制台（进度、历史、播放/下载）
Timeline 页：LOCKED 显示 “Render Episode”，否则 “请先锁定时间线”
/projects/:id/render — 剧集 Render 入口列表
15. Mobile
RenderPage.vue — 只读查看 Render 状态/历史/成片播放
Episode 详情页增加 “查看成片 Render” 入口
16. Desktop
RenderPanel.vue — 查看 Job/进度/播放 MP4；Render/Cancel/Retry 基础控制
17. Seed
pnpm db:seed:render-demo — 检查上游数据，将 E01 Timeline 设为 LOCKED
不自动执行；不造假 MP4 / 假 SUCCEEDED
18. Tests
69 files / 428 passed / 1 skipped（FFmpeg 集成测试在无 FFmpeg 时 skip）
覆盖：core 状态机、command builder、RenderService、api-client、storage、FFMPEG_NOT_FOUND
19. lint
PASS

20. typecheck
PASS（全 monorepo）

21. prisma validate
PASS

22. build
PASS（turbo build 9 packages）

23. API health
PASS — {"status":"ok"}（已启动带 Render 路由的新 API 实例验证）

若你本地已有旧 API 进程，请手动重启 API 以加载 Render 路由。

24. Existing Data
实体	数量
Project
1
World
1
Character
3
Season
1
Episode
3
Provider
1
GenerationTask
3
Script / Storyboard / Asset / Timeline
0
RenderJob / RenderArtifact
0
未 reset、未删除历史数据。

25. Existing Provider
未新增 AI Provider；Render 不产生 AI 成本。

26. Existing GenerationTask
保留 3 条，未改动。

27. Real Render Verification
FFmpeg:     NOT_FOUND
FFprobe:    NOT_FOUND
RenderJob:  （无，上游 Timeline Demo 未 seed）
Timeline:   （无 LOCKED Timeline）
Output:     N/A
MP4:        NOT_FOUND
REAL RENDER NOT VERIFIED
原因：

本机 PATH 无 FFmpeg/FFprobe（pnpm render:check 已明确报错）
数据库无 Script/Storyboard/Asset/Timeline Demo 数据
已完成的真实引擎路径：

代码层 spawn + FFprobe 校验完整实现
单元/集成测试：无 FFmpeg 时返回 FFMPEG_NOT_FOUND；有 FFmpeg 时会真实出片（当前 skip）
若要完整 Demo：

pnpm db:seed:timeline-demo   # 及上游 seed
pnpm db:seed:render-demo
# 安装 FFmpeg 并配置 FFMPEG_PATH
pnpm render:check
# 重启 API 后
pnpm render:episode -- --project demo-xinghe --episode demo-ep-01
28. Remaining Limitations
本地 worker concurrency=1
无 fade/ducking/转场/高级混音
无分布式队列（Redis/BullMQ）
Windows 上 prisma generate 若 API 占用 DLL 可能 EPERM — 需手动重启 API 后再 generate
Mobile 成片 URL 与 Timeline 预览相同（相对路径，需 API base 配置）
29. Explicitly NOT Implemented
Redis、BullMQ、Cloud/GPU Render、新 AI Provider、自动补素材、字幕、高级混音、平台发布、Billing、Login/JWT、假 MP4/假 Worker/假进度

30. Recommended Next Phase
Phase 15 — Subtitle / Caption Engine（在 LOCKED Timeline → Render MP4 之上叠加字幕轨）

关键新增文件（摘要）
packages/core/src/render.ts
apps/api/src/modules/render/*（Service / Worker / FFmpeg / Controller / Module）
apps/web/pages/projects/[id]/episodes/[episodeId]/render.vue
prisma/migrations/20260818140000_render_engine/
prisma/seed-render-demo.ts
scripts/render-check.ts / scripts/render-episode.ts
开发者命令
pnpm render:check
pnpm db:seed:render-demo
pnpm render:episode -- --project <id> --episode <id>
Phase 14 实现已完成；在本机安装 FFmpeg 并完成上游 Timeline seed 后，即可跑通 LOCKED → Render → 真实 MP4。