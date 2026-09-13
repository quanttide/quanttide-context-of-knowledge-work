# 报告：stale-audit

## 抽查清单

AI 写程序时会读的资源分四处：

- 规格 `docs/specification`：规范文本，人读；
- 案例 `docs/gallery`：每件产物、每条工作流一件，工作流案旁放同名可运行 `.yaml`；
- 手册 `docs/handbook`：全体产物的共性规则；
- user-guide `packages/quanttide-work-toolkit/docs/user-guide`：接入工具箱的说明。

触发本轮的是工具箱最近一轮破坏性改动——`artifact` 立为聚合（名字 + 规格，不带位置）、落点不再拼目录、占位收敛成一套、两侧同号到 `0.1.0-beta.6`。故抽查偏规格、案例与 user-guide，手册取一篇对照。

本轮核这八篇，每篇点明真值源：

| 篇 | 位置 | 真值源 | 指向 |
| :-- | :-- | :-- | :-- |
| artifact | `packages/quanttide-work-toolkit/docs/user-guide/artifact.md` | 源码 + 契约向量 | Rust `packages/rust/src/artifact/model.rs`，Dart `packages/dart/lib/src/artifact/`；`tests/contract/artifact-location.json` |
| workspace | `packages/quanttide-work-toolkit/docs/user-guide/workspace.md` | 源码 + 契约向量 | Rust `packages/rust/src/workspace/{model,place,progress,check}.rs`；`tests/contract/{expand-placeholders,check-skip,section-names}.json` |
| criterion | `packages/quanttide-work-toolkit/docs/user-guide/criterion.md` | 源码 + 契约向量 | Rust `packages/rust/src/criterion/{model,read,items}.rs`；`tests/contract/{criteria-kinds,validate-rule-no-kind}.json` |
| versioning | `packages/quanttide-work-toolkit/docs/user-guide/versioning.md` | 源码 | 清单 `packages/rust/Cargo.toml`、`packages/dart/pubspec.yaml`、`packages/{go,python,typescript}/`；`scripts/contract.sh` |
| 工作流规格 | `docs/specification/process/workflow.md` | 源码 + 契约向量 + 命令 `--help` | 源码 `packages/rust/src/workflow/read.rs`；契约向量 `tests/contract/validate-*.json`、`expand-placeholders.json`；`qtcloud-work workflow --help` |
| 任务规格 | `docs/specification/process/task.md` | 源码 + 契约向量 + 命令 `--help` | 源码 `packages/rust/src/task/model.rs`、`workspace/place.rs`；契约向量 `tests/contract/artifact-location.json`、`check-skip.json`；`qtcloud-work task --help` |
| 工具箱代码实现案例 | `docs/gallery/workflows/code-implement-toolkit.md` | 规范 + 源码 | 同名定义 `docs/gallery/workflows/code-implement-toolkit.yaml`；toolkit 清单 `packages/quanttide-work-toolkit/packages/{rust/Cargo.toml,dart/pubspec.yaml}` 与 `scripts/contract.sh` |
| 任务手册 | `docs/handbook/process/task.md` | 规范 | `docs/specification/process/task.md`，以及案例 `docs/gallery/workflows/index.md` 的登记要求 |

范围外但值得留意：命令行自己的文档 `apps/qtcloud-work/src/cli/docs/{user-guide,dev-guide,api-references}` 不在本轮四处之内（`qtcloud-work --help` 例子已指到 `docs/api-references/*.md`），留待下一轮。

## 实跑检查

三项检查都实跑，命令与结论如下。

### 一、文档示例与契约向量

```
$ sh packages/quanttide-work-toolkit/scripts/doc-tests.sh
文档示例：rust 17 条、dart 17 条、bash 1 条，共 35 条
全部示例都有测试兜住（文档与测试对得上）
```

doc-tests 会把 user-guide 里唯一的 bash 示例 `sh scripts/contract.sh` 真跑一遍，契约的输出因此夹在里面。为逐条落账，另单独跑一次：

```
$ sh packages/quanttide-work-toolkit/scripts/contract.sh
两侧版本一致：0.1.0-beta.6
向量：13 份
Rust 侧：contract ... ok（1 passed; 0 failed）
Dart 侧：All tests passed!
```

结论：35 条文档示例全部有测试兜底；13 份契约向量两侧同号同结论。两条判据都绿。

### 二、画廊各工作流 yaml

```
$ qtcloud-work workflow --list --workflows docs/gallery/workflows
$ qtcloud-work workflow <名字> --check --workflows docs/gallery/workflows --root .
```

`--list` 读出 7 条（code-implement-studio、code-implement-toolkit、code-implement、code-refactor、design-review、devops-release、learn-task-create），步骤完整——语法层随 load 一并过，没有一条卡在 schema。`--check` 逐条核「判据里的路径在不在」「description 提到的小节有没有判据覆盖」：5 条过，2 条红。

