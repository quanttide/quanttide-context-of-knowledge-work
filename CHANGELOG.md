# CHANGELOG

所有显著变更都将记录在此文件中。

格式基于 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.0.0/)。

版本遵循语义化版本规范：0.0.x（探索期）→ 0.x.y（验证期）→ x.y.z（正式期）

---

## [0.1.0] - 2026-09-29

### 新增

- 语境仓开张：按〈主体／域〉收当日流水（`<主体>/<域>/journal/<日期>.md`），只收原始、不加工
- 主体分层：公司（`quanttide/`）与法人主体（`quanttide-tech/`）各占一层

### 变更

- 材料归位技能迁出：改名 `context-to-profile`，移至 `quanttide-work/.agents/skills/context-to-profile/`

### 说明

- 语境是最上游：流水整理后转出档案仓（`data/profile`），转出即从语境删除，流水文件留空、目录保留
