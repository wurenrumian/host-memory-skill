---
name: host-memory
description: 主机级、按目录 + 标签分片的记忆（index / rules / logs 三层），默认存在 ~/host-memory，内容自配。当需要"了解某目录或项目是什么、怎么操作、怎么维护""记一条观察 / 待办 / 结论""巡检本机""配置同步 / chezmoi / 备份""记忆太长、该压实了"时使用。加载后先读 index，按当前目录或标签命中 slug，只读命中的 rules/logs，绝不遍历。Host-wide, directory- and tag-partitioned memory (index + rules + logs); progressive disclosure; logs compaction.
---

# Host Memory

一套**主机级、按目录 + 标签分片**的记忆架构。只有三样东西：

- `index` —— 索引：把 scope（目录）和 tag（标签）映射到 slug。**加载后第一件事就是读它。**
- `rules/<slug>.md` —— 稳定：是什么、怎么操作、怎么维护。
- `logs/<slug>.md` —— 追加：发生过什么、要做什么（todo 直接写成一行，不另开文件）。积累到一定程度就**压实**（见下）。

目录用于"人在哪"自动命中；标签用于"事属于谁"跨目录命中。

skill 只管这套架构；具体记什么由用户自己配置。

## store 位置

默认 `~/host-memory/`（Windows：`%USERPROFILE%\host-memory`）。

```
~/host-memory/
├── INDEX.md          # 索引：scope / tag → slug
├── rules/<slug>.md
└── logs/<slug>.md
```

## 加载后立刻做

1. 读 `INDEX.md`，不存在就当空。
2. 按**当前目录**命中 slug；主题导向的任务再按标签命中。
3. 只读命中 slug 的 `rules/<slug>.md`；要看脉络 / 待办，再读 `logs/<slug>.md` 的尾部。
4. **不读**未命中的 slug，**不遍历** `rules/`、`logs/`——标签只查 INDEX，不翻正文。

## 写入

- 稳定知识 → `rules/<slug>.md`；观察 / 运维结果 / todo → `logs/<slug>.md` 追加一行。
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
- 压实是**唯一**允许重写 `logs/<slug>.md` 的操作；其余情况仍然只追加。INDEX 不因压实改动。

## 命名与标签

- slug：稳定、可读、唯一、kebab-case，优先用目录 / 项目本名（`agent-things`）。
- 主机级：`host-<主机名小写>`（`host-wuren`）。
- 标签：小写 kebab-case，一条 3–6 个，优先复用已有标签（`#config` `#wsl` `#backup`）。它是 INDEX 里的检索键，别写成一句话。
- 冲突时加父级限定。

## 格式

`INDEX.md`（每行一条：`scope → slug — 说明  #标签`；scope 用 `-` 表示只靠标签命中）：

```markdown
# INDEX

- `D:\Project` → `d-project` — 本机所有项目的根  #projects #root
- `D:\Project\agent-things` → `agent-things` — …  #node #project
- `-` → `chezmoi` — 点文件 / 配置同步  #dotfiles #config #chezmoi
```

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