| 工作流 | 红在哪 | 原样 |
| :-- | :-- | :-- |
| code-implement | 判据路径不在 | `test·apps/qtcloud-work/src/cli/tests/usecases.rs` |
| design-review | 小节无判据覆盖 | `description` 提到「拿不准」，没有 `contains: '## 拿不准'` 的判据 |

`{{report}}` 一类带运行时占位的判据按工具箱的规矩记「未核」（○），不算红。

### 三、全仓相对链接可达性

本轮用一段临时 Python 扫描脚本实跑（未落盘进仓库）：遍历 `data/ docs/ apps/ packages/` 下全部 Markdown，跳过 `.git/node_modules/.venv/target`，去掉围栏代码与行内代码，取 `[](…)` 与引用式链接，跳过外链与纯锚点，按文件所在目录解析相对路径。

```
扫描 Markdown：232 篇；相对链接：184 条；外链跳过：10 条
不可达：4 条
```

4 条不可达：

- `apps/qtcloud-work/src/cli/STATUS.md` → `../../../../code/data/insight/code-agent/contract.md`（仓根没有 `code/`）
- `apps/qtcloud-work/src/studio/STATUS.md` → 同上
- `apps/qtcloud-work/src/studio/doc/index.md` → `../../../../packages/quanttide-work-toolkit/packages/dart`（少一层 `..`，落点是 `apps/packages`）
- `data/profile/iGuo/materials/default/index.md` → `../../roadmap/qtcloud-work/material.md`（真值在 `data/roadmap/qtcloud-work/material.md`，少两层 `..`）

### 跑不了的

无。三项都实跑，没有留空过。

一处如实说明：工作流 yaml 的 schema 校验没有独立命令——`--check` 走的 `load` 会把语法错误吞掉。本轮用「7 条是否读出完整步骤」侧证，均过；若要有独立入口，留待改进。

## 落后清单

拿抽查清单里每篇去对真值，落后七处，另有一处命名要创始人拍板。每条给「资源 → 落后点 → 真值 → 怎么改」。

### 一、workspace.md 把 `{{log}}` 算进产物落点

- **资源**：`packages/quanttide-work-toolkit/docs/user-guide/workspace.md`「落点与占位」。
- **落后点**：「`{{report}}` / `{{journal}}` / `{{log}}` 是这三样产物的落点」。
- **真值**：`{{log}}` 换的是任务文件本身 `tasks/<任务名>.yaml`，流水不是产物。源码 Rust `packages/rust/src/workspace/place.rs`（`LOG` → `tasks/{}.yaml`）、Dart `packages/dart/lib/src/workspace/place.dart`；契约向量 `tests/contract/expand-placeholders.json`（`{{log}}` → `tasks/甲.yaml`）；规范 `docs/specification/process/workflow.md`「`{{log}}` 是任务文件本身」与 `piece/artifact.md`「流水不是产物」。
- **改法**：这句改成「`{{report}}` / `{{journal}}` 是这两样产物的落点，`{{log}}` 是任务文件本身，`{{artifacts}}` 是产物目录」——与规范 workflow.md 同句。

### 二、versioning.md 说包「还没发」

- **资源**：`packages/quanttide-work-toolkit/docs/user-guide/versioning.md`「现在的版本状态」。
- **落后点**：「包**还没发**——按发布纪律等创始人放行。发完再把端侧（命令行 / 工作台）引到同一个号」。
- **真值**：两侧 `0.1.0-beta.6` 早发了，端侧也早引到同一个号。远端有 tag `rust/v0.1.0-beta.6`、`dart/v0.1.0-beta.6`；crates.io 的 `quanttide-work` 与 pub.dev 的 `quanttide_work`，newest 都是 `0.1.0-beta.6`（两处 API 实查）；`apps/qtcloud-work/src/cli/Cargo.toml` 写 `quanttide-work = "0.1.0-beta.6"`（`Cargo.lock` 来源 registry），`apps/qtcloud-work/src/studio/pubspec.yaml` 写 `quanttide_work: ^0.1.0-beta.6`（`pubspec.lock` 来源 hosted）。
- **改法**：改成「两侧 `0.1.0-beta.6` 已发（远端 tag 与 crates.io / pub.dev 均在），端侧已引同一个号」。

### 三、versioning.md 的端侧尺子条数

- **资源**：同上「尺子」。
- **落后点**：「`apps/qtcloud-work/src/studio/scripts/parity.sh`，24 条」。
- **真值**：实跑 `sh apps/qtcloud-work/src/studio/scripts/parity.sh --report`（工作目录 studio）→「结果一致 11 条，没对上 0 条」。11 条 = `workflow --list` 1 + `task --list` 1 + 8 件任务 + `help` 1；脚本默认 `WORKFLOWS_DIR=data/profile/quanttide/workflows`，该目录不存在，工作流那批一条没进来。
- **改法**：24 改成 11（或不写死条数）。另立一条：把 `parity.sh` 的默认工作流目录改到实际目录（`data/profile/iGuo/workflows`）——那是端侧脚本改动，需单列。

### 四、工具箱代码实现案例的发布纪律落后

