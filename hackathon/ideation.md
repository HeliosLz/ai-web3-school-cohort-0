# Hackathon Ideation

> Week 2 产物：列出 2-3 个候选方向，并用 cohort 5 问判断它是不是 AI x Web3 真交叉，而不是 buzzword 拼接。

## 候选 1：Hermes Auditor - staking transaction risk reviewer

### 一句话

Hermes Auditor 守在 `planning -> review -> authorization` 这条缝上，把 Agent 生成的不可读 staking 计划、calldata、simulation、session key scope 翻译成人能快速判断的风险摘要，并在字段不一致时 STOP。

### 为什么是 AI x Web3

AI 侧的问题是：Agent 会把模糊用户意图补全成具体行动计划，但模型输出有概率性，不能直接变成不可逆交易。

Web3 侧的问题是：staking deposit 是 32 ETH、官方 deposit contract、withdrawal credentials、session key、nonce、gas、tx hash 共同组成的不可逆链上动作，错误可以公开验证，但未必可撤销。

Auditor 的价值在两侧接缝：它不是单纯聊天助手，也不是普通钱包提示框，而是把 AI plan 和 Web3 execution 之间的差异结构化、复算、标红，再交给用户确认。

### Cohort 5 问

| 问题 | 当前答案 |
|---|---|
| 谁发起？ | 用户发起高层意图，例如"帮我准备一次 32 ETH solo staking deposit"。Agent 也可能发起后续建议，但不可逆动作必须回到用户确认。 |
| 谁执行？ | Agent 生成 staking plan，Web3 tools 构造和模拟 tx，Auditor 生成 risk summary，session key / wallet policy 执行有界授权，最终 send tool 发交易。 |
| 谁付钱？ | 用户支付 32 ETH 本金和 gas。产品层可能由项目方承担少量 RPC / simulation / AI 推理成本。这里是最需要验证的商业问题：Auditor 是否能作为高价值 staking flow 的安全层收费。 |
| 谁验证？ | 系统验证 raw_facts / derived_checks / simulation / policy；用户验证 risk summary；链上验证最终交易；回放日志和 golden set 验证 Auditor 是否漏报或误报。 |
| 谁担风险？ | 用户承担资金损失和提款权错误的最终风险；产品方承担误导性摘要、漏拦截、重复提交造成的责任；Agent / tool layer 必须通过 STOP、single-use authorization、schema binding 降低风险。 |

### 保护的资产

- 32 ETH 本金
- `withdrawal_credentials` / 提款权
- official deposit contract address
- user confirmation 的完整性
- session key scope
- chain-aware context 的新鲜度和来源

### 核心 threat model

| Threat | 例子 | Auditor / system response |
|---|---|---|
| transaction substitution | 用户确认的是 withdrawal address A，submit 前 calldata 变成 B | `calldata_hash` / `deposit_data_root` / `withdrawal_credentials_raw` mismatch -> STOP_AND_REVIEW |
| context poisoning | 错误 context 声称 attacker 地址是用户提款地址 | 要求 provenance + ownership proof + cross-check，不满足 STOP_OR_REQUIRE_PROOF |
| authorization replay | 旧 confirmation / session key 在 15 分钟内被重复使用 | `max_transactions=1`、single-use confirmation、submit 后 mark used / revoke |
| tool misuse | 用户授权 staking deposit，但 Agent 调用 x402 payment / generic transfer / swap | allowed_tools / allowed_contracts / allowed_functions 硬拦截 |
| stale context execution | 使用旧 gas / balance / nonce 发送交易 | submit 前 REFRESH，刷新后不满足条件则 BLOCK_SUBMIT |

### 最小 demo 路径

1. 输入：一份 staking plan JSON + context package + tx draft + simulation result + session key scope。
2. Auditor 生成 risk summary：资产变化、权限变化、目标对象、失败风险、来源核对。
3. 用户确认后生成 authorization package。
4. 提交前做 refresh / revalidate。
5. 展示 5 个 regression cases：正常通过、合约地址错误、withdrawal credentials 错误、calldata 被偷换、旧 authorization replay。

### Demo 不做什么

- 不真实转 32 ETH。
- 不自研完整 wallet。
- 不做通用合约审计。
- 不承诺自动选 validator 或收益优化。

## 候选 2：Agent Wallet Policy Auditor - policy mirror checker

### 一句话

给定链上 session key / smart account policy，自动生成应用层 guardrails mirror，并检查两层是否一致，避免 Agent 以为自己能做的事和链上真正允许的事不一致。

### Cohort 5 问

| 问题 | 当前答案 |
|---|---|
| 谁发起？ | 开发者或 wallet operator 在配置 session key policy 后发起检查。 |
| 谁执行？ | Auditor 读取 policy schema，生成 guardrail rules，跑测试样本和组合攻击检查。 |
| 谁付钱？ | 开发团队 / wallet provider / staking service 付钱，因为这是上线前安全检查。 |
| 谁验证？ | 链上 policy 是 canonical source；应用层 guardrails 是 mirror；测试集验证 mirror 是否漏掉 allowed action / forbidden action。 |
| 谁担风险？ | 如果 mirror 和链上 policy 不一致，用户承担资产风险，产品方承担安全责任。 |

### 当前判断

这是很强的 AI x Web3 工程方向，但依赖具体 policy schema（例如 smart sessions / account abstraction SDK）。Hackathon 上可能比候选 1 更偏开发工具，demo 需要接真实库文档和 schema。

## 暂定排序

1. **Hermes Auditor - staking transaction risk reviewer**：更贴近当前连续实验，用户价值和 demo 场景清晰。
2. **Agent Wallet Policy Auditor - policy mirror checker**：技术味更强，适合后续作为候选 1 的底层模块或第二 demo。

## Week 2 末待决

- [ ] 选择最终主线：staking transaction reviewer vs policy mirror checker。
- [ ] Q11 架构决定：FSM 实现路径（自研 / LangGraph / LangGraph+SDK）。
- [ ] 把 `context_package`、`authorization_package`、`threat_model` 合并成一份 tracer-bullet 输入输出 schema。
- [ ] 选 5 个 regression cases，防止 Auditor 变成"全打 FAIL"或"只会解释不会拦截"。
