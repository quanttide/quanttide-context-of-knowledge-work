# 报告：code-implement-toolkit

## 对照

两侧逐个名词对下来（cli 在 `apps/qtcloud-work/src/cli/src/`，studio 在 `apps/qtcloud-work/src/studio/lib/core/`）：

| 领域模型 | cli（Rust） | studio（Dart） | 判定 |
|---|---|---|---|
| 信封 | `outcome::Result`（ok / lines / columns / rows / payload）+ `short()` | `Outcome` + `encodeOutcome()` + `short()` | 完全相同 |
| 执行者 | `AGENT` / `HUMAN` / `EXECUTORS` | `agent` / `human` / `executors` | 只差写法（取值同） |
| 判据种类 | `RULE` / `AGENT` / `HUMAN` / `TYPES` | `rule` / `agent` / `human` / `criterionTypes` | 只差写法（取值同） |
| 定义的字段表 | `TOP_FIELDS` / `STEP_FIELDS` / `CRITERION_FIELDS` | `topFields` / `stepFields` / `criterionFields` | 完全相同 |
| 定义校验与报错 | `load()`（逐条报错文字） | `loadDefinition()` | 完全相同（报错文字逐字同） |
| 步骤与判据的视图 | `Step`（name / description / executor / criteria / rules / agents / gates） | `Step` | 完全相同 |
| 定义核对 | `check()` / `Finding` / `describe()` / `all_ok()` / `looks_like_section()` / `expand_placeholders()` | `checkWorkflow()` / `Finding` / `describeFindings()` / `allOk()` / `looksLikeSection()` / `expandPlaceholders()` | 完全相同 |
| 机械判据 | `audit::Kind` / `Item` / `items_of()` / `description_of()` | `RuleKind` / `RuleItem` / `itemsOf()` / `descriptionOf()` | 完全相同（四种判法与说明文字同） |
| 「走过」的算法 | `Task::done()`（附加判定投票、重新执行从头算） | `Task.done()` | 完全相同 |
| 流水的结构 | `log` 里 `at` / `step` / `detail` / `ok`，闸门 `gates`，产物 `products` | 同 | 完全相同 |
| 给 AI 的两段话 | `prompt_for()` / `judge_prompt()` / `criteria_text()` | `promptFor()` / `judgePrompt()` / `criteriaText()` | 完全相同（逐字同） |
| 文件与目录的读写、YAML 解析与序列化 | `std::fs` + `serde_yaml` | `dart:io` + `package:yaml` | 只是同名不同义：各自的语言库，留各自包 |
| 起进程（`pi` / `sh`） | `std::process::Command` | `lib/core/host/` | 只是同名不同义：各自的平台边界，留各自包 |

抽取范围（这一步的闸门要创始人拍）：上表「完全相同 / 只差写法」那十行进 toolkit——信封、执行者与判据种类、定义的字段表与校验（含报错文字）、步骤与判据视图、定义核对、机械判据的四种判法、「走过」的算法、流水结构、给 AI 的两段话。最后两行（各自的文件系统与起进程）留各自包：那是语言库与平台的事，抽出去只会多一层壳。

尺子此刻全绿：两侧现在说的是同一件事——`sh apps/qtcloud-work/src/studio/scripts/parity.sh` 一致 21 条、没对上 0 条（六条工作流的 list/展示/核对 + 每件任务的状态 + help）。

## 抽取

抽进 `packages/quanttide-work-toolkit`，两侧各一份（抽的是纯逻辑：从已解析的 Map / Value 进、从值出；文件读写、YAML 解析、起进程留各自包）：

| 模块 | Rust（`packages/rust/src/`） | Dart（`packages/dart/lib/src/`） |
|---|---|---|
| 信封 | `envelope.rs`（`Outcome` + `to_json` + `short`） | `envelope.dart` |
| 定义 | `definition.rs`（字段表、`validate`、`Step`、`Workflow`、`check`、`describe`、`all_ok`、`looks_like_section`、`expand_placeholders`） | `definition.dart` |
| 判据 | `criteria.rs`（`RuleKind`、`RuleItem`、`items_of`、`description_of`） | `criteria.dart` |
| 流水 | `tasklog.rs`（`done`、`next_step`、`state_line`） | `tasklog.dart` |
| 两段话 | `prompts.rs`（`criteria_text`、`prompt_for`、`judge_prompt`、`Facts`） | `prompts.dart` |

