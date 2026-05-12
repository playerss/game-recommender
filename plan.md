# 开发计划

## 任务列表

| ID | 任务描述 | 优先级 | 状态 | 开发者 | 测试者 |
|----|----------|--------|------|--------|--------|
| T001 | 项目初始化：创建目录结构和基础文件 | P0 | pending | dev-1 | - |
| T002 | 爬虫开发：3DM 数据抓取模块 | P0 | pending | dev-2 | - |
| T003 | 爬虫开发：游侠网数据抓取模块 | P0 | pending | dev-3 | - |
| T004 | 数据聚合：统一数据格式和去重 | P1 | pending | dev-2 | - |
| T005 | 前端开发：游戏列表页面 | P1 | pending | dev-4 | - |
| T006 | 前端开发：响应式设计和样式美化 | P1 | pending | dev-4 | - |
| T007 | 集成测试：完整流程测试 | P2 | pending | - | tester-1 |
| T008 | 文档和部署说明 | P2 | pending | dev-1 | - |

## 任务详情

### T001: 项目初始化
- 创建目录结构: `scraper/`, `data/`, `templates/`, `static/`
- 创建 `requirements.txt` (依赖列表)
- 创建 `README.md`
- 创建 `run_scraper.py` (主入口)

### T002: 3DM 爬虫
- 分析 3DM 新游中心页面结构
- 使用 requests + BeautifulSoup 抓取
- 处理反爬 (User-Agent, 延迟)
- 输出到 `data/3dm_games.json`

### T003: 游侠网爬虫
- 分析游侠网排行榜页面结构
- 使用 requests + BeautifulSoup 抓取
- 处理反爬
- 输出到 `data/ali213_games.json`

### T004: 数据聚合
- 读取两个网站的 JSON 文件
- 统一字段: title, url, image, description, source, date
- 去重 (基于 URL 或标题相似度)
- 合并输出到 `data/games.json`

### T005: 前端页面
- 创建 `index.html`
- 使用 JavaScript 读取 `data/games.json`
- 渲染游戏卡片列表

### T006: 样式美化
- 创建 `static/style.css`
- 响应式设计 (移动端适配)
- 卡片悬停效果
