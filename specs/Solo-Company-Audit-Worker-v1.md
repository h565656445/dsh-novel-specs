---
type: spec
status: active
created: 2026-07-17
updated: 2026-07-17
---

# 内容审计只读审查 Worker v1

## 目标

从已校验且路由到 `content_audit` 的 TaskContract 恢复，只读审查当前平台文案、内容登记、成片与封面，输出可定位的 Markdown 报告和 JSON 回执。

```text
TaskContract → 校验当前机器注册表 → 只读快照与 SHA-256
→ 成片包/文案/服务边界检查 → 确定性验证 → 报告 + 回执 + Ledger
```

## 公共入口

```powershell
pwsh -File .\10-项目\Hermes-Harness\runner\content_audit_worker.ps1 `
  -ContractPath ".\10-项目\Hermes-Harness\runtime\tasks\<task_id>\contract.json" `
  -AsJson
```

必须从同一 `routed` 合同恢复；不新建替代任务。
公共 Runner 仅接受合同路径和输出格式参数；运行目录、项目注册表与两份 Schema 均由 Harness 根目录固定解析，调用方不能覆盖这些权威路径。

## 执行边界

- 仅接受且必须恰好存在一次 `content_audit + content_distribution + analyze` 路由；多余项目域或动作同样拒绝。
- 合同含有发布、删除、付款、账号变更或核心规则修改时直接拒绝。
- 合同的项目根目录与项目记忆必须逐路径匹配当前机器注册表。
- 只读取项目规则、当前状态、发布文案资产、`content-registry.json`、`projects.json`及登记的成片/封面。
- 成片和封面路径必须是项目根下的相对路径；路径越界、任一现存路径段为符号链接/联接点、扩展名不符或文件头与声明类型不符时失败关闭。
- 不读取线索、客户联系方式、私聊、凭据或账号信息；不登录、不发布、不修改目标项目。

## 质量门

- 内容审计五项机器门禁必须各自恰好出现一次且取值正确；重复或互相冲突的声明失败关闭。
- 输入文本通过凭据模式扫描。
- 所有读取源（包括本轮生成的机器注册表）必须先记录路径、长度和 SHA-256，再解析内容；二进制包在同一只读句柄中先计算证据、再校验与扩展名绑定的文件头。写报告前重新检查路径联接点、文件头和证据，漂移即停止。
- 报告只写 Harness 任务目录；回执必须通过 `content_audit_audit_receipt.schema.json`。
- 回执写入时状态为 `verified`；只有报告、回执、合同和 Ledger 都持久化成功后，合同与 Ledger 才进入权威终态 `completed`。
- 包缺失、文案不完整或无运行数据是审查结果，不是 Worker 伪造成功的理由；必须显式写入风险。

## 非目标

- 不自动上传、发布、登录或记录客户数据。
- 不修改平台文案、价格、服务范围或内容审计运行状态。
- 不代替用户对最终内容、报价、交期和发布的复核。

返回：[[10-项目/Hermes-Harness/README|Hermes Harness]]。
