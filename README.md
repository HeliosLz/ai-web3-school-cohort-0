# AI × Web3 School — Cohort 0 Learning Journal

我在 [AI × Web3 School](https://aiweb3.school/zh/handbook/) 的个人学习仓库，也是我的 proof-of-work workspace。

- **Handbook**：https://aiweb3.school/zh/handbook/
- **WCB 课程页**：https://web3career.build/zh/programs/AI-Web3-School
- **WCB Learning（打卡入口）**：https://web3career.build/zh/programs/AI-Web3-School#tab=learning

## 目录结构

| 目录 / 文件 | 用途 |
|---|---|
| `profile.md` | 我的学习画像（背景、目标、节奏） |
| `learning-plan.md` | 整体学习计划与里程碑 |
| `daily/` | 每日学习笔记，文件名 `YYYY-MM-DD.md` |
| `tasks/` | 课程任务拆解和完成记录 |
| `experiments/` | 小型实验、原型、代码片段 |
| `handbook-feedback/` | 对 Handbook 的反馈：错别字、过期内容、结构建议 |
| `hackathon/` | Hackathon 项目相关 |
| `submissions/` | 已经提交到 WCB / 打卡平台的内容存档 |
| `templates/` | 笔记模板 |

## 隐私与安全

本仓库为 **public**，**不要**提交：

- API key / secret / token
- 钱包助记词、私钥、keystore
- 未公开的联系方式、邮箱、电话
- 内部会议链接、未公开的合作方信息
- 他人个人数据

WCB Agent API 的 secret key 只放在本地环境变量（`WCB_AGENT_SECRET_API_KEY`）或 secrets manager 中，绝不写进任何文件。

## 工作流

每日学习节奏：

1. 早晨 / 晚间打开学习 Agent，让它读取 Handbook + WCB Learning 页面。
2. Agent 生成 `daily/YYYY-MM-DD.md` 草稿和打卡内容。
3. 学习、做题、做实验。
4. 把今日笔记和打卡草稿确认无误后提交：本地 `git push`，WCB 平台手动确认打卡。
5. 学习中遇到的问题 → `handbook-feedback/` 沉淀。
