# NeuroRobotics Lab — 共享知识库（Lab Vault）

这是 **NeuroRobotics Lab** 的共享知识库（Obsidian 仓库 + GitHub 同步）。它把散落的论文理解、术语、公式、方法、会议结论沉淀成**可搜索、可链接、可版本管理**的实验室知识资产，解决：同一篇论文重复读、术语翻译不统一、公式符号混乱、会议结论散落、师兄师姐毕业知识断层、新生不知从哪看。

- **GitHub 仓库**：https://github.com/GeStoi/NeuroRoboticsLab_Vault
- **Zotero 群组**：https://www.zotero.org/groups/6589850/neuroroboticslab （`neuroroboticslab`，群组 ID `6589850`）

> **两库分工**：你自己电脑上随便乱记、翻译、吐槽的"个人库"，和这个整理好、别人看得懂、能复用的"Lab 库"是**分开的**。本仓库 = Lab 库。论文先在个人库读熟，成熟后再"晋升"进这里。

---

## 🚀 最快上手：一句话让 AI 帮你接入

装了 **Claude Code** 的话，复制下面这一句发给它即可（它会读本 README 自动干）：

```
请按这个仓库 README 的「📋 给 AI 的接入指令」整段执行，帮我把 NeuroRobotics Lab 的知识库接入到本机。仓库地址 https://github.com/GeStoi/NeuroRoboticsLab_Vault.git ，克隆到 C:\Document\LabVault（不要放进任何云盘同步目录）。
```

AI 能做的会自动做，做不了的（装 Obsidian 插件、浏览器登录）会明确提示你。**不用 AI** 就往下看「🧑‍💻 手动接入」，每步都能复制粘贴。

---

## 📋 给 AI 的接入指令

> 成员把上面那句话发给 Claude Code 即可；本节是给 AI 的待办清单，人也能照做。
> AI 请按顺序执行，每步报告进度；遇到需要用户手动操作（GUI / 交互式登录）时明确提示并等待。

1. **检查前置工具**：确认 `git`、Obsidian、Zotero 已装；`git` 缺则 `winget install Git.Git`。建议装 GitHub CLI：`winget install GitHub.cli`（用于一键登录）。Obsidian/Zotero 是 GUI 程序，缺则提示用户自行下载，别静默装。
2. **GitHub 认证**（交互式）：提示用户运行 `gh auth login` 按浏览器完成；或在后续 `git push` 时由 Git 凭据管理器弹窗登录。
3. **克隆仓库**：`git clone https://github.com/GeStoi/NeuroRoboticsLab_Vault.git "C:\Document\LabVault"`（或用户指定路径）。
   - ⚠️ **绝不要**克隆到坚果云 / OneDrive / Dropbox 等云盘同步目录——云盘 + git 双同步会损坏仓库。
4. **设置本仓库 git 身份**（只设本仓库）：`git config user.name "<真实姓名>"`、`git config user.email "<学校邮箱>"`。向用户询问这两项。
5. **GUI 步骤（AI 无法代点，提示用户做）**：用 Obsidian「打开文件夹作为仓库」选中克隆目录 → 首次打开允许并安装社区插件（dataview / templater / obsidian-git）。
6. **配 obsidian-git 自动同步**：待插件装好后，按「🔄 同步设置」表帮用户配置（或写进 `.obsidian/plugins/obsidian-git/data.json`，键名以该版本插件为准；不确定就让用户照表手点）：启动时拉取=开、自动 commit-and-sync 间隔=10 分钟、先拉后推、提交信息 `vault backup: {{date}}`。
7. **配 Templater 模板目录**：Obsidian → 设置 → Templater → Template folder location 设为 `_templates`（该设置属个人插件配置，不随仓库下发，需手动设一次）。
8. **（可选）连 Zotero**：提示用户加入群组 https://www.zotero.org/groups/6589850/neuroroboticslab ，并在 Zotero → 设置 → 高级 勾选"允许其他应用通信"。
9. **收尾**：`git pull` 确认能拿到最新内容，告知用户"以后开着 Obsidian 每 10 分钟自动同步"。

---

## 🧑‍💻 手动接入（不用 AI）

### 0. 前置安装（一次）
| 工具 | 用途 | 安装 |
|---|---|---|
| Obsidian | 知识库本体 | https://obsidian.md |
| Git | 同步引擎 | https://git-scm.com 或 `winget install Git.Git` |
| GitHub CLI（推荐） | 一键登录 | `winget install GitHub.cli` |
| GitHub Desktop（可选） | 图形界面 | https://desktop.github.com |
| Zotero（可选） | 文献/PDF | https://zotero.org |

