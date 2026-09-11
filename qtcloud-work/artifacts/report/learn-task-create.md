# 报告：task-sheet-to-site

## 档案

任务书原样进了学习管理档案的 `tasks/`。

- 来源：`data/profile/iGuo/artifacts/learn-profile/try-qtcloud-work-cli.md`（知识工作个人档案，本仓 `data/profile` 子模块）
- 落点：`../quanttide-learn/data/profile/tasks/try-qtcloud-work-cli.md`（仓 `quanttide-profile-of-learning-management`）
- 复制：`cp -p`，文件名与源文件一致，内容逐字未改——`diff` 无输出，两处 sha256 同为 `b0e3872e04a1025f808d2e457f1ba8ecbfdba398f17fd62f82d6c0446ef5880b`
- 分类：学习任务书归 `tasks/`，未落到 `learners/` 或 `schedules/`
- 提交：`3b7790e3eee4ae6133658eafee193bebdd89d465`，信息 `docs: 学习档案新增任务书 try-qtcloud-work-cli（试用量潮工作云命令行工具）`
- 推送：`7e64324..3b7790e  main -> main`，推后 `git status --porcelain` 为空

## 站点

站点的学习内容是学习档案对应目录的逐字副本（`learning.ts` 里写明的同步方式），落点 `src/site/src/data/learning/tasks/try-qtcloud-work-cli.md`，学习页用 `import.meta.glob` 自动收进去，不用改代码。

- 复制：`cp -p`，与档案那份 `diff` 无输出
- 提交：`50abf88`，信息 `feat(site): 学习任务书上线——试用量潮工作云命令行工具`
- 推送：`main -> main`，推后工作区干净

## 预检

- 版本：`src/site/package.json` 与 `src/site/CHANGELOG.md` 都到 `0.1.2-beta.7`，两处对齐（`0.1.2-beta.7` 条目录入「学习任务书新增《试用量潮工作云命令行工具》」）
- 提交：`1280ead`
- 预检：`qtcloud-devops release audit -v site/v0.1.2-beta.7` —— 首轮 6/7（卡在「工作区有未提交变更」，版本与变更记录还没提交），提交后 **7/7 全过**

## 发布

- 放行：创始人放行（授权范围含本次 site 发布）
- 发布：`qtcloud-devops release publish -y -v site/v0.1.2-beta.7`，tag `site/v0.1.2-beta.7` → `1280ead` 已推远端
- 部署：`Deploy Site` run [34599410303](https://github.com/quanttide/qtclass/actions/runs/34599410303) **success**（构建静态站 → 上传 OSS 桶 `qtclass-site` → 刷 CDN）
- 站点核对：`https://class.quanttide.com/` 首页 200；线上产物 `assets/index-D3o8ueN9.js` 里查得到任务书标题与摘要
- 一处遗留：详情页直开 `https://class.quanttide.com/learn/tasks/try-qtcloud-work-cli` 返回 404——单页应用没有回退配置，从学习页点进去正常。既有的任务书也一样，不是这次引入的

## 结论

任务书已送出：学习档案（`tasks/try-qtcloud-work-cli.md`）、课堂站点（`src/site/src/data/learning/tasks/`）两处都有，站点新版本 `0.1.2-beta.7` 已上线。
现在停在：站点正在对外提供，学生到 `https://class.quanttide.com/learn` 就能看到这份任务书。
交付方式任务书里已经写死：学生自己在 `quanttide/qtcloud-work` 仓按四款模板开 issue（一条发现一条 issue），能被复现就算通过；开 issue 的入口也从模板配置指到学习页上这份任务书。

仍拿不准的：

- 学习页这份任务书要不要在站内给个直达链（详情页直开现在 404）
- 详情页直开 404 要不要治（单页应用加回退配置，属 qtclass 自己的事）
- site 里的任务书要不要按学期或训练营归档（现在平铺在 `tasks/` 下）
- 这条工作流的定义暂放在知识工作个人档案里，但它跨学习域与课堂站，要不要搬到 learn 域
