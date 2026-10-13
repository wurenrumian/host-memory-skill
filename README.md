# host-memory

一套**主机级、按目录 + 标签分片**的记忆架构。默认存放在 `~/host-memory/`（Windows：`%USERPROFILE%\host-memory`），内容由用户自配。

它只有三样东西：

- `INDEX.md` —— 索引：把 scope（目录）和 tag（标签）映射到 slug。**加载后第一件事就是读它。**
- `rules/<slug>.md` —— 稳定：是什么、怎么操作、怎么维护。
- `logs/<slug>.md` —— 追加：发生过什么、要做什么（todo 写成一行，不另开文件）。

目录用于"人在哪"自动命中；标签用于"事属于谁"跨目录命中。skill 只管架构，具体记什么由用户决定。

```
~/host-memory/
├── INDEX.md          # 索引：scope / tag → slug
├── rules/<slug>.md
└── logs/<slug>.md
```

## 何时用

- 想"了解某目录或项目是什么、怎么操作、怎么维护"
- 要"记一条观察 / 待办 / 结论"
- 巡检本机、配置同步 / chezmoi / 备份
- 记忆太长、该压实了

## 加载后立刻做（渐进披露）

1. 读 `INDEX.md`，不存在就当空。
2. 命中 slug：**scope** 与当前目录**相等**才算命中（嵌套子目录不命中父级）；**tag** 按主题**跨目录**匹配。
3. 只读命中 slug 的 `rules/<slug>.md`；要看脉络 / 待办，再读 `logs/<slug>.md` 尾部。
4. 命中多条就都读；**不读未命中的 slug，不遍历** `rules/`、`logs/`——标签只查 INDEX，不翻正文。

## 写入与压实

- 稳定知识 → `rules/`；观察 / 运维结果 / todo → `logs/` 追加一行。
- 新建记忆先在 `INDEX.md` 加一行，再建文件；**只读不建**，写入才创建文件。
- 只记**入口与位置**，不记密钥 / token / 密码明文。
- **压实**只对命中过的 slug 做：把仍有效的上提进 `rules/`，过时的删掉，未完成 todo 一律保留，近期一段原样留作脉络；结尾追加一行压实标记。压实是唯一允许重写 `logs/` 的操作。

## 命名

- slug：稳定、可读、唯一、kebab-case，优先用目录 / 项目本名；主机级用 `host-<主机名小写>`。
- 标签：小写 kebab-case，一条 3–6 个，优先复用已有标签，作为 INDEX 里的检索键。

## 与相邻 skill 的边界

- 项目内部探索过程 → **project-memory**（项目内 `./tmp/`）
- chezmoi / dotfiles / age 操作 → **chezmoi-config-manager**（本 skill 只记"由 chezmoi 管理 + 状态/入口"）
- Windows 命令可靠性 → windows-shell-reliability、powershell-safe-invocation
- GUI 操作 → computer-use

本 skill 只做台账与索引，不重抄别的 skill。

## 安装

把本目录复制或软链到 agent 的 skills 目录，目录名须与 frontmatter 的 `name:`（`host-memory`）一致：

```powershell
Copy-Item -Recurse . "$env:USERPROFILE\.config\opencode\skills\host-memory"
```

## 目录内容

- `SKILL.md` — skill 本体。
- `.gitattributes` — 强制 LF 行尾。
