# 单机游戏推荐站

> 自动抓取 3DM、游侠网等国内热门游戏网站的最新新游资讯，以精美卡片形式聚合展示。

## 项目简介

本项目是一个**单机游戏推荐网站**，核心功能包括：

- **数据抓取**：自动从 3DM (3dmgame.com) 和游侠网 (ali213.net) 抓取热门新游信息
- **数据聚合**：清洗、去重、统一字段格式（标题 / 链接 / 封面图 / 简介 / 来源 / 日期）
- **前端展示**：响应式暗色主题页面，游戏卡片式布局，支持移动端 / 平板 / 桌面自适应

### 技术栈

| 层级 | 技术 |
|------|------|
| 语言 | Python 3.x |
| 爬虫 | requests + BeautifulSoup4 + lxml |
| 前端 | HTML5 + CSS3 + Vanilla JS（无需框架） |
| 数据存储 | 本地 JSON 文件 |

## 安装说明

```bash
# 1. 克隆项目
git clone <repository-url>
cd game-recommender

# 2. 安装依赖
pip install -r requirements.txt

# 3. 验证安装
python run_scraper.py --help
```

**依赖列表**：

| 包 | 版本要求 | 用途 |
|----|---------|------|
| requests | >= 2.28.0 | HTTP 请求 |
| beautifulsoup4 | >= 4.11.0 | HTML 解析 |
| lxml | >= 4.9.0 | 快速 XML/HTML 解析器 |

## 使用教程

### 抓取游戏数据

```bash
# 抓取全部来源并聚合
python run_scraper.py

# 仅抓取指定来源
python run_scraper.py --source 3dm
python run_scraper.py --source ali213

# 设置请求间隔（防反爬）
python run_scraper.py --delay 2.0

# 指定输出路径
python run_scraper.py --output data/custom_games.json
```

### 查看结果

1. **数据文件**：抓取结果输出到 `data/games.json`，每条数据包含 `title`、`url`、`image`、`description`、`source`、`date`、`type` 字段
2. **前端页面**：用浏览器打开 `index.html`，即可看到游戏推荐列表（推荐使用本地 HTTP 服务访问）

### 本地 HTTP 服务

```bash
# Python 3 内置 HTTP 服务器
python -m http.server 8080
# 然后访问 http://localhost:8080

# 或在 scraper 目录中使用
cd templates
python -m http.server 8080
```

### 目录结构

```
game-recommender/
├── run_scraper.py          # 主入口脚本
├── requirements.txt        # 依赖列表
├── README.md               # 项目说明
├── DEPLOY.md               # 部署指南
├── index.html              # 前端页面（数据直读模式）
├── scraper/                # 爬虫模块
│   ├── scrape_3dm.py       # 3DM 数据抓取
│   ├── scrape_ali213.py    # 游侠网数据抓取
│   └── aggregate.py        # 数据聚合与去重
├── data/                   # 数据存储
│   ├── games.json          # 聚合后的完整数据
│   ├── 3dm_games.json      # 3DM 原始数据
│   └── ali213_games.json   # 游侠网原始数据
└── templates/              # 前端模板（含 HTTP 服务入口）
```

## 部署说明

### 方式一：GitHub Pages 部署（推荐）

1. 将项目推送到 GitHub 仓库
2. 在仓库 Settings → Pages 中，选择 main 分支 + `/root` 目录
3. **注意**：GitHub Pages 是纯静态托管，无法直接读取 `data/games.json`。有以下解决方案：
   - 方案 A：将 `data/games.json` 的内容通过 JavaScript 内嵌到 `index.html` 的 `<script>` 标签中
   - 方案 B：使用 GitHub Actions 在每次 push 时自动运行爬虫，将生成的 JSON 提交到仓库
4. 部署成功后访问 `https://<username>.github.io/<repo-name>`

### 方式二：本地 Python 服务器部署

```bash
# 先抓取最新数据
python run_scraper.py

# 启动本地 HTTP 服务
python -m http.server 8080
```

浏览器访问 `http://localhost:8080` 即可查看。适用于本地开发或内网部署。

### 方式三：生产服务器部署

详见 [DEPLOY.md](DEPLOY.md)，包含 Nginx + systemd 服务、Docker 容器化及定时任务配置。

## 贡献指南

欢迎为本项目贡献力量！

### 开发流程

1. **Fork** 本仓库
2. 创建特性分支：`git checkout -b feature/your-feature`
3. 提交更改：`git commit -m "feat: add xxx"`
4. 推送到分支：`git push origin feature/your-feature`
5. 提交 **Pull Request**

### 代码规范

- Python 代码遵循 PEP 8 规范
- 关键函数需添加中文注释和 docstring
- 爬虫模块需包含请求延迟（反爬处理）
- 前端代码保持轻量，避免引入额外依赖

### 贡献方向

- [ ] 新增更多数据源（如：机核网、游民星空等）
- [ ] 前端增加搜索 / 筛选 / 分类功能
- [ ] 数据库支持（SQLite / PostgreSQL）
- [ ] 移动端 App
- [ ] 完善测试覆盖率
- [ ] Docker 容器化部署

### 问题反馈

如发现 bug 或有功能建议，请通过 GitHub Issues 提交。

---

**数据来源**：3DM、游侠网  
**许可证**：本项目仅供学习交流使用
