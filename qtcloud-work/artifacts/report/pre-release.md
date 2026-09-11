# 报告：pre-release

## 版本

2026-09-11，定 pre-release 的版本号并收 Unreleased。

- **版本号**：`0.1.0-pre.1`。Cargo.toml 原为 `0.1.0`，此前没有任何 tag（`git tag` 空），故这是 0.1.0 的第一个预发布，`pre` 序号取 1。
- **CHANGELOG**：把 `## [Unreleased]` 收成 `## [0.1.0-pre.1] - 2026-09-11`，按「新增 / 变更 / 修复」列出这一次的变更（工作流与任务两族动作、三类判据、工作区四个只读动作、全局上下文与输出契约、文档与测试、`health` 保留；模块重排、任务与产物分家、数据仓默认；人的步骤挂 agent 判据、审查 ✗ 与只看最近一次尝试）。
- **Cargo.toml**：`version` 由 `0.1.0` 改为 `0.1.0-pre.1`；`Cargo.lock` 里 `qtcloud-work-cli` 的版本随之同步（否则 `cargo --locked` 核不过）。

三处一致：Cargo.toml 的 `0.1.0-pre.1`、CHANGELOG 的 `## [0.1.0-pre.1]`、tag 前缀 `cli/v0.1.0-pre.1`。

改了哪些文件：

```
apps/qtcloud-work/src/cli/Cargo.toml      version = "0.1.0-pre.1"
apps/qtcloud-work/src/cli/Cargo.lock      qtcloud-work-cli = "0.1.0-pre.1"
apps/qtcloud-work/src/cli/CHANGELOG.md    ## [0.1.0-pre.1] - 2026-09-11
```

两条判据当场核过（从工作区根跑，与工作流里的命令一致）：

```text
$ cd apps/qtcloud-work/src/cli && sh scripts/validate-changelog.sh "0.1.0-pre.1"
（退出码 0，无输出）
$ cd apps/qtcloud-work/src/cli && sh scripts/validate-version.sh "cli/v0.1.0-pre.1"
0.1.0-pre.1
```
