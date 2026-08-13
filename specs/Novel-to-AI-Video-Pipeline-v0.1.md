---
type: harness-spec
status: active
created: 2026-07-17
updated: 2026-07-17
---

# 小说到 AI 视频闭环 v0.1

## 目标

在不改写小说正典、不把模型候选误标为正式成片的前提下，建立一条可恢复、可审计、可人工放行的控制链：

`小说快照 → 导演包 → 图像资产清单 → 视频镜头清单 → Remotion 装配回执 → QC → 人工放行`

本版本只接通合同、状态、哈希、门禁和回执，不宣称已经接入任何云端生图或生视频模型。

## 权威边界

- 小说工作区仍是正文、人物、设定、时间线和伏笔的唯一事实源。
- 管线只读取明确指定的正典或已获批候选，并把原始字节快照到任务目录；源文件始终只读。
- 导演包、图像、视频和装配结果都是派生候选，不能反向修改小说真相。
- 每个图像和视频资产必须记录模型、版本、参数、参考资产、许可或授权、路径和 SHA-256。
- 正式母版固定由 Remotion 装配；生成模型只提供候选镜头。发布仍是独立人工确认动作。

## 公共接口

入口：`runner/novel_video_pipeline.ps1`。

### 初始化

```powershell
pwsh -File .\10-项目\Hermes-Harness\runner\novel_video_pipeline.ps1 `
  -Action Initialize `
  -SourcePath "C:\path\chapter.md" `
  -SourceAuthority approved_candidate `
  -Title "第六章 60 秒竖屏改编" `
  -TargetDurationSeconds 60 `
  -AsJson
```

初始化先拒绝疑似密码、Key、Cookie 或令牌，再创建不可变源快照、五个阶段产物骨架、合同和 JSONL Ledger。不会启动模型。

### 检查与推进

```powershell
pwsh -File .\10-项目\Hermes-Harness\runner\novel_video_pipeline.ps1 `
  -Action Inspect -RunPath "<任务目录>" -AsJson

pwsh -File .\10-项目\Hermes-Harness\runner\novel_video_pipeline.ps1 `
  -Action Approve -RunPath "<任务目录>" -Gate adaptation_scope -AsJson

pwsh -File .\10-项目\Hermes-Harness\runner\novel_video_pipeline.ps1 `
  -Action Advance -RunPath "<任务目录>" -AsJson
```

使用自定义 `-RuntimeRoot` 初始化时，后续 `Inspect`、`Approve` 和 `Advance` 必须显式传入同一个根；Runner 拒绝根目录外的合同和任务目录名/`pipeline_id`失配。

`Advance` 只检查当前阶段。产物未就绪时等待 Worker；产物声明 `ready` 后进行 Schema、上游哈希、资产文件、资产哈希和人工复核状态检查。首次失败生成 `repair_request.json`，第二次失败终止任务。

## 阶段合同

| 阶段 | 产物 | 关键验证 | 人工门 |
| --- | --- | --- | --- |
| source_lock | `source/source_snapshot.*` | 与合同 SHA-256 一致 | `adaptation_scope` |
| director | `director_packet.json` | 来源哈希、场景/镜头、连续性、无未决问题 | `director_packet` |
| image_design | `image_manifest.json` | 导演包哈希、镜头映射、模型/版本/完整参数/seed/来源/许可、文件哈希 | `image_assets` |
| video_generation | `video_manifest.json` | 图像清单哈希、镜头与资产ID映射、完整参数、参考路径/哈希、许可、输出哈希 | `video_clips` |
| assembly | `assembly_receipt.json` | 视频清单哈希、Remotion、母版路径/哈希 | 无；必须继续进入 QC |
| qc | `qc_report.json` | 装配回执哈希、全部检查通过、无阻塞项 | `final_master` |

## Loop

每个阶段最多两次验证，只允许一次修正：

`Execute → Verify → Repair once → Verify → Waiting approval / Failed`

人工批准只批准一个具体阶段产物，并把批准记录绑定到该阶段清单与实际资产字节的组合 SHA-256。批准后文件漂移会使当前批准失效；上游已批准产物漂移会阻断下游。批准不等于发布，不修改长期规则，也不自动把一次经验晋级为稳定知识。

## 非目标

- 不把整个小说工作区复制到 Harness。
- 不自动生成或提交小说正文。
- 不自动调用收费模型、下载权重或安装节点。
- 不自动发布、付款、改账号或替换现有体育成片。
- 不在本版本移动或重命名 `<user-home>\Documents\<video-workspace>`。

## 研究依据

- [[30-资源/AI视频/小说到AI视频闭环_第一方资料基线_2026-07-17]]

## 验证资产

- [[10-项目/Hermes-Harness/tests/fixtures/novel_source|最小小说源测试夹具]]：只用于管线回归，不是小说正典或研究资料。
