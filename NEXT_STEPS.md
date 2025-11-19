## 后续计划：Flask + React 管理后台（基于 SQLite，同一端口）

目标：做一个自用的“数据管理 + 简单分析”后台，前端 React，后端复用现有 Flask，数据源仍是本地 SQLite (`data/xhs_rank.db`，相对路径，Win/Mac 通用)。
开发阶段采用双端口（前端 dev server + 后端 8000），上线后改为单端口（后端托管前端静态资源）。

---

### 1. 数据模型（与当前 SQLite 保持一致）

- 库文件：`data/xhs_rank.db`（自动创建）。
- 表：
  - `note_rank`：`uuid`（主键）、`title`、`nickname`、`publish_time`、`read_count`、`click_rate`、`pay_conversion_rate`、`gmv`、`fetch_date`、`created_at`（东八区时间）。
  - `account_rank`：`uuid`（主键）、`shop_name`、`fans_count`、`read_count`、`click_rate`、`pay_conversion_rate`、`gmv`、`fetch_date`、`created_at`（东八区时间）。
  - `audit_log`：`uuid`、`action`、`detail`、`created_at`（东八区时间）。
- 采集/上传：浏览器扩展已有两个独立按钮（仅保存到库、仅上传飞书），避免重复写入。

---

### 2. 后端（Flask）

- 复用现有 Flask 服务进程，新增 REST API：列表、分页、条件查询、创建、更新、删除（note_rank/account_rank）。
- 补充接口：批量导出 CSV、按日期/店铺模糊查询、审计日志查询。
- 认证：简单 Token 或 Basic（仅本机/内网用）。
- 端口：开发阶段使用 8000 提供 API；前端 dev server 走另一个端口并通过代理访问 8000。上线时可将前端打包后由 Flask/Nginx 静态托管，切回单端口（仍 8000）。

---

### 3. 前端（React）

- 技术栈建议：React + Vite + TypeScript + Ant Design（快速出好看的表格/表单/筛选）。
- 代码位置：`Data Management System/`（相对仓库根目录），前端相关代码与管理系统文件统一放在此目录。
- 端口模式：开发阶段前端 dev server（默认 5173/3000 等）+ 后端 8000 双端口联调；上线后打包静态资源交由后端/Nginx 提供，转为单端口。
- 页面/功能：
  - 列表页：note_rank/account_rank 两个 Tab，表格支持列筛选、排序、分页、模糊搜索。
    - 默认排序：`fetch_date` 从新到旧，其次按排名（0-9）升序。
  - 详情/编辑：抽屉或 Modal 内联编辑，支持新增/删除。
  - 导出：当前筛选结果导出 CSV。
  - 审计：简单日志列表，查看最近写入动作。
- 主题：使用 Ant Design 默认主题或自定义主色，保证桌面端体验。

#### 3.1 页面设计细节
- **全局布局**：顶部导航（Logo/标题 + Token 设置入口），左侧可选切换（笔记榜 / 账号榜 / 审计）。
- **笔记榜列表（Tab 1）**：
  - 查询条件（表格上方表单）：`关键词`（标题/昵称模糊）、`获取日期`（单选或范围）、`发布日期`（范围，文本模糊即可）、`GMV区间`（文本）、`排序`（按 created_at / fetch_date / GMV）。
  - 表格列：标题、昵称、发布日期、阅读数、点击率、支付转化率、GMV、获取日期、created_at、uuid（可隐藏）。列可排序/筛选，支持列宽拖拽。
  - 操作列：编辑（Modal 抽屉表单）、删除（二次确认）。
  - 创建按钮：右上角“新增记录”，弹出表单（同编辑项），uuid 由后端生成。
  - 导出：根据当前筛选条件导出 CSV（调用后端导出接口）。
- **账号榜列表（Tab 2）**：同笔记榜，字段换成店铺名/粉丝数等，筛选新增 `粉丝数` 文本匹配。
- **审计日志（Tab 3）**：表格字段 uuid、action、detail、created_at；支持时间范围筛选、关键词过滤（detail）。
- **体验优化**：表格 Loading 状态、空数据占位、错误提示（message/notification），分页记忆页码。

#### 3.2 前端状态与接口约定
- 分页：前端传 `page` / `page_size`；后端返回 `items`+`total`。
- 模糊查询字段统一用 `q`（标题/昵称/店铺名）；其他条件独立参数，如 `fetch_date`, `date_from`, `date_to`。
- 日期格式：`YYYY-MM-DD` 文本，created_at 由后端 ISO（含 +08:00）。
- 数值/区间：阅读数、点击率、GMV 等仍用文本存储，搜索时用包含匹配。
- 错误处理：后端统一返回 `{ok: false, error: "message"}`，前端用 message.error。

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
