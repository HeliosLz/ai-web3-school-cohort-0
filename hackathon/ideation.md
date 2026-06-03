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

### ⚠️ 关键边界（2026-06-03 调研 Cobo 后修正）：Auditor 坐在 CAW **之上**，不重写它

读 Cobo 文档后发现：**CAW 的 Pact 已经做完了授权执行层**——Pact = 意图+执行计划+policy+完成条件，owner 批准，签名层强制 allowlist/额度（agent 碰不到私钥、绕不过），完整审计、随时 revoke、MPC 双签。这几乎逐字就是旧 `authorization_package`。

→ **不要重写这层**，否则评委看到的是"把 Cobo 自家核心功能重做一遍"。Auditor 的差异化只能在 CAW **够不到**的三件事上（CAW 只查"在不在 allowlist/额度内"，不查下面这些）：

1. **语义正确性**：这笔采购真服务用户意图吗？vendor 是真的、还是恰好落在过宽 allowlist 里的**仿冒地址**？价格合理吗？
2. **输入侧防注入**：agent 被恶意文档/context 污染，误把 attacker 当合法 vendor，而 pact policy 够宽 → CAW 会放行；Auditor 在 planning 阶段就拦（= 6.01 学的输入侧攻击）。
3. **可读 risk summary**：CAW 的 Pact 第 3 步"你审阅批准"给的是通用风险展示；Auditor 产出专业版，让 owner 的批准从橡皮图章变成真决策（= 5.25 最弱环 planning→review 可读性）。

**定位一句话**：CAW = 无法被越权的合规部门（执行/审计/撤销，直接用）；**Hermes Auditor = 坐在 CAW 之上的分析师**（语义 + 防注入 + 可读摘要）。这正好落在 [[project_auditor_seam]] 早定的 risk_summary 节点上。

### 赛道决定

- **主攻 Cobo（Agentic Economy × CAW）**：Auditor 的权限控制 / 安全隔离 / 风险边界，正是 Cobo **评审侧重点**白纸黑字要的（"项目需体现 CAW 在权限控制、安全隔离方面的价值""风险边界说明"）。最稳的拿分点。
- **GLM-5.1 当 Agent 的脑**：反正要个模型驱动 planning，用 GLM-5.1 = 免费保留 Z.AI 资格。
- **长程是架构与叙事，不是第二个合规目标**：能做多少做多少，绝不拖垮"可运行"承重墙。提交那刻按分数挑赛道。

### 场景（CAW 能力已调研确认，待 Open Day 终敲）

倾向 **Agent Resource Procurement（Cobo 建议方向 03）**：Agent 按任务自主发现 / 比价 / 采购数据、API、算力等资源，Auditor 把关那笔采购支付，CAW 执行。
- 理由：天然就是"长程自主准备（发现比价）+ 不可逆支付（gate）"，比 Autonomous Trading 少很多移动部件，可运行性高。
- **x402 是金矿**：CAW 已有现成 **X402 Payment recipe**（Base，agent 自动付费调用 API）。其 typical prompt 是"只要**目标地址看起来安全**就自动付 0.2 USDC"——这个"看起来安全"现在是塞 prompt 里碰运气，**把它做硬 = Auditor + 采购/支付最自然的结合点**，且直接命中 Cobo 建议方向 01（Agent-Native Payments）。
- 最小故事：用户"用 ≤50 测试 USDC 买齐分析 X 需要的数据"→ Agent 找到两个数据源比价 → Auditor 发现其一收款地址来源不可信（注入防御）/ 价格离谱（语义检查）标红 → 用户放行正常那个 → CAW 在 Sepolia 真付一笔 → 审计日志回放。

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

## 接入事实（2026-06-03 调研 Cobo + Z.AI 官网，已确认）

**Cobo CAW**（公司背景：2017 成立的机构托管/钱包基础设施，$3.8T+ 资产、80+ 链、MPC 看家本领；CAW = "AI Agent 上链的信任层"，早期免费）
- **链 / 测试网**：Ethereum / Base / Solana 主网；测试网 **Sepolia (SETH)、Base Sepolia、Solana Devnet**，**内置 faucet**。→ 测试网真执行可行。
- **操作**：transfer / contract_call / payments / 批量 / 签名；约 20 工具，预设 Pact Drafting / Execution / Observer。Recipes 含 Token Transfer、**X402 Payment**、Uniswap/Jupiter/Aave 等。
- **接入**：CLI + Python SDK + TS SDK + MCP；框架集成含 **LangChain**（接 LangGraph 零摩擦）/ OpenAI Agents / CrewAI / Agno。
- **Claude Code skill**（写代码时装）：`npx skills add CoboGlobal/cobo-agentic-wallet --skill cobo-agentic-wallet-developer --yes --global`
- 文档：cobo.com/products/agentic-wallet/manual/ · recipes：agenticwallet.cobo.com/agentic-wallet/recipes

**Z.AI GLM-5.1**
- **用 General API**（`https://api.z.ai/api/paas/v4/chat/completions`，Bearer，`model="glm-5.1"`），SDK `pip install zai-sdk` **或直接用 OpenAI SDK 改 base_url**（接 LangGraph 几乎零成本）。
- ⚠️ **别用 GLM Coding Plan**：那是给 Claude Code 等编码工具的订阅，文档明说 SDK/第三方集成访问可能被限制——不是给你 Agent 运行时用的。
- **API 补贴 ≠ 官网**：是 hackathon 给的，走社群申请（Telegram t.me/aiweb3school / 微信 clynn2024）→ Open Day 问。

## 待决（部分留给 Open Day 6.02 20:00–21:00）

- [x] ~~CAW 能力确认（链/操作/测试网/SDK）~~ → 已调研，见上
- [x] ~~GLM-5.1 接入方式~~ → General API + OpenAI 兼容 SDK，见上
- [ ] **Open Day 问**：① API 补贴怎么申请 ② CAW 的 Pact 批准流程能否塞进我自己的 risk summary（Auditor↔CAW 接缝）③ Agent Resource Procurement 方向 Cobo 认不认
- [ ] **场景终敲**：Procurement（首选）vs x402 Payments——听完 Open Day 定。
- [ ] **最小 golden set**：5–6 个 regression cases（正常通过 / 合约错 / 授权 replay / 字段偷换 / 恶意文档注入 / 余额不足）。
