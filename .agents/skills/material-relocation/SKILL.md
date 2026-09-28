---
name: material-relocation
description: 量潮各仓之间的「材料归位」工作流——把草稿从 quanttide-work 的 data/context 中转站搬到目标域仓的对应子模块，并沿「内容仓 → 域仓指针 → quanttide-work 指针 → 伞仓指针」逐层提交推送。当用户说「把 xxx 迁到 / 归位到 / move to 某仓」「材料搬去 quanttide-xxx」「这个该放哪个仓」时使用。
---

# 材料归位工作流

把 `quanttide-work/data/context/quanttide/<域>/` 下的草稿搬到各域仓的正主位置，全链路提交推送。中转站只收草稿，材料的唯一事实源永远在目标域仓。

## 仓库拓扑

```
/home/iguo/repos/quanttide                      伞仓（quanttide/quanttide）
└── domains/quanttide-work                      项目根（当前工作仓）
    ├── data/context/quanttide/<域>/            中转站，按域预分类的草稿
    ├── data/profile/                           材料仓（留在本仓，不外迁）
    └── 22 个子模块
└── domains/<域>/                               各域仓，如 quanttide-data、quanttide-code
    ├── data/<kind>  或 docs/<kind>             每个都是独立 git 子模块
    └── 子模块内容统一放 default/ 目录
```

## 路由表

源目录 `data/context/quanttide/<域>/<kind>/x.md` → 目标 `domains/<域>/<该 kind 对应路径>/default/x.md`：

| kind | 目标路径 | 子模块仓名后缀 |
|---|---|---|
| context | `data/context` | -context-of-<领域> |
| journal | `data/journal` | -journal-of-<领域> |
| insight | `data/insight` | -insight-of-<领域> |
| roadmap | `data/roadmap` | -roadmap-of-<领域> |
| tutorial | `docs/tutorial` | -tutorial-of-<领域> |
| profile | 留在 quanttide-work `data/profile` | -profile-of-knowledge-work |

先 `ls` 源目录与 `cat domains/<域>/.gitmodules` 核对实际路径，路由表可能滞后于仓结构。内容放子模块的 `default/` 下（若目标子模块已有该惯例）。

## 执行步骤

五步链，每步提交后立即推送（AGENTS 约定）：

1. **落材料**：`cp` 到目标子模块 → 在子模块内 `git add <显式路径>` → commit → push。
   - 仓名示例：`quanttide-context-of-data-engineering`、`quanttide-journal-of-data-engineering`。
   - 子模块若处于分离头指针：先 `git -C <sub> checkout main` 再操作；push 被拒则 `pull --rebase` 后重推。
2. **域仓指针**：`git -C domains/<域> add -- data/<kind> ...` → commit → push。
3. **删源**：`git -C data/context rm -r -- quanttide/<域>` → commit → push。
4. **work 指针**：项目根 `git add -- data/context` → commit → push。
5. **伞仓指针**：`git -C /home/iguo/repos/quanttide add -- domains/quanttide-work domains/<域>` → commit → push（若本次动了多个域，全部列入）。

## 提交信息风格

- 内容仓：中文简句，如 `迁入 2026-09-28 日志`、`材料迁至 quanttide-data 仓`
- 域仓：`chore: 更新 data/context、data/journal 子模块指针（迁入工作区材料）`
- quanttide-work：`docs: 更新 data/context 指针（材料迁至 quanttide-data 仓）`
- 伞仓：`chore: 同步 quanttide-work、quanttide-data 指针（材料归位）`

## 硬性约束

- **显式 pathspec**：永远 `git add -- <path>`，禁止 `git add -A`，避免卷入工作区里用户自己的未提交改动。动手前先 `git status -s` 摸底。
- **`terminal` 的 `cd` 只能是项目根或子目录**：操作项目外仓库用 `cd=quanttide-work` + `git -C /home/iguo/...`；`read_file` 拒绝项目外路径，用 `cat`。
- **不传 `timeout_ms`**：会导致工具请求 JSON 解析失败。
- **一条命令只做一步、控制在 1–2 个短调用**：并行发长命令会 `tool input was not fully received`。
- **核对再提交**：每次 commit 前看 `git diff --cached --stat`，确认只有预期文件。

## 完成后

汇报四层提交哈希（内容仓 / 域仓 / work / 伞仓），并指出目标子模块里那条内容现在的路径。若伞仓还有其他落后指针（`git -C /home/iguo/repos/quanttide status -s`），只提及、不擅自同步。
