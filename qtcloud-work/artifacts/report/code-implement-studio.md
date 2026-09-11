# 报告：code-implement-studio

## 点货

命令面（`qtcloud-work help` 那一页，4 组 18 条）逐条对到 studio 侧：

| 组 | 命令 | studio 侧 | 状态 |
|---|---|---|---|
| 工作区 | `find <名字> [--show]` | — | 没搬 |
| 工作区 | `catalog` | — | 没搬 |
| 工作区 | `audit [--make]` | — | 没搬 |
| 工作区 | `material [<路径>…]` | — | 没搬 |
| 定义侧 | `workflow --list` | `lib/core/definition.dart` | 已搬，对表一致 |
| 定义侧 | `workflow <名字>` | `lib/core/definition.dart` | 已搬，对表一致 |
| 定义侧 | `workflow <名字> --check` | `lib/core/definition.dart` | 已搬，对表一致 |
| 定义侧 | `workflow --new` | `lib/core/definition.dart` | 已搬 |
| 定义侧 | `workflow <名字> --export` | `lib/core/definition.dart` | 已搬 |
| 定义侧 | `workflow --import` | `lib/core/definition.dart` | 已搬 |
| 执行侧 | `task --list` | `lib/core/task.dart` | 已搬（第二轮），对表一致 |
| 执行侧 | `task --new` | `lib/core/task.dart` | 已搬（第二轮） |
| 执行侧 | `task <名字>` | `lib/core/task.dart` | 已搬（第二轮），对表一致 |
| 执行侧 | `task <名字> --next` | — | 没搬（要先把规则引擎那块搬过来） |
| 执行侧 | `task <名字> --done` | — | 没搬（同上） |
| 执行侧 | `task <名字> --journal` | `lib/core/task.dart` | 已搬（第二轮） |
| 其他 | `health` | — | 没搬 |
| 其他 | `help` | `lib/core/help.dart` | 已搬（本轮），对表一致 |

尺子跑出来的原样（`sh apps/qtcloud-work/src/studio/scripts/parity.sh --report`；第二轮结束时，尺子把每件任务的状态也管上了）：

```
  一致　　workflow --list
  一致　　workflow code-implement-studio
  一致　　workflow code-implement-studio --check
  一致　　workflow code-implement
  一致　　workflow code-implement --check
  一致　　workflow code-refactor
  一致　　workflow code-refactor --check
  一致　　workflow design-review
  一致　　workflow design-review --check
  一致　　workflow devops-release
  一致　　workflow devops-release --check
  一致　　workflow learn-task-create
  一致　　workflow learn-task-create --check
  一致　　task --list
  一致　　task code-implement-studio
  一致　　task compare-course-profile
  一致　　task context-to-profile
  一致　　task lab-to-platform
  一致　　task learn-task-create
  一致　　task pre-release
  一致　　help

一致 21 条，没对上 0 条。
全部一致：这一批命令，studio 与命令行给的是同一个信封。
```

搬运次序（这一步的闸门要创始人拍）：先把**执行侧**整块搬完——`task` 的六个子命令，界面现在最缺的就是它；再搬**工作区**四件（`find` / `catalog` / `audit` / `material`）；`health` 放最后，它要连服务端。

## 搬运

这一轮搬的是 `help`（导览；命令行那边是 `src/help.rs`，89 行）：

- 落点 `lib/core/help.dart`：四组命令的清单与命令行那份逐字相同、话题查询、导览的三种输出（导览、话题要点、认不出的话题）
- 入口：`bin/qtcloud.dart` 的 `help [<话题>]` 接上
- 测试 `test/core/help_test.dart` 五条：四组都在、命令列宽 42 且「做什么」从第 45 格起、话题要点两行、认不出的话题报「没有这条命令」、不给话题就是导览
- 边界照抄：认不出的话题给 `没有这条命令：x（\`qtcloud-work help\` 看全部）`，退出码非零

## 搬运（第二轮）

这一轮搬的是 `task`（执行实例；命令行那边是 `src/task.rs`，1060 行）里的**数据层与非 AI 动作**：

- 落点 `lib/core/task.dart`：任务文件的读写、产物落点（声明优先、没声明落草稿区）、闸门项、流水、`done()` 的算法、下一步与状态行、起任务（带运行上下文）、列表、状态、日志收叙事
- 入口 `bin/qtcloud.dart`：`task --list` / `<名字>` / `--new` / `--journal` 接上；`--next` / `--done` 如实报「还没搬」（它们要跑判据，等规则引擎那块）
- 测试 `test/core/task_test.dart` 16 条，钉住 `done()` 那张真值表：执行 ok 才算；审查 ✗ 投反对票；机器判据 ✗ 也投反对票；重走一遍都 ok 就翻过来；工作流上没有的步骤名不算数

**读实现读出一处与我原先以为的不同**：`done()` 不是「看最近一条流水」，而是「带后缀的（`·审` / `·判`）给这一步的结论投票，不带后缀的（重新执行）把结论从头算」。界面那层的 `lib/models/task.dart` 按「最近一条」算，是简化的看法；核心这层照命令行的规矩来。

## 对表

第一轮开始时 `help` 是不一致，搬完转一致；第二轮把尺子的默认清单扩大到「每件任务的状态」，再跑——**21 条全一致，0 条没对上**：

- 六条工作流的 `--list` / `<名字>` / `--check`
- `task --list` 与每件任务的 `task <名字>`
- `help`

改的一直是 studio 这一侧，尺子只加过清单，没改过判断。
