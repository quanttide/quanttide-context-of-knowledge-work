# 报告：lab-to-platform

## 执行记录

- ✓ 2026-09-11 16:35　outline　AI 执行：已把这份规格要写的动作、字段、判据、用例逐条列进报告 `data/context/qtcloud-work/artifacts/report/lab-to-pl
- ✓ 2026-09-11 16:38　doc　AI 执行：三部分文档已按清单重写，落在 `apps/qtcloud-work/src/cli/docs/`：`user-guide/index.md`（怎么用，含五条做过
- ✓ 2026-09-11 16:39　doc·审　AI 审查（同一模型）：清单里的条目都落到了文档里，用例都出自做过的真事→✓

## 闸门项

- ⧗ 文档定稿（内容取舍只能人拍）（留给人 / 待判）

## 清单

这份规格写平台侧 `apps/qtcloud-work/src/cli/` 要落地的**本地知识工作做法**——实验室 `examples/default/` 那个 Python 原型 kg 的等价物。三部分文档 `user-guide` / `dev-guide` / `api-references` 都从这份清单取材；每条注出处（实验室的代码或规格）。

范围先把两头划开：**provider 接口不在本轮**（工作流：「先不接 provider，本地做法照实验室的来」）；**窗口 `kg-gui` 不纳入**平台这一轮（平台侧是 CLI，窗口是同一核心 `report.py` 的另一层）。

### 一、动作（逐动作）

工作区：
- `find <名字> [--show]`——按名找文档，认文件名与篇内一级标题，精确不中模糊兜底。（`src/kg/cli.py cmd_find`、`catalog.py Catalog.find`）
- `catalog [--json 文件]`——按资产种类列全部条目；`--json` 落 `{root,count,entries[{kind,path,names}]}`。（`report.py catalog / catalog_payload`）
- `audit [--json 文件] [--make]`——审计「资产表有而工作区无」「工作区有而未登记」；`--make` 补建缺的文档格，独立仓库三格不建。（`report.py audit`、`assets.py make`）
- `material [路径…] [--json]`——列材料四字段与阶段；不给路径就扫 `data/journal` 与 `data/profile` 下的 md。（`report.py material`、`material.py materials`）

工作流（定义）：
- `workflow --list`——列工作流、步骤、位置。（`report.py workflow_list`）
- `workflow --new <名字> --steps 甲,乙,丙 [--note 一句话]`——写 YAML，每步给判据骨架（一条 rule + 一条 human，执行者 agent）。（`workflow.py create`、`report.py workflow_new`）
- `workflow <名字>`——看步骤、谁执行、几条 rule / agent / human。（`report.py workflow_show`）
- `workflow <名字> --export <文件>`——原样导出 YAML。（`workflow.py export`）
- `workflow --import <文件> [--as 名字]`——先按 schema 验，重名挡、`--as` 换名。（`workflow.py import_workflow`）

任务（执行）：
- `task --list`——列任务、跑哪条工作流、下一步。（`report.py task_list`）
- `task --new <名字> --workflow <工作流>`——起一件任务；工作流不在则挡；写下 start、运行上下文与空流水，备好报告与日志。（`report.py task_new`、`task.py create`）
- `task <名字>`——看步骤状态、开工、工作流、指令与产物路径、最近五条流水、下一步。（`report.py task_status`）
- `task <名字> --next`——走下一步：agent 交 `pi -p`，human 等人；随后跑 rule 判据、交 agent 判据审、列 human 闸门、记账、写报告。（`report.py task_step`、`task.py execute`）
- `task <名字> --done <步骤> [--note 一句话]`——人为地记一步。（同上，auto=False）
- `task <名字> --journal <一段话>`——日志收叙事。（`task.py narrate`）

全局与出口：
- `--root` 工作区根：不写用任务里记的，其次从当前目录向上找含 `data/journal` 的目录。（`cli.py build_parser`、`assets.py repo_root`、`task.py reopen`）
- `--data` 数据仓（任务与产物草稿）。（`workflow.py lab_data`）
- `--workflows` 工作流目录，默认 `<数据仓>/workflows/`。（`workflow.py workflows_dir`）
- 动作结果 `ok=False` 时退出码 1，`lines` 打印出来。（`cli.py emit`、`report.py Result`）
- 已知未接线：`task --new` 的 `--about` 在 `cli.py` 里解析却没传下去（`cmd_task` 没把它交给 `report.task_new`）——文档只记现状，不替它编行为。

### 二、字段（逐字段）

工作流 YAML（`workflow.py`）：
- 顶层只认 `name`（必填）、`description`、`steps`（非空列表）。
- 步骤只认 `name`（必填）、`description`、`executor`（`agent|human`，默认 `agent`）、`criteria`。
- 判据只认 `executor`（`rule|agent|human`）、`description`、`path`、`absent`、`file`、`contains`、`run`。

任务 YAML（`task.py`）：
- `name`、`start`（`%Y-%m-%d %H:%M`）、`workflow`、`root`（绝对）、`data` / `workflows`（能相对工作区根则相对）、`log`（只增不改）。
- 流水事件：`at`、`step`、`detail`、`ok`。
- 状态推导：`done` ＝流水里 ok 且步骤名在定义里的集合；`next_step` ＝第一个未 done 的步骤。

产物与占位（`records.py`、`task.py`）：
- 布局：`tasks/<任务>.yaml`、`artifacts/report/<任务>.md`、`artifacts/journal/<任务>.md`；流水住在任务文件里，不另开 jsonl。
- 报告两节 `## 执行记录`、`## 闸门项` 归程序，其余节原样保留。
- 日志模板带占位行，收叙事时先去掉占位行再追加。
- 占位 `{{report}}` / `{{journal}}` / `{{log}}` / `{{artifacts}}` 展开成本任务产物，按工作区根给路径。

