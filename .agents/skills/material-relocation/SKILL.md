---
name: material-relocation
description: 量潮各仓之间的「材料归位」工作流——把暂存在 quanttide-work 档案仓 `data/profile` 里的材料（按〈主体／域／记忆类型〉归置），转出到对应主体仓或领域仓的对应子模块，并沿「目标子模块 → 目标仓 → quanttide-work 指针 → 伞仓指针」逐层提交推送。当用户说「把 xxx 迁到／归位到某仓」「这份材料该放哪个仓」「材料搬去 quanttide-xxx」时使用。
---

# 材料归位工作流

材料先住在 `data/profile`（档案仓，量潮工作台，按〈主体／域／记忆类型〉暂存），成熟后转出到正主仓；转出后档案不留副本，正主仓是这份材料唯一的事实源。语境仓 `data/context` 只留当日流水，不是本工作流的源；还只是流水的材料，先经 `context-to-profile` 工作流（`docs/gallery/workflows/`）收成档案材料再归位。

## 仓库拓扑

```
<伞仓>/                                  quanttide/quanttide，项目根的上上级
├── domains/quanttide-work               项目根（当前工作仓）
│   ├── data/profile/<主体>/<域>/<记忆类型>/   材料暂存（档案仓，本工作流的源）
│   ├── data/context/                    语境仓：当日流水与技能
│   └── 其余子模块
├── domains/<域>/                         公司领域仓，各带 data/ 与 docs/
└── default/<主体>/                       法人主体仓，同为 data/ 与 docs/
```

各仓实际路径用 `git rev-parse --show-toplevel` 逐层解析，不写死绝对路径。

## 路由

材料从 `data/profile/<主体>/<域>/<记忆类型>/x.md` 转出。目标仓按主体定：公司的领域材料进 `domains/<域>`；法人主体（quanttide-tech 等）的进 `default/<主体>`；创始人（quanttide-founder）的进创始人仓（在伞仓之外，结构见其 `.gitmodules`）。注意 `<域>` 可能就是这个项目（quanttide-work），那时目标仓是项目根自己。

落点按记忆类型定，类型名照搬：`data/` 下有 archive、brochure、context、history、insight、intention、journal、library、profile、report、roadmap；`docs/` 下有 bylaw、essay、gallery、handbook、specification、tutorial。每件是独立子模块，仓名形如 `quanttide-<kind>-of-<领域英文名>`。

落点前先 `ls` 源目录、`cat <目标仓>/.gitmodules` 核对——路由表可能滞后于仓结构。

## 执行步骤

五步链，每步提交后立即推送（AGENTS 约定）：

1. 落材料：`cp` 进目标子模块 → 子模块内 `git add -- <显式路径>` → commit → push。先写后删，迁移期间材料至少在一处。子模块若处于分离头指针，先 `git -C <子> checkout main`；push 被拒则 `pull --rebase` 后重推。
2. 目标仓指针：`git -C <目标仓> add -- data/<kind>` → commit → push。目标仓是项目根时，与第 4 步并作一次提交。
3. 删源：`git -C data/profile rm -r -- <源路径>`，在 data/profile commit → push。顺手删掉指向源材料的相对链接，不做跨仓补丁，来源文字保留。
4. work 指针：项目根 `git add -- data/profile` → commit → push。
5. 伞仓指针：`git -C <伞仓> add -- <目标仓相对路径> domains/quanttide-work` → commit → push。

## 提交信息风格

- 目标子模块：中文简句，如 `迁入 vibe-coding-workflow 洞察`、`材料迁至 quanttide-code 仓`
- 目标仓：`chore: 更新 data/insight 子模块指针（迁入知识工作材料）`
- 档案仓：`docs: 迁出 vibe-coding-workflow 洞察（归位 quanttide-code 仓）`
- quanttide-work：`chore: 更新 data/profile 指针（材料归位 quanttide-code）`
- 伞仓：`chore: 同步 quanttide-code、quanttide-work 指针（材料归位）`

## 硬性约束

- 显式 pathspec：永远 `git add -- <path>`，禁用 `git add -A`，免得卷入工作区里用户自己的未提交改动；动手前先 `git status -s` 摸底。
- `terminal` 的 `cd` 只能是项目根或子目录：操作项目外仓库用 `cd=<项目根>` 加 `git -C <该仓路径>`；`read_file` 拒绝项目外路径，改用 `cat`。
- 不传 `timeout_ms`：会导致工具请求 JSON 解析失败。
- 一条命令只做一步、控制在 1–2 个短调用：并行发长命令会 `tool input was not fully received`。
- 核对再提交：每次 commit 前看 `git diff --cached --stat`，确认只有预期文件。

## 完成后

汇报各层提交哈希（目标子模块／目标仓／档案仓／quanttide-work／伞仓），并指出这份材料现在的路径。若伞仓还有其他落后指针（`git -C <伞仓> status -s`），只提及、不擅自同步。
