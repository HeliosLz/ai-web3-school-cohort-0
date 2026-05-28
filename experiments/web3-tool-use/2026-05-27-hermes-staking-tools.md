# Hermes Staking Web3 Tool Specs

> 日期：2026-05-27
> 目的：把 32 ETH staking workflow 拆成受限 Web3 tools，明确读 / 草稿 / 模拟 / 授权 / 写链边界。

## 1. 设计原则

Web3 Tool Use 的核心不是让 agent 能调用链，而是让每次链上动作都有：

- 结构化输入；
- 明确权限等级；
- 执行前硬检查；
- 可审计日志；
- 高风险动作的人类确认或 wallet policy 授权。

Hermes 不提供万能 RPC、万能 calldata、万能合约写入工具。写链能力必须被限制在白名单链、白名单合约、白名单方法和单笔额度内。

## 2. Tool 权限等级

| 等级 | 名称 | 是否改变链上状态 | 是否需要 human gate | 例子 |
|---|---|---:|---:|---|
| A | read | 否 | 否 | `get_eth_balance` |
| B | draft | 否 | 否 | `build_validator_deposit_tx` |
| C | simulate / risk-check | 否 | 否 | `simulate_deposit_tx` |
| D | wallet / authorization | 否，但决定能否执行 | 是 | `request_user_confirmation` |
| E | irreversible write | 是 | 必须已授权 | `send_deposit_tx` |

## 3. Tool Specs

### 3.1 `get_eth_balance`

**等级**：A / read

**用途**：读取用户在指定链上的 ETH 余额。

**输入 schema**

```json
{
  "chain_id": 1,
  "user_address": "0x..."
}
```

**输出字段**

```json
{
  "chain_id": 1,
  "rpc_provider": "string",
  "block_number": 0,
  "user_address": "0x...",
  "balance_eth": "0"
}
```

**STOP 条件**

- chain id 不在允许列表；
- address 格式非法；
- RPC 返回数据 block 过旧；
- 多 provider 结果明显不一致。

**日志字段**

`user_intent`、`tool_name`、`input`、`output`、`chain_id`、`block_number`、`rpc_provider`、`timestamp`、`error`

### 3.2 `build_validator_deposit_tx`

**等级**：B / draft

**用途**：生成 validator deposit 交易草稿，但不签名、不广播。

**输入 schema**

```json
{
  "chain_id": 1,
  "deposit_contract": "0x00000000219ab540356cBB839Cbe05303d7705Fa",
  "validator_pubkey": "0x...",
  "withdrawal_credentials": "0x...",
  "amount_eth": "32",
  "deposit_data_root": "0x..."
}
```

**输出字段**

```json
{
  "chain_id": 1,
  "to": "0x00000000219ab540356cBB839Cbe05303d7705Fa",
  "method": "deposit",
  "value_eth": "32",
  "calldata": "0x...",
  "validator_pubkey": "0x...",
  "withdrawal_credentials": "0x...",
  "deposit_data_root": "0x..."
}
```

**STOP 条件**

- chain id 与目标网络不一致；
- deposit contract 不是该 chain 的官方白名单地址；
- amount 不是 policy 允许的单笔额度；
- withdrawal credentials 不属于用户控制地址；
- deposit data root 与 pubkey / withdrawal credentials / amount 不一致；
- validator pubkey 来源不可追踪。

**Auditor 标红字段**

- `deposit_contract`：错误合约会导致资产进入错误路径；
- `withdrawal_credentials`：决定未来提款权，是 staking 特有高危字段；
- `amount_eth`：必须等于本次授权额度；
- `validator_pubkey`：首次出现或来源变化必须标红；
- `deposit_data_root`：必须能从结构化字段重算核对。

### 3.3 `simulate_deposit_tx`

**等级**：C / simulate / risk-check

**用途**：在广播前模拟 deposit 草稿，生成资产变化和失败风险。

**输入 schema**

```json
{
  "tx_draft": {
    "chain_id": 1,
    "to": "0x...",
    "method": "deposit",
    "value_eth": "32",
    "calldata": "0x..."
  },
  "simulation_provider": "tenderly-or-local"
}
```

**输出字段**

```json
{
  "simulation_status": "success",
  "expected_asset_delta": {
    "eth": "-32"
  },
  "expected_contract": "0x...",
  "expected_method": "deposit",
  "revert_reason": null,
  "warnings": []
}
```

**STOP 条件**

- simulation 失败；
- expected asset delta 与草稿不一致；
- 目标合约或方法与草稿不一致；
- 出现未知 token / ETH transfer；
- simulation provider 不可用时，不允许降级到直接发送。

### 3.4 `request_user_confirmation`

**等级**：D / wallet / authorization

**用途**：把机器计划和模拟结果翻译成人可批的风险摘要，并获取用户确认。

**输入 schema**

```json
{
  "risk_summary": {
    "action": "Stake 32 ETH",
    "chain": "Ethereum mainnet",
    "asset_change": "-32 ETH",
    "deposit_contract": "0x00000000219ab540356cBB839Cbe05303d7705Fa",
    "validator_pubkey": "0x...",
    "withdrawal_address": "0x...",
    "irreversible": true,
    "warnings": []
  },
  "policy_id": "staking-single-deposit-v0"
}
```

**输出字段**

```json
{
  "confirmed": true,
  "confirmed_by": "user",
  "confirmed_at": "2026-05-27T00:00:00Z",
  "policy_id": "staking-single-deposit-v0"
}
```

**STOP 条件**

- risk summary 缺少资产变化、权限变化、失败风险任一类；
- 用户拒绝或超时；
- policy id 与交易草稿不匹配；
- confirmation 内容与最终发送交易不一致。

### 3.5 `send_deposit_tx`

**等级**：E / irreversible write

**用途**：在已授权前提下广播 staking deposit 交易。

**输入 schema**

```json
{
  "signed_tx": "0x...",
  "chain_id": 1,
  "policy_id": "staking-single-deposit-v0",
  "confirmation_id": "string"
}
```

**输出字段**

```json
{
  "broadcast_status": "submitted",
  "tx_hash": "0x...",
  "chain_id": 1,
  "nonce": 0
}
```

**STOP 条件**

- 没有关联的人类确认或 session key 授权；
- signed tx 内容与已确认草稿不一致；
- chain id / to / value / method 任一项变化；
- simulation 结果过期；
- policy 超额度或过期；
- RPC timeout 后状态未知，不允许自动重发。

**timeout 处理**

RPC timeout 进入 `broadcast_unknown` 状态。系统必须先查 nonce、mempool、explorer、tx hash 或 provider 记录，确认交易是否已广播。未确认前不得重新发送 staking deposit。

## 4. Hermes 的 Auditor 插入点

Auditor 不替代 tool wrapper 的硬拦截。它主要住在：

```text
build_validator_deposit_tx -> simulate_deposit_tx -> Auditor risk_summary -> request_user_confirmation -> send_deposit_tx
```

Auditor 的工作是把不可读的交易草稿和模拟结果翻译成人能判断的摘要。tool wrapper 的工作是保证即使模型或用户界面出错，也不能绕过 chain / address / amount / policy / confirmation 检查。

## 5. 今日结论

Web3 Tool Use 把昨天的 workflow 落到"手臂"层：模型可以选择工具，但工具必须用确定性边界限制模型。Hermes 的安全性不来自让 agent 更聪明，而来自不给它万能手臂，并让所有不可逆动作必须穿过结构化 schema、simulation、Auditor 摘要、用户确认和写链硬拦截。