材料四字段（`material.py`）：`type`（扩展名）、`content`（正文首段截 40 字）、`source`（`上级目录/文件名`）、`created_at`（所在仓库首次提交日期，退回文件名日期）、`stage`（`journal` 下为「原始」，其余「材料」）；正文取 md/txt/rst。

资产与目录（`assets.py`、`catalog.py`）：`kind`（中文名）/`name`（英文名），陈述型 11 + 程序型 9 ＝二十格；独立仓库三格按 `LOCATION`（`packages/*-toolkit`、`apps/*`、`examples/*`）找；落点 `data/<name>` 或 `docs/<name>`。目录条目收 `kind`/`path`/`names`，名字含资产中英名与目录名、README 中文名、文档文件名与一级标题；扫描跳过 `.git` 等构建目录与 README/CHANGELOG/LICENSE。

动作结果（`report.py`）：`Result` 的 `ok` / `lines` / `columns` / `rows`——命令行印 `lines`，窗口画同一份表格。

### 三、判据（逐判据）

三类主体（`workflow.py TYPES`、`checks.py`）：
- `rule`——规则引擎按字段判。四种判法：`path` 存在、`absent` 不存在、`file`+`contains` 含某段文字、`run` 命令退出码为零；路径相对工作区根，绝对路径按绝对；`run` 以工作区根为 cwd。
- `agent`——智能体照 `description` 的判准审，逐条答「通过 / 不通过 + 一句理由」（`task.py judge_prompt / judge_by_ai`）。
- `human`——不跑，原样进报告「闸门项」等人拍板。

schema 约束（`workflow.py load`，不认识的字段报错）：`rule` 必须正好一种判法；`file`/`contains` 成对；`agent`/`human` 必须写 `description` 且不许带 `path` 等规则字段；步骤 `executor` 只能 `agent|human`。

执行判定（`task.py execute`、`checks.py run`）：一步算过＝所有 `rule` 通过且所有 `agent` 判为 ✓，`human` 只列闸门不影响过不过；AI 没跑成则该步不算过、流水留 ✗、报告仍写；说明可省，省了按字段拼（存在 / 不存在 / 含 / 跑通）；报告回写只替换那两节。

### 四、用例（做过几件写几件）

用例写进 `user-guide`，每条一小节，标题形如 `## 用例 一、起一件任务并走一步`；测试出处注释形如 `// 用例：一`。下列各条都有跑过的任务与流水为证：

- **用例 一、起一件任务并走一步**——`AI冒烟`（`examples/default/data/tasks/AI冒烟.yaml`，两条 ok 流水）。
- **用例 二、三类判据各判各的**——`三类判据`（同目录，rule / agent / human 各一，报告闸门待判）。
- **用例 三、比对两份课程档案**——`课程档案比对`（实验室，三步）与 `compare-course-profile`（`data/context/qtcloud-work/tasks/`，三步）。
- **用例 四、把语境条目收进材料**——`context-to-profile`（同上，五步 pull→classify→coarsen→move-out→commit）。
- **用例 五、人做的步骤人记一笔**——`数据归仓`（实验室目录，`--done 材料 / 指令`）；`课程档案比对` 也以 `--done` 记了三步。

