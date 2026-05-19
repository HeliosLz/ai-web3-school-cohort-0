# AI × Web3 School — Cohort 0 Learning Journal

我在 [AI × Web3 School](https://aiweb3.school/zh/handbook/) 的个人学习仓库，也是我的 proof-of-work workspace。

- **Handbook**：https://aiweb3.school/zh/handbook/
- **WCB 课程页**：https://web3career.build/zh/programs/AI-Web3-School
- **打卡入口**：https://intensivecolearn.ing/en（用注册时的 GitHub 账号登录，左侧 "Check-in" 按钮）
- **后台镜像**（不要手改）：https://github.com/IntensiveCoLearning/AI-Web3-School/blob/main/notes/HeliosLz.md

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
| `submissions/` | 提交到 cohort 仓库 / 其他平台的内容存档 |
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

两个地方各司其职：

- **个人仓库（本仓库）= proof-of-work workspace**：深度笔记、实验代码、问题沉淀、Hackathon 项目。
- **intensivecolearn.ing 网站 = 打卡 sink**：点 "Check-in" 按钮 → 粘 Markdown → "Submit study notes"。后台自动同步到 cohort GitHub repo（`notes/HeliosLz.md`），**学员不要手改那个文件**。

每日节奏：

1. 学习 Handbook 当日章节，做题 / 实验。
2. 在 `daily/YYYY-MM-DD.md` 写完整笔记和 follow-up（**source of truth**）。
3. 从 daily note 中提炼一段精炼正文（200–500 字 Markdown）→ 粘到 intensivecolearn.ing 的 Check-in 表单 → Submit。
4. 实验代码进 `experiments/`，对 Handbook 的问题进 `handbook-feedback/`。
5. 本地 `git push` 个人仓库（每天至少一次）。
