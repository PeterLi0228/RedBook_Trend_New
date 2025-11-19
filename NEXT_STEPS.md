## 后续计划：FastAPI + React 管理后台（基于 SQLite）

目标：做一个自用的“数据管理 + 简单分析”后台，前端 React，后端 FastAPI，数据源仍是本地 SQLite (`data/xhs_rank.db`，相对路径，Win/Mac 通用)。

---

### 1. 数据模型（与当前 SQLite 保持一致）

- 库文件：`data/xhs_rank.db`（自动创建）。
- 表：
  - `note_rank`：`uuid`（主键）、`title`、`nickname`、`publish_time`、`read_count`、`click_rate`、`pay_conversion_rate`、`gmv`、`fetch_date`、`created_at`（东八区时间）。
  - `account_rank`：`uuid`（主键）、`shop_name`、`fans_count`、`read_count`、`click_rate`、`pay_conversion_rate`、`gmv`、`fetch_date`、`created_at`（东八区时间）。
  - `audit_log`：`uuid`、`action`、`detail`、`created_at`。
- 采集/上传：浏览器扩展已有两个独立按钮（仅保存到库、仅上传飞书），避免重复写入。

---

### 2. 后端（FastAPI）

- 提供 REST API：列表、分页、条件查询、创建、更新、删除（note_rank/account_rank）。
- 补充接口：批量导出 CSV、按日期/店铺模糊查询、审计日志查询。
- 认证：简单 Token 或 Basic（仅本机/内网用）。
- 部署：本地运行 `uvicorn`，默认端口 8000，可通过 Nginx 反代；保持无绝对路径，使用相对 `data/xhs_rank.db`。

---

### 3. 前端（React）

- 技术栈建议：React + Vite + TypeScript + Ant Design（快速出好看的表格/表单/筛选）。
- 页面/功能：
  - 列表页：note_rank/account_rank 两个 Tab，表格支持列筛选、排序、分页、模糊搜索。
  - 详情/编辑：抽屉或 Modal 内联编辑，支持新增/删除。
  - 导出：当前筛选结果导出 CSV。
  - 审计：简单日志列表，查看最近写入动作。
- 主题：使用 Ant Design 默认主题或自定义主色，保证桌面端体验。

---

### 4. 集成与联调顺序（先规划，不写代码）

1) 定义后端 API schema（Pydantic 模型、路由）→ 与前端对齐字段命名。  
2) 前端搭 Vite 工程，接入 AntD，做基础表格 + 查询表单。  
3) 接入后端分页/筛选接口；确认日期/数值格式（文本字段保留区间字符串）。  
4) 增删改查联调；确认 UUID 主键的前端处理。  
5) 导出 CSV 功能：后端提供下载接口或前端 client-side 生成。  
6) 基础分析（可选）：增加简单图表（按日期/GMV/阅读数聚合）。  

---

### 5. 备份与运维

- 数据仍在 `data/xhs_rank.db`；换机复制整个项目目录即可。
- 备份脚本（以后再做）：定期打包 `data/xhs_rank.db` 到 `backup/`，文件名带时间戳。

---

### 6. 当前状态

- 已有 SQLite 存储、独立的“仅保存到本地库”和“仅上传飞书”按钮。  
- 这一阶段不写代码，只更新方案文档；后续按上述步骤落地 FastAPI + React 管理后台。
