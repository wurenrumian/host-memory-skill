---
name: host-memory
description: 主机级、按目录分片的记忆架构，只含三样东西：index（索引）、rules（稳定）、logs（追加）。加载本 skill 后先读 index，按当前目录做最长前缀匹配，只读命中的那一条 rules/logs，绝不遍历。默认 store 在 ~/host-memory，内容由用户自行配置。用于"了解某目录/项目是什么、怎么操作、怎么维护""记一条观察/待办""巡检本机""配置同步 / chezmoi / 备份"这类需要主机级按目录记忆的场合。Host-wide, directory-partitioned memory (index + rules + logs); progressive disclosure; read only the matching entry.
---

# Host Memory

一套**主机级、按目录分片**的记忆架构。只有三样东西：

- `index` —— 索引：把目录（scope）映射到条目（slug）。**加载后第一件事就是读它。**
- `rules/<slug>.md` —— 稳定：是什么、怎么操作、怎么维护。
- `logs/<slug>.md` —— 追加：发生过什么、要做什么（todo 直接写成一行，不另开文件）。

skill 只管这套架构；具体记什么由用户自己配置。

## store 位置

默认 `~/host-memory/`（Windows：`%USERPROFILE%\host-memory`）。

```
~/host-memory/
├── INDEX.md          # 索引：scope → slug
├── rules/<slug>.md
└── logs/<slug>.md
```

## 加载后立刻做

1. 读 `INDEX.md`，不存在就当空。
2. 按**当前目录**做最长前缀匹配，得到 slug（规则见下）。
3. 只读 `rules/<slug>.md`。
4. 要看脉络 / 待办时，再读 `logs/<slug>.md` 的尾部。
5. **不读**其它 slug，**不遍历** `rules/`、`logs/`。

### 匹配规则

- 归一化路径后按**目录段**做最长前缀匹配：`D:\Project` 命中 `D:\Project\a`，但不命中 `D:\Projects`。
- 没有命中就回退到最近的祖先条目；再没有就当无记忆。
- 命中的文件缺失即跳过，不报错、不猜。

## 写入

- 稳定知识 → `rules/<slug>.md`。
- 观察 / 运维结果 / todo → `logs/<slug>.md` 追加一行。
- 要给一个新目录建记忆 → 先在 `INDEX.md` 加一行（scope → slug），再建文件。
- 跨目录共性归最近的祖先 slug；整机级的归 `host-<host>`。
- **只读不建**；只有写入才创建文件。
- 只记**入口与位置**，不记密钥 / token / 密码明文。

## 命名

- slug：稳定、可读、唯一、kebab-case，优先用目录 / 项目本名（`agent-things`）。
- 主机级：`host-<主机名小写>`（`host-wuren`）。
- 冲突时加父级限定。

## 格式

`INDEX.md`（每行一条）：

```markdown
# INDEX

- `D:\Project` → `d-project` — 本机所有项目的根
- `D:\Project\agent-things` → `agent-things` — …
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