### 1. 登录并克隆
```bash
gh auth login            # 浏览器登录一次（或克隆/推送时由凭据管理器弹窗）
git clone https://github.com/GeStoi/NeuroRoboticsLab_Vault.git "C:\Document\LabVault"
git config user.name  "你的真实姓名"     # 在仓库目录内执行
git config user.email "你的学校邮箱"
```
> ⚠️ 克隆路径**不要**放进坚果云 / OneDrive / Dropbox 等云盘目录。GitHub 本身就是云端备份。

不熟命令行可用 **GitHub Desktop**：Clone repository → 填仓库地址 → 选本地路径 → Clone。

### 2. 在 Obsidian 打开并装插件
打开 Obsidian →「打开文件夹作为仓库」→ 选克隆目录 → 允许启用插件 → 确认 **dataview / templater / obsidian-git** 已装启用（仓库带清单会自动提示）。

### 3. 🔄 配置 obsidian-git 自动同步（关键）
设置 → Git 插件：

| 设置项（标签可能因版本略异） | 推荐值 |
|---|---|
| Pull updates on startup（启动时拉取） | ✅ 开 |
| 自动 commit-and-sync / Vault backup 间隔（分钟） | `10` |
| Auto pull interval（分钟） | `10` |
| commit-and-sync 顺序 / Pull before push | 默认 commit→pull→push |
| Commit message on auto backup | `vault backup: {{date}}` |
| List filenames in commit body | ✅ 开 |

配好后**只要开着 Obsidian 就后台每 ~10 分钟自动双向同步**；想立刻同步按 Ctrl+P → `Git: Commit-and-sync`。

### 4. 配置 Templater 模板目录
设置 → Templater → Template folder location → 填 `_templates`。

### 5.（可选）连 Zotero
加入群组 https://www.zotero.org/groups/6589850/neuroroboticslab ；Zotero → 设置 → 高级 → 勾"允许本机其他应用与 Zotero 通信"。Lab 库**不存 PDF**，只写蒸馏笔记并用 citekey 引用。

---

## 📁 目录结构
```
├── README.md / CONTRIBUTING.md / START-HERE.md   # 说明文档
├── _assets/        # 图片等小附件（PDF 不放这）
├── _templates/     # 5 个统一模板（Templater 调用）
├── literature/     # 文献笔记（一篇一文件，文件名 = citekey）
├── glossary/       # 术语表（一术语一文件）
├── notation/       # 公式符号（一条一文件）
├── methods/        # 方法卡片（一方法一文件）
├── meetings/       # 会议结论（一次一文件）
├── projects/       # 项目判断、实验启发
├── onboarding/     # 新生引导、阅读路径
├── maps/           # dataview 自动索引（文献/术语/符号/方法）
└── archive/        # 毕业成员/旧项目归档
```
新人从哪开始？→ `START-HERE.md`、`onboarding/`、`maps/`。

---

## ✍️ 日常工作流（详见 CONTRIBUTING.md）
- **加文献**：`literature/` 新建文件，文件名 = citekey → Ctrl+P → `Templater: Insert template` → `literature-note` → 填来源/结构/适用范围。
- **记组会**：`meetings/` 新建 `YYYY-MM-DD-主题.md` → `meeting-note` 模板 → 重点写决策和 Action items。
- **补术语/公式/方法**：在 `glossary/` `notation/` `methods/` **新建独立小文件**，`maps/` 自动收录。

---

## 🔄 同步与协作规则（详见 CONTRIBUTING.md）
- 开着 Obsidian 即后台自动同步，基本无感。
- **日常直推**：个人文献/组会/项目笔记 → 自动进 `main`。
- **资产级文档走 PR**：`glossary/` `notation/` `methods/` `_templates/` `CONTRIBUTING.md` → 分支 + Pull Request + ≥1 人 review。
- **防冲突**：一文件一主、改前先拉、小步多次、别乱重命名。冲突解决见 CONTRIBUTING 的 FAQ（含一句话 AI 指令）。

---

## 📌 约定速查
- **citekey 格式**：全组统一 `auth.lower + shorttitle(3,3) + year`（Better BibTeX）；文献笔记文件名 = citekey。
- **命名**：英文、无编号；术语/公式/方法一条一文件。
- **附件**：图片进 `_assets/`，PDF 留 Zotero（`*.pdf` 已被 git 忽略）。
- **三要素**：进 Lab 的笔记必须有 **来源 / 结构 / 适用范围**。
- **不放云盘**：仓库别放坚果云/OneDrive/Dropbox 同步目录。

有问题在课题组群里问，或直接把问题丢给 Claude Code。