- **资源**：`docs/gallery/workflows/code-implement-toolkit.md`（publish-rust / publish-dart 两节）与同名 `.yaml`。
- **落后点**：「toolkit 的约定是**一个语言包一条发布线**——Rust 与 Dart 各自定版本、各自发，互不牵连（标签形如 `rust/vX.Y.Z-alpha.N`）」「与 Rust 那条各发各的——Dart 的版本号跟自己的改动走，不必与 Rust 对齐」；两条版本判据 grep `-alpha.N`。
- **真值**：两侧同号 `0.1.0-beta.6`。`packages/quanttide-work-toolkit/scripts/contract.sh` 开头核「两侧清单版本一致」，错号即红；两包 CHANGELOG 的 `0.1.0-beta.6` 条目写「版本对齐：Rust 与 Dart 同号发」；清单 `packages/rust/Cargo.toml` 与 `packages/dart/pubspec.yaml` 同为 `0.1.0-beta.6`。
- **改法**：案例与 yaml 同改成「两侧同号发（同一个版本号），当前档位 beta」；两条版本判据由 grep `-alpha.N` 改成两侧同号的写法。改的是工作流定义，需创始人点头。

### 五、工具箱代码实现案例里的 Dart 模型目录不存在

- **资源**：`docs/gallery/workflows/code-implement-toolkit.yaml`（compare 步骤 description）。
- **落后点**：输入写「`apps/qtcloud-work/src/studio/lib/core/` 里的 Dart 模型」。
- **真值**：`apps/qtcloud-work/src/studio/lib/` 下只有 `app.dart` / `main.dart` 与 `repositories/`、`screens/`、`states/`、`views/` 四个目录，没有 `core/`（studio 仓 `git log` 里有过 `lib/core`，现已重排）。
- **改法**：路径改到 studio 模型实际所在，或删去具体路径、只写「studio 侧」。

### 六、代码实现案例的用例测试文件路径落后

- **资源**：`docs/gallery/workflows/code-implement.md` 与 `.yaml`（test 步骤）。
- **落后点**：测试「写在 `apps/qtcloud-work/src/cli/tests/usecases.rs`」，判据 `path: …/tests/usecases.rs`。
- **真值**：该文件已拆成多件。实现是 `apps/qtcloud-work/src/cli/tests/*.rs`（`agent_step.rs`、`contract.rs`、`criteria_matrix.rs`、`criteria.rs`、`defaults.rs`、`definition_check.rs`、`human_step.rs`、`material_intake.rs`、`run_context.rs`、`state_machine.rs`、`task_start.rs`；`common/` 算夹具）；对账脚本 `scripts/validate-usecases.sh` 已改成扫整个 `tests/` 目录（脚本头部即写「测试：`tests/*.rs`，`common/` 里的夹具不算」）。
- **改法**：两处单文件路径改成 `apps/qtcloud-work/src/cli/tests/`（判据核目录），或直接以 `sh scripts/validate-usecases.sh` 的实跑为准。

### 七、设计评审案例：description 提到「拿不准」，没有判据核到

- **资源**：`docs/gallery/workflows/design-review.md` 与 `.yaml`（review 步骤）。
- **落后点**：description 写「放进报告的『拿不准』一节」，判据里只有 `## 主诊断` / `## 证据` / `## 旁枝`，没有 `contains: '## 拿不准'`。
- **真值**：规范 `docs/specification/process/workflow.md`·定义核对要求「描述提到的小节应当有 `contains` 判据核到」；实跑 `qtcloud-work workflow design-review --check` 给红回执。
- **改法**：补一条 `file: '{{report}}'` + `contains: '## 拿不准'` 的判据，或在 description 里不提这一小节。

### 待拍板：手册的「任务」与规范不同义

- **资源**：`docs/handbook/process/task.md`（题名「任务规范」）与手册 index 的目录条。
- **落后点**：手册把「任务」定义成「产物之间的流动关系」，按「目标/步骤/验收」三段验收。
- **真值**：规范里「任务（Task）是过程的一次执行实例」（`docs/specification/process/task.md`、`process/index.md`），是运行数据；「流动的编排」在规范里叫「工作流」（`process/workflow.md`），在画廊里叫「流程案例」（`docs/gallery/workflows/index.md`）。工具箱 `quanttide_work::task` 的 `Task` 字段是 `name` / `start` / `workflow` / `log` / `gates` / `artifacts`，也是执行实例。
- **改法**：手册该节改题「流程规范」（内容大体可留），或另立一名把两义分开——命名取舍，请创始人定。

### 已核、未落后

artifact.md、criterion.md、规范 workflow.md、规范 task.md 四篇逐条对过源码与契约向量，没有落后：`Artifact::named` / `of`、四种判据字段名与 `RuleKind::as_str`、四个占位、落点默认处 `artifacts/<名字>/<任务名>.md`、`{{log}}` → `tasks/<任务名>.yaml` 都对得上。本节第六、七条与前面实跑检查的两条红（code-implement 判据路径、design-review 小节无覆盖）是同一处。
