# Learning Plan

> 整体学习路径的活文档。具体每日任务在 `daily/`，具体课程任务在 `tasks/`。
> 来源：[Handbook](https://aiweb3.school/zh/handbook/) + [Cohort README](https://github.com/IntensiveCoLearning/AI-Web3-School)

## 北极星

在 AI × Web3 School Cohort 0 结束前（**2026-06-14**），**完成 1 个 AI × Web3 交叉的 Hackathon-friendly demo**，并产出：

- 可运行的 demo（仓库 + README + 演示视频或截图）
- 项目复盘 writeup
- 至少 5 条 Handbook feedback

## Cohort 时间表

| 项目 | 日期 |
|---|---|
| 报名期 | 2026-05-06 → 2026-05-17（已截止） |
| 学习期 | **2026-05-18 → 2026-06-14**（4 周） |
| 请假规则 | 每周允许 2 天 |
| 打卡 sink | [cohort 仓库 notes/HeliosLz.md](https://github.com/IntensiveCoLearning/AI-Web3-School/blob/main/notes/HeliosLz.md) |
| 社区 | Telegram https://t.me/aiweb3school |
| Registration Guide | [Notion](https://www.notion.so/lxdao/Registration-Guide-283dceffe40b80e883fddfa045afef5e) |
| Check-in Guide | [Notion](https://www.notion.so/lxdao/Check-in-Guide-283dceffe40b80ad9936db5a15468eb5) |

## 每周主题（cohort 官方）

- **Week 1（5.18–5.24）**：建立 AI 与 Web3 的共同语言
  - AI 侧：LLM、Prompt、Workflow、Agent、Tool Use、AI Coding
  - Web3 侧：账户、钱包、地址、签名、交易、Gas、Smart Contract、Testnet、Block Explorer
  - 完成至少 1 个 AI 工具实践 + 1 个 Web3 testnet/合约交互
  - 串起最小链条：user intent → AI planning → human review → wallet authorization → on-chain execution → verifiable record
- **Week 2（5.25–5.31）**：探索 AI × Web3 交叉
  - 拆解方向：Agentic Commerce / Payment、Dev Tooling、AI Security / Privacy、AI × Governance / Coordination、Open Track
  - 学会判断"真需要双侧"与"两个 buzzword 拼接"的差别
  - 选 1 个方向，回答 5 问：谁发起、谁执行、谁付钱、谁验证、谁担风险
  - 产出：方向、问题地图、风险拆解、Demo 路径
- **Week 3（6.01–6.07）**：深化练习 + Hackathon 准备
  - 强化方向所需技能
  - Hackathon 赛道开启、组队、技术路径选型
  - 明确哪些自动化、哪些 human-in-the-loop、哪些上链
  - 产出：proposal、角色、技术路径、Hackathon checklist
- **Week 4（6.08–6.14）**：Build + Submit + Demo
  - Hackathon focused build
  - 准备 repo、README、demo link、视频、testnet 地址、交易 hash
  - 项目演示与社区反馈

## Handbook 章节 → 阶段映射

> 学员现状：AI 有基础、Web3 有基础、会基础脚本。可以跳过最基础的 LLM / Prompt / Wallet 入门，直奔中间层与 Bridge。

### Week 1（推荐 6–7 节，每日 1 节）

| 日期 | Handbook 节点 | 备注 |
|---|---|---|
| 5.18 ✅ | [Agent](https://aiweb3.school/zh/handbook/ai/agent/) | 已完成（Hermes Agent 笔记） |
| 5.19 | [Tool Use / MCP](https://aiweb3.school/zh/handbook/ai/mcp/) | Agent 调工具的协议层 |
| 5.20 | [Smart Contract](https://aiweb3.school/zh/handbook/web3/smart-contract/) | 跳过 Wallet 入门直接进合约 |
| 5.21 | [Account Abstraction](https://aiweb3.school/zh/handbook/web3/account-abstraction/) | Agent Wallet 的前置 |
| 5.22 | [Frameworks](https://aiweb3.school/zh/handbook/ai/frameworks/) | LangChain / LangGraph / Agents SDK 选型 |
| 5.23 | [Evaluation](https://aiweb3.school/zh/handbook/ai/evaluation/) | Agent 行为可测试 |
| 5.24 | **Week 1 复盘** | 跑通最小链条 + 选 Week 2 候选方向 |

> 请假预算：5.23 / 5.24 任选 2 天可请。

### Week 2（5.25–5.31）—— Bridge 章节

按选定方向自选 5 个节点，候选池：

- [Chain-aware Context](https://aiweb3.school/zh/handbook/bridge/chain-aware-context/)
- [Web3 Tool Use](https://aiweb3.school/zh/handbook/bridge/web3-tool-use/)
- [Agent Workflow](https://aiweb3.school/zh/handbook/bridge/agent-workflow/)
- [Agent Wallet](https://aiweb3.school/zh/handbook/bridge/agent-wallet/)
- [Machine Payment](https://aiweb3.school/zh/handbook/bridge/machine-payment/)
- [Settlement & Escrow](https://aiweb3.school/zh/handbook/bridge/settlement-and-escrow/)
- [Agent Identity](https://aiweb3.school/zh/handbook/bridge/agent-identity/)
- [Verifiable AI](https://aiweb3.school/zh/handbook/bridge/verifiable-ai/)
- [AI Security](https://aiweb3.school/zh/handbook/bridge/ai-security/)

Week 2 末写 `hackathon/ideation.md`，列 2–3 个候选方向 + 5 问回答。

### Week 3（6.01–6.07）—— 锁定方向 + 前沿探索

从下面 6 个 track 选 1，对应 Week 4 Hackathon：

- [Agentic Commerce](https://aiweb3.school/zh/handbook/tracks/agentic-commerce/)
- [Dev Tooling](https://aiweb3.school/zh/handbook/tracks/dev-tooling/)
- [Wallet / Permission](https://aiweb3.school/zh/handbook/tracks/wallet-permission/)
- [AI Security](https://aiweb3.school/zh/handbook/tracks/ai-security/)
- [Governance](https://aiweb3.school/zh/handbook/tracks/governance/)
- [Open Track](https://aiweb3.school/zh/handbook/tracks/open-track/)

Week 3 末完成 proposal + 技术路径 + tracer-bullet 跑通。

### Week 4（6.08–6.14）—— Demo

每天聚焦 build，最后 2 天准备 demo materials（视频 / README / testnet 交易 hash）。

## 节奏约束

- 每日 ~1 小时 → 适合每日 1 个 Handbook 节点 + 必要时 30 分钟实验
- 周末可补做累积内容 / 推进 hackathon 项目
- 请假预算：每周 2 天，全程上限 8 天

## 提交规则

每天的工作流：

1. 读当日 Handbook 节点 + 必要实验 → 完整笔记落 `daily/YYYY-MM-DD.md`
2. 从 daily 提炼精炼打卡正文（即 daily 文件末尾的"打卡草稿"块）
3. 打开 [intensivecolearn.ing](https://intensivecolearn.ing/en) → 用注册时的 GitHub 账号登录 → AI × Web3 School → 左侧 **Check-in** → 粘贴打卡正文提交
   - 站点后台会镜像到 https://github.com/IntensiveCoLearning/AI-Web3-School/blob/main/notes/HeliosLz.md（**不要手改这个文件**）
4. 实验代码 → `experiments/`；Handbook 问题 → `handbook-feedback/`
5. 个人仓库 `git push`
