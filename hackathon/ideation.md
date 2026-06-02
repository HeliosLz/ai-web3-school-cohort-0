# Hackathon Ideation

> **2026-06-02 重写**：hackathon 规则落定（https://casualhackathon.com/hackathons/cmpsjubkg0003p80kxuzrdyjy），方向重构。
> 旧版"32 ETH staking 单步审查 + 不真转钱"被规则推翻（Cobo 要求 Agent 真实资金执行，不收纯流程设计层）。旧框架见 git 历史。

## 比赛硬约束（来自规则页）

- **时间**：Build 6.1–6.12 / **提交截止 6.13 12:00（UTC+8）** / Demo Day 6.14 / 获奖公示 6.17
- **必须可运行 Demo**，不接受 PPT / 概念 / Mockup
- **每队只提交一个项目，进一个赛道**（项目可归多赛道时，选最能体现价值的方向）
- 奖池 7000 USDT（Cobo 3500 / Z.AI 3500）
- 单干 · ~1 小时/天 · 0 行代码起步 → 承重墙只能有一面

## 锁定方向：长程可审计 Agent，主攻 Cobo，保留 Z.AI 选择权

**一句话**：一个 GLM-5.1 驱动、能自主拆解多步任务的 Agent，自主完成所有**可逆**的准备工作（发现 / 比价 / 拆解 / 构造 / 模拟 / 迭代修复），在走到**不可逆的资金操作**时由 Auditor 停下、标红、等用户放行，再经 **Cobo Agentic Wallet (CAW)** 在测试网真实执行。

### 核心设计洞察（这是项目最锋利的点）

> **长程自主跑在"可逆"的活上，human gate 只卡在"不可逆"那一步。**

Auditor 因此从"碍事的限制器"升级成"**让 Agent 敢于长程自主的前提**"——正因为有人守着不可逆那道闸，才敢让 Agent 自己跑那么久。这一句同时命中两个赛道：长程自主 = Z.AI，安全边界 + CAW 真执行 = Cobo。

### 赛道决定

- **主攻 Cobo（Agentic Economy × CAW）**：Auditor 的权限控制 / 安全隔离 / 风险边界，正是 Cobo **评审侧重点**白纸黑字要的（"项目需体现 CAW 在权限控制、安全隔离方面的价值""风险边界说明"）。最稳的拿分点。
- **GLM-5.1 当 Agent 的脑**：反正要个模型驱动 planning，用 GLM-5.1 = 免费保留 Z.AI 资格。
- **长程是架构与叙事，不是第二个合规目标**：能做多少做多少，绝不拖垮"可运行"承重墙。提交那刻按分数挑赛道。

### 场景（待 Open Day 确认 CAW 能力后定稿）

倾向 **Agent Resource Procurement（Cobo 建议方向 03）**：Agent 按任务自主发现 / 比价 / 采购数据、API、算力等资源，Auditor 把关那笔采购支付，CAW 执行。
- 理由：天然就是"长程自主准备（发现比价）+ 不可逆支付（gate）"，比 Autonomous Trading 少很多移动部件，可运行性高。
- 备选：Agent-Native Payments（402）/ A2A 分账。Open Day 问清 CAW 支持的链 / 操作类型 / 测试网后定。

## Cohort 5 问（按新框架重答）

| 问题 | 答案 |
|---|---|
| 谁发起？ | 用户给高层意图 + 预算（如"用 ≤50 测试 USDC 帮我采购完成 X 任务所需的数据/API"），Agent 自主拆解为多步计划。 |
| 谁执行？ | Agent（GLM-5.1 驱动）自主跑可逆准备（发现/比价/构造/模拟/迭代）；Auditor 生成 risk summary 把关不可逆步；CAW 在授权边界内真实执行资金操作。 |
| 谁付钱？ | 用户的 CAW 账户资金（测试网）；Agent 在用户授权的边界内支配。商业层：安全审计 + 边界执行可作为 agentic commerce 的信任层收费。 |
| 谁验证？ | 系统验 raw facts / simulation / policy；用户验 risk summary 放行不可逆步；链上验 CAW 交易；回放日志 + golden set 验 Auditor 是否漏报/误报。 |
| 谁担风险？ | 用户承担资金损失；产品方承担误导摘要/漏拦截；Auditor 用 STOP + single-use 授权 + 字段绑定 + CAW 权限隔离降风险。 |

## 可平移的已有资产（旧设计几乎不浪费）

旧设计本就是"审一笔不可逆资金动作"，把 `deposit` 换成 `CAW 的一笔 payment/procurement` 基本平移：

- **threat_model**（5.30）：transaction substitution / context poisoning / authorization replay / tool misuse / stale context —— 五类威胁对 CAW 资金操作同样成立，+6.01 新增恶意文档注入。
- **authorization_package**（5.29）：`risk_summary → user_confirmation → session_key_scope → execution_allowed`，session_key 换成 CAW 的权限/policy。
- **web3 tool specs**（5.27）：A–E 工具阶梯（read/draft/simulate/wallet/write）平移到 CAW 调用。
- **FSM 状态机图**：IDLE→PLANNING→RISK_REVIEW→AUTHORIZED→SUBMITTING→DONE（+STOPPED/BROADCAST_UNKNOWN），把 SUBMITTING 接到 CAW 执行。
- **架构**：LangGraph（[[project_architecture_langgraph.md]]）——interrupt=HUMAN GATE，checkpointer=回放，GLM-5.1 接进 node 当 planning 脑。

## 承重墙 / 最小可跑核（先立这一面）

```text
用户意图+预算
  → Agent (GLM-5.1) 自主拆解 + 准备一笔 CAW 资金操作（可逆，可多步）
  → Auditor 生成 risk summary + 字段绑定校验
  → 用户放行（LangGraph interrupt）
  → CAW 在测试网真实执行一笔
  → 回放日志（LangGraph checkpointer）
```

立住后再往上加砖：长程步数、自我纠错、更多 regression cases（含恶意文档注入）。

## 待决（部分留给今晚 Open Day 6.02 20:00–21:00）

- [ ] **CAW 能力确认**：支持哪些链 / 操作类型（payment / transfer / procurement / treasury）？测试网？SDK 接入难度？API 补贴怎么申请？
- [ ] **场景定稿**：Agent Resource Procurement vs Payments vs A2A——按 CAW 实际能力选。
- [ ] **GLM-5.1 接入**：docs.z.ai，确认 API 接入 + LangGraph 怎么对接。
- [ ] **最小 golden set**：5–6 个 regression cases（正常通过 / 合约错 / 授权 replay / 字段偷换 / 恶意文档注入 / 余额不足）。
