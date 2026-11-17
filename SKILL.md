---
name: host-memory
description: 主机级、按目录 + 标签分片的记忆（INDEX / rules / logs 三层，另有固定的 misc 收纳），默认在 ~/host-memory。当需要了解某目录或项目是什么、怎么操作、怎么维护，或要记一条观察 / 待办 / 结论、巡检本机、配置同步 / chezmoi / 备份、压实过长记忆时使用。加载后先读 index，只读命中的 slug，绝不遍历。Host-wide, directory- and tag-partitioned memory with progressive disclosure.
---

# Host Memory

一套**主机级、按目录 + 标签分片**的记忆架构。只有三样东西：

- `index` —— 索引：把 scope（目录）和 tag（标签）映射到 slug。**加载后第一件事就是读它。**
- `rules/<slug>.md` —— 稳定：是什么、怎么操作、怎么维护。
- `logs/<slug>.md` —— 追加：发生过什么、要做什么（todo 直接写成一行，不另开文件）。积累到一定程度就**压实**（见下）。

零碎东西不另起名字：统一进一个固定的收纳 slug **`misc`**，INDEX 里那行只写"现在里面都有啥"。

目录用于"人在哪"自动命中；标签用于"事属于谁"跨目录命中。

skill 只管这套架构；具体记什么由用户自己配置。

## store 位置

默认 `~/host-memory/`（Windows：`%USERPROFILE%\host-memory`）。

```
~/host-memory/
├── INDEX.md          # 索引：scope / tag → slug（含一行固定的 misc 摘要）
├── rules/<slug>.md   # 杂项固定用 rules/misc.md
└── logs/<slug>.md    # 杂项固定用 logs/misc.md
```

## 加载后立刻做

1. 读 `INDEX.md`，不存在就当空。
2. 命中 slug：**scope** 与当前目录**相等**才算命中（嵌套子目录不命中父级 scope）；**tag** 按主题**跨目录**匹配。
3. 只读命中 slug 的 `rules/<slug>.md`；要看脉络 / 待办，再读 `logs/<slug>.md` 的尾部。
4. 命中多条就都读；**不**自动加载当前目录上级的 scope，除非用户明确指向它。**不读**未命中的 slug，**不遍历** `rules/`、`logs/`——标签只查 INDEX，不翻正文。
5. 判断"这条该不该进 `misc`"只看 INDEX 里 `misc` 那行的摘要；觉得沾边再去读 `rules/misc.md`。

## 写入

- 稳定知识 → `rules/<slug>.md`；观察 / 运维结果 / todo → `logs/<slug>.md` 追加一行。
- **时间不预设**：log 行、压实标记里的时间戳一律取**真实当前时间**——落笔前先查系统（`date` / 环境信息），绝不凭印象猜、不沿用上一次的值。不是不让记时间，而是不能编时间。
- **杂项**：不归任何目录 / 主题、又不够格单独成 slug 的东西，**统一进固定 slug `misc`** —— 稳定部分写 `rules/misc.md`，流水写 `logs/misc.md`。不新建 slug，也不在 INDEX 里加新行。
  - 写进去之后，**同时更新 INDEX 里 `misc` 那行的摘要**，让人不点进去就知道里面有什么（几条主题关键词即可）。
  - 同一主题在 `misc` 里长到需要"是什么 / 怎么操作 / 怎么维护"这种篇幅，就拆成独立 slug，把摘要里那部分挪走。
- 新建记忆 → 先在 `INDEX.md` 加一行（scope → slug + 标签），再建文件。
- 跨主题、不落在某个目录的知识：scope 写 `-`，只靠标签命中。
- 跨目录共性归最近的祖先 slug；整机级的归 `host-<host>`。
- **只读不建**；只有写入才创建文件。
- 只记**入口与位置**，不记密钥 / token / 密码明文。

## 压实（压缩 logs）

logs 只追加，会越积越长；读起来贵、噪声也大。压实把它重新收窄：**把仍有效的上提进 rules，把过时的删掉，只留近期脉络。**

- **只压实命中过的 slug**，绝不为压实去遍历。
- 稳定事实（仍适用的"是什么 / 怎么操作 / 怎么维护"）上提到 `rules/<slug>.md`；已完成的 todo、被推翻的结论、纯过程流水，删。
- **未完成 todo 一律保留**；拿不准还算不算有效的行，宁可留。
- 近期一段（比如最近几次记录）原样留着，当作脉络，不要清到只剩标记。
- 结尾追加一行压实标记，压了多少说清楚：
  ```markdown
  - 2026-10-05 14:20 compact  32 行 → 9 行；上提 4 条到 rules
  ```
- 压实是**唯一**允许重写 `logs/<slug>.md` 的操作；其余情况仍然只追加。INDEX 不因压实改动（`misc` 的摘要行除外：压实可能改变里面有什么）。

## 命名与标签

- slug：稳定、可读、唯一、kebab-case，优先用目录 / 项目本名（`agent-things`）。
- 主机级：`host-<主机名小写>`（`host-wuren`）。
- 标签：小写 kebab-case，一条 3–6 个，优先复用已有标签（`#config` `#wsl` `#backup`）。它是 INDEX 里的检索键，别写成一句话。
- 冲突时加父级限定。
- 杂项固定用 `misc` 这个名字，别再起别名（`misc` / `others` / `tmp` 只留一个）。

## 格式

`INDEX.md`（每行一条：`scope → slug — 说明  #标签`；scope 用 `-` 表示只靠标签命中）：

```markdown
# INDEX

- `D:\Project` → `d-project` — 本机所有项目的根  #projects #root
- `D:\Project\agent-things` → `agent-things` — …  #node #project
- `-` → `chezmoi` — 点文件 / 配置同步  #dotfiles #config #chezmoi
```

杂项在 INDEX 里就是普通的一行，只是说明部分要写"现在里面有什么"：

```markdown
- `-` → `misc` — 杂项：.NET 运行时家底与卸载记录、几条待办零碎  #misc #note #tidy
```

摘要保持一行、几个关键词就够，不进 `misc` 正文；`misc` 里内容变了就回来改这一行。具体内容仍写 `rules/misc.md`（稳定）和 `logs/misc.md`（追加）。

`rules/<slug>.md`：自由 markdown，写"是什么 / 怎么操作 / 怎么维护"。格式、字段、分节都按用户习惯，**不预设**。

`logs/<slug>.md`（只追加）：

```markdown
# log: agent-things

- 2026-10-04 20:40 todo  清一下 node_modules
- 2026-10-04 20:52 done  D: 盘剩 195G，暂不清
- 2026-10-04 21:03 obs   Fedora-44 是默认 WSL
```

## 与相邻 skill 的边界

- 项目内部探索过程 → project-memory（项目内 `./tmp/`）
- chezmoi / dotfiles / age 操作 → chezmoi-config-manager；本 skill 只记"配置由 chezmoi 管理 + 状态/入口"
- Windows 命令可靠性 → windows-shell-reliability、powershell-safe-invocation
- GUI 操作 → computer-use

本 skill 只做台账与索引，不重抄别的 skill。

## 惰性初始化

第一次写入时才建对应的 `rules/` / `logs/` 文件，并在 `INDEX.md` 补一行。