两处与原来不同的地方，都是「抽出去必须变的」：

- 校验从「读文件」改成「吃已经解析好的值」，文件名由调用方递进来（只用来在报错里说话）——工具箱不碰文件系统；
- 定义核对里的「路径在不在」也改成由调用方传一个 `exists` / `bool Function(String)` 进来——同上。

报错文字、字段名、取值、算法逐字照抄：`cargo clippy --all-targets -- -D warnings`、`cargo test --locked` 全过（Rust 侧），`dart analyze lib/ test/`、`dart test` 全过（Dart 侧）。骨架原来那两个常量（`DOMAIN` / `VERSION`）也留着。

## 契约

契约写在**两侧共用的向量**里（按仓库惯例放在 ）：`packages/quanttide-work-toolkit/tests/contract/*.json` 十份，一对输入与期望输出；Rust 的 `packages/rust/tests/contract.rs` 与 Dart 的 `packages/dart/test/contract_test.dart` 读的是同一批文件，跑 `sh packages/quanttide-work-toolkit/scripts/contract.sh` 两边各跑一遍。

| 向量 | 管什么 |
|---|---|
| `validate-ok` / `validate-no-name` / `validate-unknown-top` / `validate-rule-no-kind` / `validate-human-needs-description` | 定义校验：过的过，报错的话逐字对（含「只认 name、description、steps」这类清单） |
| `criteria-kinds` | 四种判法翻出来的说明、判法种类与参数 |
| `done-voting` | 走过的算法：审查投反对票、重走翻盘、工作流上没有的步骤不算数 |
| `section-names` | 哪些算小节名（「收尾」算，版本号写法、占位、路径不算） |
| `expand-placeholders` | 四个占位展开成数据仓下的路径 |
| `envelope-json` | 信封的 JSON 形状（键序、字段名） |

一次**故意写坏**的记录，证向量真会红：把 `done-voting` 的期望从 `["乙"]` 改成 `["甲","乙"]`（`丙` 被 `·判` 投了反对票，本不该算走过），跑契约立刻红——`assertion left == right failed: done-voting.json：走过哪几步不一样`；复原后再跑，两侧都过：

```
向量：10 份
— Rust 侧：契约：10 份向量，两侧一致
— Dart 侧：All tests passed!
```

边界是齐的：缺字段（少了 name）、多字段（顶层多个 version）、取值不对（rule 没写判法、human 带了 rule 的字段）各给了报错文字。

## 发布

**两条线各发各的**（toolkit 的约定：一个语言包一条发布线，标签 `{语言}/vX.Y.Z`，版本号各自定、互不牵连）：

| 线 | 版本 | tag | 工作流 | 上到哪 |
|---|---|---|---|---|
| Rust | `0.1.0-alpha.5` | `rust/v0.1.0-alpha.5` | `release-rust` success | crates.io：`quanttide-work`（alpha.3、alpha.4、alpha.5 在） |
| Dart | `0.1.0-alpha.10` | `dart/v0.1.0-alpha.10` | `release-dart` success | pub.dev：`quanttide_work 0.1.0-alpha.10` |

两条都在 alpha 档（`X.Y.Z-alpha.N`），按档位递增；清单版本与各自 CHANGELOG 对齐，由工作流的校验步卡着。

**这一节花的时间比抽代码还长**，记下根子：

- 第一版把两个包合在一步、一个版本号发——与 toolkit 自己的约定冲突，改成两条线；
- Dart 那条连着挂了五次（alpha.5 到 alpha.9），都卡在认证上：我手写凭证落点，把整份 `credentials.json` 写进 `~/.pub-cache`，而新版 pub 读的是 pub 自己的 token 仓（`dart pub token add` 写的那个）——**pub 找不到凭证不报错，而是打印授权链接、轮询等人点**，所以看起来是「慢」，其实是在等。
- 最后是照着能用的那份抄：`quanttide-data-toolkit` 与 `quanttide-execute-toolkit` 用的是同一个 action（`k-paxian/dart-package-publisher`，`with: credentialJson:`），凭证落点交给它，一次就过。

**结论（这一节）**：手写凭证落点这件事，别再手写——组织里已经有能用的写法，抄它。
