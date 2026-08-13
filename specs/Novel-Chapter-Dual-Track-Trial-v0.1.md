---
type: harness-spec
status: trial
created: 2026-07-17
updated: 2026-07-17
---

# 小说章节双轨拆分试点 v0.1

## 目标

在不复制小说正文进 Vault、不修改候选正文、不写正式正典的前提下，把单章执行材料拆成两份相互绑定的候选包：

- **逻辑包**：锁定状态变化、场景因果、时间、空间、知情、道具和高风险动作链。
- **表现包**：管理文风、对话、笑点、情绪和意外感，同时明确正文可自由发挥的区域。

这两份包都是候选运行材料，不是 Canon、正式章纲或正式状态。

## 为什么分轨

逻辑规划越细，越能减少时间错算、空间跳跃、人物凭空知情和道具错位；但若把句子、台词和情绪反应也提前写死，正文会退化成机械扩写。双轨方案只把因果锁死，把具体措辞、停顿、动作质感和局部包袱留给正文阶段。

## 输入边界

初始化必须绑定：

1. 同一条 Harness TaskContract；
2. 活动小说 TaskContract；
3. 当前章节执行规划卡；
4. 当前隔离候选正文；
5. 当前正式文风指纹；
6. 当前角色声音卡。

规划卡、候选和文风指纹必须与活动小说 TaskContract 的 `authority_hashes` 一致。角色声音卡若未进入活动合同，只能以试点级哈希绑定，不能冒充正式合同输入。

初始化时，Runner 会在同一 Harness 任务目录写入 `novel_chapter_trial_anchor.json`。该锚点固定任务身份、小说根目录、试点目录、五个且仅五个来源角色、来源哈希与三个输出文件名；试点目录里的 `trial_contract.json` 不能自行改写这些边界。Harness 合同必须是 `<RuntimeRoot>/tasks/<task_id>/contract.json`，合同、来源及试点路径均不得穿过重解析点或目录联接。

正文只读取并计算 SHA-256，不复制到 Harness Runtime 或 Obsidian Vault。

## 状态机

```text
waiting_for_user
→ routed
→ awaiting_packs
→ retrying（最多一次）
→ waiting_for_approval

任一源哈希漂移 → failed
第二次结构验证失败 → failed
```

验证通过只表示两份包结构完整、身份一致、源文件未漂移，并不表示正文质量已经获得用户认可。

## 逻辑包

逻辑包必须包含：

- 章节进入状态、退出状态和不可逆变化；
- 每场的进入状态、目标、阻碍、已知信息、替代方案、选择、行动、后果和退出状态；
- 至少一条高风险动作链；
- 每个动作节拍的时间、位置、知情、道具前后状态、即时后果和下一步成立条件；
- 本章不变量和非目标。

## 表现包

表现包必须覆盖：

- **文风**：声音原则、僵硬风险、自由发挥区、禁用模式；
- **对话**：人物目标、隐瞒、误判、压力和说话动作；
- **笑点**：铺垫、错位、落点、真实后果和剧情功能；
- **情绪**：场景情绪起点、终点、失去、代价选择和余波；
- **意外感**：读者预期、角色预期、实际事件、反转轴、公平线索和后续改变；
- 五项独立 QC 门，并保留用户口味否决权。

喜剧单元只能引用逻辑包中存在的场景；情绪弧必须逐场覆盖逻辑包中的全部场景，且每场恰好一次。这样表现设计可以自由，但不能脱离本章真实发生的因果场。

## 有界修正

- 验证最多两次；
- 只允许一次结构修正；
- 源文件漂移不进入修正，直接失败关闭；
- `Verify` 和 `Status` 都会重新核对五类源文件哈希，不能返回已经过期的待审状态；
- `Status` 还会核对待审逻辑包、表现包与已验证哈希；输出包变化会撤销待审状态，剩余次数不足时直接失败关闭；
- 结构通过后进入人工审阅，不自动修改候选；
- 若用户批准把表现包用于改稿，必须另开正文修订任务并重新绑定候选哈希。

## Runner

初始化：

```powershell
pwsh -File .\10-项目\Hermes-Harness\runner\novel_chapter_trial.ps1 `
  -Action Initialize `
  -ContractPath "<Harness contract.json>" `
  -ClarifiedRequest "拆分并验证第6章逻辑包与表现包，不修改正文" `
  -NovelRoot "<小说工作区根>" `
  -NovelTaskContractPath "<活动小说合同>" `
  -ChapterPlanPath "<章节规划卡>" `
  -CandidatePath "<隔离候选>" `
  -StyleFingerprintPath "<正式文风指纹>" `
  -VoiceCardPath "<角色声音卡>" `
  -AsJson
```

填充 `logic_pack.json` 与 `performance_pack.json` 后验证：

```powershell
pwsh -File .\10-项目\Hermes-Harness\runner\novel_chapter_trial.ps1 `
  -Action Verify `
  -TrialPath "<harness_dual_track_trial>" `
  -NovelRoot "<小说工作区根>" `
  -AsJson
```

`Verify` 与 `Status` 都必须重新提供小说工作区根；Runner 同时用自身默认 Harness RuntimeRoot 复核可信锚点、TaskContract、初始化 Ledger 事件与 Ledger 的真实位置。每次验证仍会完整重跑 Schema、身份、来源绑定和场景交叉引用校验，不能只凭试点目录里的旧回执跳过验证。验证转换先写回执，再追加可信 Ledger，最后同步合同和试点状态；若中途崩溃，下一次 `Verify` 或 `Status` 都依据 Ledger 恢复，不重复消耗验证次数。Ledger 的次数和动态状态是单调恢复下界，试点快照不能自行回滚次数或伪造待审状态；Harness 合同的 `failed`、`completed` 终态也不能被旧试点事件复活。

## 明确非目标

- 不接入小说自动写作 Worker；
- 不把 Harness 的结构验证描述为文学质量通过；
- 不修改候选正文、正式正文、Canon、Tracker、Lore、伏笔或读者承诺；
- 不把《老掌柜》专属粗口、下三路密度等规则带入《规则怪谈》；
- 不用机器评分覆盖用户对笑点、情绪和意外感的判断。
