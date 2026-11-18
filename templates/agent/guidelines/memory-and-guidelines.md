---
name: memory-and-guidelines
description: 何时写 memory、怎样用任务页交接、何时更新长期规则和工具索引
category: agent
---

# agent/ 目录的维护规则

`agent/` 是 agent 自己的空间，可以放手更新，不需要事先征求同意。每次改动后运行 `python3 agent/tools/scripts/reindex.py`。

## memory：以会话为单位的记忆

### 什么时候写

时机由 agent 判断，原则是**在检查点增量写，不要攒到最后**。会话可能突然中断，上下文也可能被压缩，攒到最后很容易漏。

- **检查点**：做出一个决定、产出或整理了文件、解决了一个问题、发现了重要信息之后，都算检查点。
- **开始一段长任务之前**（例如大规模重构、批量处理），先把目前为止的进展记下来。
- **对话已经很长时**（工具调用很多、来回讨论很多轮），主动先记一笔，再继续。
- **上下文被压缩后**，hook 会提醒你核对，没记全的现在补上。
- 同一会话第一次写时新建文件，之后都是**更新同一个文件**，不要另开一条。
- 纯问答、没有留下任何结论的会话不写。

### 文件名
`yymmdd-three-words-description.md`，例如 `260128-competitor-pricing-scan.md`。
- 用英文小写，单词之间用 `-` 连接，三个词左右。memory 是给 agent 用的，所以不跟随项目的文件名语言。
- 同一天如果已有同名文件，加 `-b`、`-c` 后缀。

### 格式
复制 `agent/memory/_template.md`。frontmatter 必须写：
- `description`：一句话摘要，会出现在索引里，要让人只看这一句就能判断用不用打开；
- `tags`：从下面的常用标签里选，确实需要时再加新的。

常用标签：`decision` `research` `design` `writing` `build` `bug` `delivery` `files` `feedback` `planning`

### 怎么查
1. 先看 `agent/memory/INDEX.md`，按标签或日期挑出相关条目。
2. 需要时用 grep 在 `agent/memory/` 里搜关键词。
3. 只打开相关的几条，不要整目录加载。

## 可选任务页：跨会话交接

预计要跨会话继续、需要交给另一个 agent，或用户明确要求留交接时，可以用任务页。一次会话能完成并验收的小改动继续走原来的流程。

1. 复制 [任务页模板](../tasks/_template.md) 到同目录，命名为 `yymmdd-简短任务名.md`。同一任务一直更新同一页。
2. 写明目标、范围、可验证的验收条件和适用的规则。在 [PROJECT.md](../PROJECT.md) 的「当前状态」下链接这张任务页，方便新会话按任务找到它。
3. 在检查点更新任务页里的进度、验收证据、阻塞和下一步。按原有规则写本次会话的 memory 时，在两边加入相对链接：任务页链接到 `../memory/<会话文件>.md`，memory 链接回 `../tasks/<任务文件>.md`。任务页保留最新状态，memory 保留本次会话的决定、经过和原因。
4. 恢复工作时，从项目简报打开对应任务页，按需点开相关规则和 memory。验收后在页上记下结果，移除项目简报里的进行中链接，保留任务页原路径供历史记录回查。

## guidelines：沉淀下来的长期规则

### 什么时候新增或修改
- 用户明确说"记成规则"、"以后都这样"。
- 用户纠正了 agent 的某个做法：纠正的内容本身就是规则。
- 同一条经验在 memory 里出现两次以上：把它升级成 guideline，并在原 memory 里注明"已升级到 guidelines/xxx.md"。
- 已有规则和现实不符（比如用户的做法变了）：直接改规则，在当次 memory 里记一笔。

### 格式
- 一个文件讲一个主题。frontmatter 写 `name`、`description`、`category`。
- 文件多了以后按 category 建子文件夹，reindex 会按子文件夹分组。
- 规则要写出**为什么**，这样遇到边界情况时能自己判断。

## tools：项目专属工具

| 类型 | 放哪 | 说明 |
|---|---|---|
| skill | `agent/tools/skills/<name>/SKILL.md` | 已软链接到 `.claude/skills`（Claude Code）和 `.agents/skills`（Codex），两边都能自动识别 |
| subagent | `agent/tools/agents/<name>.md` | 已软链接到 `.claude/agents`（Claude Code 格式） |
| 脚本 | `agent/tools/scripts/` | 文件开头第一行注释写用途，reindex 会读这一行 |
| 线上工具、外部服务 | `agent/tools/links.md` | 写清名称、链接、用途、怎么登录或调用（不写密钥） |

- 同一段操作或提示词用到第二次，就考虑把它做成工具。
- 只对本项目有用的才放这里；通用工具应该装到全局。
- 密钥、token 一律不进仓库，只写"去哪里取"。
