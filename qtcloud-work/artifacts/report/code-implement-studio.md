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
| 执行侧 | `task --list` | — | 没搬（尺子上是唯一一条红） |
| 执行侧 | `task --new` | — | 没搬 |
| 执行侧 | `task <名字>` | — | 没搬 |
| 执行侧 | `task <名字> --next` | — | 没搬 |
| 执行侧 | `task <名字> --done` | — | 没搬 |
| 执行侧 | `task <名字> --journal` | — | 没搬 |
| 其他 | `health` | — | 没搬 |
| 其他 | `help` | `lib/core/help.dart` | 已搬（本轮），对表一致 |

尺子跑出来的原样（`sh apps/qtcloud-work/src/studio/scripts/parity.sh --report`）：

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
  没搬　　task --list
  一致　　help

一致 14 条，没对上 1 条。
```

搬运次序（这一步的闸门要创始人拍）：先把**执行侧**整块搬完——`task` 的六个子命令，界面现在最缺的就是它；再搬**工作区**四件（`find` / `catalog` / `audit` / `material`）；`health` 放最后，它要连服务端。

## 搬运

这一轮搬的是 `help`（导览；命令行那边是 `src/help.rs`，89 行）：

- 落点 `lib/core/help.dart`：四组命令的清单与命令行那份逐字相同、话题查询、导览的三种输出（导览、话题要点、认不出的话题）
- 入口：`bin/qtcloud.dart` 的 `help [<话题>]` 接上
- 测试 `test/core/help_test.dart` 五条：四组都在、命令列宽 42 且「做什么」从第 45 格起、话题要点两行、认不出的话题报「没有这条命令」、不给话题就是导览
- 边界照抄：认不出的话题给 `没有这条命令：x（\`qtcloud-work help\` 看全部）`，退出码非零

## 对表

这一轮开始时，尺子上 `help` 是**不一致**；搬完再跑，`help` 与 `help task` 都是**一致**。全量剩下一条红的：`task --list`（执行侧还没搬）。

改的是 studio 这一侧，尺子没动过一行。