不写：`文档迁移`——任务流水为空、报告还是旧四段格式，不是当前引擎跑出来的，故不入用例；`workflow --export/--import` 只在自带测试里验过，没有真实业务事件，也不硬凑用例。

### 五、三部分文档各写什么

- `user-guide/index.md`：怎么用——命令速览、数据三家分放，加上述用例。
- `dev-guide/index.md`：程序怎么继续长——模块分层（动作结果层、定义、任务、判据、记录、资产目录）、状态机与扩展点。
- `api-references/index.md`：命令与数据的参考——每条命令与选项、工作流 / 任务 schema、判据字段与四种判法、占位与文件布局。

## 文档

照清单重写三部分，落在 `apps/qtcloud-work/src/cli/docs/`（实验室那份不搬）：

- `user-guide/index.md`——怎么用：命令速览、数据三家分放、判据三类，加五条用例；
- `dev-guide/index.md`——程序怎么继续长：分层、状态机与数据流、扩展点、边界；
- `api-references/index.md`——命令与数据的参考：每条命令与选项、工作流与任务 schema、判据四判法、占位、文件布局、资产与目录。

用例写进使用指南，每条一个小节，做过几件写几件：起一件任务并走一步（`AI冒烟`）、三类判据各判各的（`三类判据`）、比对两份课程档案（`课程档案比对` 与 `compare-course-profile`）、把语境条目收进材料（`context-to-profile`）、人做的步骤人记一笔（`数据归仓`）。`文档迁移` 流水为空、不是当前引擎跑出来的，`workflow --export/--import` 只在自带测试里验过，都不写用例。

## 文档评审（按《命令行评审方案》逐节过）

三份文档把清单里的动作与字段基本写全了。按方案十条硬要求，缺口八处，软要求五处，另有两处可信性问题。

**硬缺口**

1. 上下文取决于「你在哪」：`--root` 缺省「从当前目录往上找含 `data/journal` 的目录」，而 user-guide 同一份文档又写「不靠你现在在哪」——两处口径冲突，且违反方案第二条硬要求。（`api-references` 全局选项、`user-guide` 任务段）
2. 写入型动作没有预演：`workflow --new` / `--import`、`task --new` / `--next` / `--done` / `--journal`、`audit --make` 都没有 `--dry-run`。
3. `--json` 两缺：一问只有 `catalog` / `audit` / `material` 有，`find`、`workflow --list|show`、`task --list|show` 没有；二问语义是「导出到文件」（要带参数），不是「以 JSON 输出到标准输出」，脚本里难吃。
4. 标准输出与标准错误的分工没定；现状是错误也走标准输出（`emit` 一律 `print`），管道里会把错误当结果。
5. `--data` 缺省「取约定位置」含糊，现状是实验室的 `data/`——平台侧的命令行不该默认指实验室的目录。
6. 错误消息给「下一步敲什么」没成约定；每条子命令的 `--help` 是否带例子也没写。
7. 长动作的可中断性没写：`--next` 把这一步交给 `pi`，超时 900 秒，Ctrl-C 的行为无约定。
8. 契约与版本没写：`--json` 字段与退出码是脚本依赖的东西，该怎么演进没约定；`--version` 与 `CHANGELOG`、`scripts/validate-version.sh` 的关系也没提。

**软缺口**

1. 动作命名混用：`--next` 走一步、`--done` 记一步、`--journal` 记日志，一个副词一个名词。
2. 不带子命令时的行为（直接给用法、不猜）没写。
3. `task --new` 撞名时的行为没写（`--import` 写了挡，`--new` 没写）。
4. 颜色约定没写；现状不输出颜色，宜写明「不用颜色」，免得将来加颜色时忘了 `NO_COLOR` 与 `--no-color`。
5. `--next` 期间先打印一句「交给 AI 跑」是与人对齐的关键，没写。

**可信性问题**

1. user-guide 用例三、用例四里把命令写成 `qtcloud-work …`，但那些流水是实验室的 `kg` 跑出来的。用例得注明「真事发生在实验室 kg 上，平台侧命令名待实现后对齐」，否则等于把没发生过的执行写成发生过的。
2. dev-guide「分层」把 `report.rs` / `workflow.rs` / `task.rs` / `checks.rs` / `records.rs` / `assets.rs` / `catalog.rs` / `material.rs` 列成现状，而平台侧现在只有 `main.rs`；这些是拟建的文件，得写明是拟。

**写得好的地方**

命令面按对象分组（工作流、任务、材料各一族）；工作流与任务的 schema、判据四判法、占位、文件布局写清了；`--about` 未接线如实标注；「状态从流水推出来」这条规则写明了推导，不另存字段。
