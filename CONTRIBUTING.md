# 贡献规范 — NeuroRobotics Lab Vault

这是一份"怎么往知识库里加东西、怎么不互相踩脚"的约定。新成员先读一遍。

## 一、什么能进 Lab 库（三要素）

Lab 库是**知识资产**，不是草稿堆。进库的笔记必须满足：

1. **来源**：基于哪篇论文 / 哪次实验 / 哪个数据（文献笔记标 citekey，方法卡标相关论文）。
2. **结构**：用对应模板，章节齐全，别人能看懂。
3. **适用范围（scope）**：结论在什么前提/场景下成立，避免被误用。

你自己的翻译、吐槽、半成品请留在**个人库**；成熟了再"晋升"过来。

## 二、怎么加各类笔记

| 类型 | 目录 | 模板 | 文件名约定 |
|---|---|---|---|
| 文献笔记 | `literature/` | `literature-note` | **= citekey**（如 `liuLearningExoskeleton2024`） |
| 组会记录 | `meetings/` | `meeting-note` | `YYYY-MM-DD-主题` |
| 术语 | `glossary/` | `term-note` | 术语英文名（一术语一文件） |
| 公式/符号 | `notation/` | `formula-note` | 符号/概念名 |
| 方法卡片 | `methods/` | `method-card` | 方法名 |

插模板：Ctrl+P → `Templater: Insert template`。链接文献用 `[[@citekey]]`，链接术语/符号用 `[[名称]]`。

## 三、同步：开着 Obsidian 就行

配好 obsidian-git（见 README 第 3 步）后，**启动自动拉取 + 每 10 分钟自动 提交→拉取→推送**，你不用敲任何 git 命令。关 Obsidian 前确保最后同步过一次（或 Ctrl+P → `Git: Commit-and-sync`）。

## 四、直推 vs Pull Request

- **可以直推 `main`**（自动同步即可）：你自己的 `literature/`、`meetings/`、`projects/` 笔记。
- **必须走 PR + review**（影响全组的共用资产）：`glossary/`、`notation/`、`methods/`、`_templates/`、本文件 `CONTRIBUTING.md`。
  - 改这些时：新建分支 → 改 → 推分支 → 在 GitHub 提 Pull Request → 至少 1 人 review 后合并。
  - 改前最好在群里/组会说一声，避免撞车。

## 五、防冲突（照做几乎不会出问题）

1. **一文件一主**：别多人同时改同一个文件；大主题已拆成小文件就是为此。
2. **改前先拉**：启动自动拉取已覆盖；长时间没开过先 `Git: Pull`。
3. **小步多次提交** > 攒一大批。
4. **别随意重命名/移动文件**：会断别人的 `[[链接]]`。要改名先在组里说一声。
5. 图片放 `_assets/`，PDF 不进库（留 Zotero）。

## 六、FAQ / 出问题怎么办

**冲突了（conflict）？** 最省事——把这句发给 Claude Code：
```
我的 Lab Vault 仓库 git 同步出现冲突，请帮我查看冲突文件，保留双方有价值的内容、清理冲突标记，再重新提交并推送。
```
手动：obsidian-git 列出冲突文件，打开找 `<<<<<<<` / `=======` / `>>>>>>>`，保留需要的内容、删标记、存盘、再 `Git: Commit-and-sync`。

**推送被拒（rejected / non-fast-forward）？** 远端有新改动你没拉。先 `Git: Pull`（或 `Commit-and-sync` 会自动先拉）再推。

**图裂 / 看不到附件？** 设置 → 文件与链接 → 新附件默认位置 = `_assets/`；图片要提交进 git，PDF 不进。

**插件行为和别人不一样？** 确认 dataview / templater / obsidian-git 都启用；obsidian-git 同步设置和 Templater 模板目录每人要手动配一次（这些不随仓库下发）。

## 七、citekey 必须全组统一

全组用 Better BibTeX，citekey 格式 `auth.lower + shorttitle(3,3) + year`。这样同一篇论文每个人生成的 key 一致，文件名才能去重、`[[@citekey]]` 链接才对所有人有效。加入 Zotero 群组 https://www.zotero.org/groups/6589850/neuroroboticslab 后请确认你的 Better BibTeX 用的是这个格式。
