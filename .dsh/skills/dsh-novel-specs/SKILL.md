---
name: dsh-novel-specs
description: 小说双轨拆分、小说到 AI 视频闭环与内容审计只读审查 Worker 规格的解读与应用。 / Expert for the novel dual-track trial, novel-to-AI-video pipeline and solo-company audit worker specifications.
---

# 小说与内容审计规格文档专家 / Novel & Solo-Company Specification Expert

本技能面向与内容业务相关的三份规格：`specs/Novel-Chapter-Dual-Track-Trial-v0.1.md`（逻辑包/表现包双轨拆分）、`specs/Novel-to-AI-Video-Pipeline-v0.1.md`（导演包→图像→视频→装配→QC 五阶段管线）、`specs/Solo-Company-Audit-Worker-v1.md`（内容审计只读审查）。三者都以合同、状态、哈希、门禁与人工放行为核心，不把模型候选误标为正式产物。规格为纯文档。

This skill covers three content-business specifications: `specs/Novel-Chapter-Dual-Track-Trial-v0.1.md` (logic/performance dual-track packs), `specs/Novel-to-AI-Video-Pipeline-v0.1.md` (director → images → video → assembly → QC pipeline), and `specs/Solo-Company-Audit-Worker-v1.md` (read-only solo-company audit). All center on contracts, states, hashes, gates and human release — model candidates are never mistaken for official artifacts. Pure documentation.

## When to use / 何时使用

评审或实现小说章节的候选包拆分（锁因果、留表现自由）；设计小说到 AI 视频的可审计管线与人工门；实现内容审计的只读审查闭环；任何需要「源只读 + 哈希绑定 + 人工放行」的内容管线。

When reviewing or implementing dual-track candidate packs for novel chapters (lock causality, keep expressive freedom); designing auditable novel-to-AI-video pipelines with human gates; implementing read-only solo-company audits; or any content pipeline needing read-only sources, hash binding and human release.

## Workflow / 工作流

1. 双轨拆分：先读逻辑包规格，理解场景因果与高风险动作链；再读表现包规格，理解文风/对话/笑点/情绪与 QC 门。
2. 视频管线：按 source_lock → director → image_design → video_generation → assembly → qc 的阶段顺序核对哈希链与人工门。
3. 只读审查：按机器注册表校验 → 只读快照 → 边界/文件头/证据检查 → 报告 + 回执 + Ledger 的顺序执行。

1. Dual-track trial: read the logic-pack spec for scene causality and high-risk action chains, then the performance-pack spec for style/dialogue/humor/emotion and QC gates.
2. Video pipeline: trace source_lock → director → image_design → video_generation → assembly → qc, checking hash chains and human gates per stage.
3. Read-only audit: registry verification → read-only snapshot → boundary/header/evidence checks → report + receipt + Ledger.

## References / 参考

- 项目 README: 见仓库根目录
- 作者: h565656445 (GitHub)