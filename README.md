# dsh-novel-specs

<!-- DeepSeek Harness 衍生声明 -->
> **DeepSeek Harness 个人适配声明（Personal Adaptation Notice）**
>
> 本项目是 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) 的**个人适配产物（personal adaptation）**，**并非 DeepSeek Harness 官方文件（not an official DeepSeek Harness file）**，随附功能、使用说明与个人产物（bundled with features, documentation, and personal artifacts），可与 DeepSeek Harness 搭配使用，也可独立使用。
>
> This project is a **personal adaptation** for DeepSeek Harness, and is **NOT an official DeepSeek Harness file**, bundled with features, documentation, and personal artifacts. It can be used alongside DeepSeek Harness or standalone.

**作者 / Author**: [h565656445](https://github.com/h565656445)

**合作 / Collaboration**: 如有项目可以一起合作，欢迎联系。微信：`wohaishihenshuaide`。If you have projects, let's collaborate. WeChat: `wohaishihenshuaide`.


---

## 用途 / What this is for

小说管线规格集：章节双轨试写、小说到 AI 视频管线与审计 Worker 的规格文档。

Novel pipeline specs: dual-track chapter trial, novel-to-AI-video pipeline and audit worker specifications.

---
## Novel & Solo-Company Specifications / 小说与内容审计规格文档

本仓库收录 Hermes Harness 中与内容业务相关的三份规格：小说章节双轨拆分试点 v0.1、小说到 AI 视频闭环 v0.1、内容审计只读审查 Worker v1。它们分别定义候选包拆分、可审计的视频管线与只读审查闭环的控制链——全程以合同、状态、哈希、门禁与人工放行为核心，不把模型候选误标为正式产物。本仓库为纯文档仓库，不包含任何源码。

This repository collects three content-business specifications from Hermes Harness: the Novel Chapter Dual-Track Trial v0.1, the Novel-to-AI-Video Pipeline v0.1, and the Solo-Company Read-Only Audit Worker v1. They define control chains for candidate-pack splitting, an auditable video pipeline, and a read-only audit loop — centered on contracts, states, hashes, gates and human release, never mistaking model candidates for official artifacts. Pure documentation; no source code is included.

## Features / 功能

- **双轨拆分试点 v0.1**：逻辑包（因果/状态）与表现包（文风/对话/笑点/情绪）分离，源哈希绑定与有界修正
  Dual-track trial v0.1: logic pack (causality/state) and performance pack (style/dialogue/humor/emotion), source-hash binding, bounded repair
- **小说到 AI 视频闭环 v0.1**：五阶段产物（导演包→图像→视频→装配→QC）逐级哈希校验与人工门放行
  Novel-to-AI-video pipeline v0.1: five-stage artifacts (director → images → video → assembly → QC) with per-stage hash checks and human gates
- **内容审计只读审查 v1**：机器注册表校验、路径/文件头/证据检查、Markdown 报告 + JSON 回执 + Ledger
  Solo-company read-only audit v1: registry verification, path/header/evidence checks, Markdown report + JSON receipt + Ledger
- **安全边界**：源只读、凭据扫描、拒绝发布/删除/付款/账号等敏感副作用
  Safety boundaries: read-only sources, credential scanning, rejection of publish/delete/payment/account side effects
- **人工放行**：所有候选产物必须经人工复核，机器验证不替代用户判断
  Human release: every candidate artifact requires human review; machine verification never substitutes user judgment

## What's inside / 目录结构

```
dsh-novel-specs/
├── README.md              # 双语说明 + DSH 衍生声明
├── LICENSE                # MIT
├── specs/                 # 内容业务规格（3 份，纯文档）
│   ├── Novel-Chapter-Dual-Track-Trial-v0.1.md
│   ├── Novel-to-AI-Video-Pipeline-v0.1.md
│   └── Solo-Company-Audit-Worker-v1.md
└── .dsh/                  # DeepSeek Harness 衍生包
    ├── preset.yml
    ├── agent.cordis.yml
    ├── README.md
    └── skills/dsh-novel-specs/SKILL.md
```

## Quick start / 快速开始

```powershell
# 1. 浏览规格清单
$repo = "E:\path\to\dsh-novel-specs"
Get-ChildItem (Join-Path $repo "specs") -Filter *.md | Select-Object Name

# 2. 阅读双轨拆分规格
Get-Content (Join-Path $repo "specs\Novel-Chapter-Dual-Track-Trial-v0.1.md")

# 3. 安装 DSH 预设（可选）
$dst = Join-Path $env:DSH_HOME ".agent-presets\novel-specs"
Copy-Item -Recurse -Force (Join-Path $repo ".dsh") $dst
```

## DeepSeek Harness 衍生 / DSH Derivative

本项目附带 DeepSeek Harness 衍生包，位于 `.dsh/` 目录：

- `preset.yml` — Agent 预设元数据
- `agent.cordis.yml` — Cordis 组装（基于 standard 预设，persona 已定制）
- `skills/dsh-novel-specs/SKILL.md` — 项目专属技能（skill）

安装与接入方式见 [`.dsh/README.md`](.dsh/README.md)（双语）。


## License / 许可证

[MIT](LICENSE)