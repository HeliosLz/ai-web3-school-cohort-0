# Session Key Policy: Hermes Agent Tool Payment (Base Sepolia)

> Handbook 最小实践产出 — [Account Abstraction](https://aiweb3.school/zh/handbook/web3/account-abstraction/)
> 场景：让 Hermes Agent 在 Base Sepolia 上自动为已注册工具调用付费，限定额度与目标。
> 这是一份**授权契约**，不是代码。目的是把"Agent 能做什么"写成机械可核对的规则。

## 设计原则

1. **default-deny**：白名单制，没明确允许的全部拒绝。
2. **最小可工作集**：不问"Agent 可能需要什么"，问"Agent 完不成任务就坏了的最小集合"。
3. **按可承受损失定额**：`per_tx_budget` = 最坏一笔愿意一次性损失多少；`window_budget` = 反应窗口内的总损失上限。
4. **必须列 escalation**：哪些情况要打断 Agent 回到用户签名——漏掉这条，session key 就退化成"小号私钥"。

## 4 层切片

```
1. Identity  — 谁签发、谁持有、签发者用什么验证
2. Scope     — 在哪条链、哪个合约、哪个方法
3. Limits    — 每笔多少、累计多少、多久内、最多几次
4. Lifecycle — 何时失效、谁能撤销、出事怎么停、日志在哪
```

## 策略表

```yaml
# === 1. Identity ===
session_id:        ses_2026-05-21_hermes_tool_payment_01
issuer:            <你的 Smart Account on Base Sepolia>     # TODO 填地址
holder:            <Hermes Agent 临时密钥>                  # TODO 填地址
issuer_signature:  <on-chain authorize tx hash>            # TODO

# === 2. Scope ===
chain_id: 84532  # Base Sepolia
allowed_targets:
  - contract: 0xToolRegistry            # TODO 实际地址
    methods:  [payForCall]
    notes:    "只允许给已注册工具付费，不能 transfer 任意地址"
  - contract: 0xUSDC_BaseSepolia        # TODO 实际地址
    methods:  [approve]
    extra_constraints:
      - "spender 必须 = 0xToolRegistry"
      - "amount ≤ 10 USDC"

# === 3. Limits ===
per_tx_budget:  2 USDC
window_budget:  20 USDC / 24h
max_tx_count:   20 / 24h
gas_policy:     paymaster sponsors all UserOps in scope, cap 0.005 ETH/24h

# === 4. Lifecycle ===
valid_from:   2026-05-21T08:00:00Z
valid_until:  2026-05-24T08:00:00Z   # 3 天
revocation:
  - "issuer 调 SmartAccount.revokeSession(ses_2026-05-21_hermes_tool_payment_01) 立即生效"
kill_switch:  "连续 3 笔失败 → 自动暂停 30 min，需 issuer 重启"
escalation_to_user:
  - "调用 ToolRegistry / USDC 以外的合约"
  - "单笔 > per_tx_budget (2 USDC)"
  - "24h 累计 > window_budget (20 USDC)"
  - "工具地址首次出现（未在 last_seen_tools 中）"
  - "策略到期续签"
audit_log:
  - "每次 UserOp 写入 0xLogger 合约 event: (session_id, tool_id, amount, op_hash)"
  - "Agent 本地存 experiments/session-keys/audit-<session_id>.jsonl"
```

## 红队自检

> 攻击者拿到 holder 私钥后，能在策略范围内做到什么？

| 攻击路径 | 最大可提取价值 | 缓解 |
|---|---|---|
| 反复 payForCall 给攻击者控制的"工具" | 20 USDC / 24h，3 天合计 60 USDC | escalation: 首次出现的 tool_id 必须用户确认；事后审计 jsonl |
| 把 USDC approve 给 ToolRegistry 后利用合约 bug 提走 | 取决于 ToolRegistry 实现 | approve 额度 ≤ 单笔预算；ToolRegistry 自身需要审计（依赖项目而定） |
| 通过 paymaster 套薅 gas | 0.005 ETH / 24h | paymaster 限定 scope；超出阈值需 issuer 续签 |
| 让 Agent 卡死消耗 paymaster | 至 kill_switch 触发为止 | 连续 3 笔失败暂停 30 min |

**底线判断**：拿到 holder 私钥的最坏 3 日总损失 ≤ 60 USDC + 0.015 ETH gas + ToolRegistry 合约风险。**如果这个数字让你不舒服，缩 budget / 缩 valid_until / 加 escalation。**

## 真要上链时的映射

这份 yaml 不能直接部署。如果走完整 AA 路径，每个字段对应到具体实现：

| 本策略字段 | Rhinestone Smart Sessions | ZeroDev Session Keys |
|---|---|---|
| `allowed_targets` | `Policy` modules (TargetCallPolicy) | `permissions[].target` |
| `per_tx_budget` / `window_budget` | `SpendingLimitPolicy` | `permissions[].valueLimit` + custom |
| `valid_from` / `valid_until` | `TimeFramePolicy` | `validAfter` / `validUntil` |
| `escalation_to_user` | 不在 session 范围 → 自动 fallback 主签名 | 同上 |
| `revocation` | `SmartSession.revokeSession()` | `revokePermission()` |

## TODO（如果继续推进）

- [ ] 部署 Smart Account（候选：Safe / Kernel / Biconomy v3）到 Base Sepolia
- [ ] 部署或选择 Paymaster（候选：Pimlico / Alchemy / StackUp）
- [ ] 把上述 yaml 转成 Rhinestone Smart Sessions 的 policy 配置
- [ ] 跑一笔真实 payForCall，观察 UserOp → Bundler → EntryPoint → Smart Account 全链路
- [ ] 验证 escalation 路径：触发"非白名单合约"是否真的回到用户签名

## 学到了什么

- **AA 的核心不是免 gas**，是把账户控制权从私钥扩展成可编程规则。
- **Session Key = capability-based security 的链上实现**：钥匙形状（合约 × 方法 × 额度 × 时间 × 链）的合取定义 blast radius。
- **4337 的三个外部角色都是新的失败面**：Bundler（模拟与执行不一致）、Paymaster（spam / 套利）、EntryPoint（合约本身）。
- **escalation_to_user 是 session key 区别于"小号私钥"的关键字段**。
