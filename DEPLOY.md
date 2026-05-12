# 部署指南

本文档介绍单机游戏推荐站的多种部署方式。

---

## 目录

- [服务器部署步骤](#服务器部署步骤)
- [Docker 部署](#docker-部署)
- [定时任务配置](#定时任务配置)

---

## 服务器部署步骤

以下以 **Ubuntu 22.04 LTS** 为例，介绍使用 Nginx + Python HTTP Server 的部署方式。

### 1. 环境准备

```bash
# 更新系统
sudo apt update && sudo apt upgrade -y

# 安装 Python3 和 pip
sudo apt install -y python3 python3-pip python3-venv

# 安装 Nginx
sudo apt install -y nginx

# 安装 Git
sudo apt install -y git
```

### 2. 部署应用

```bash
# 克隆项目（或使用已有的代码）
cd /opt
sudo git clone <repository-url> game-recommender
sudo chown -R $USER:$USER game-recommender

# 进入项目目录
cd game-recommender

# 创建虚拟环境
python3 -m venv venv
source venv/bin/activate

# 安装依赖
pip install -r requirements.txt

# 首次抓取数据
python run_scraper.py
```

### 3. 创建 systemd 服务

```bash
sudo tee /etc/systemd/system/game-recommender.service > /dev/null << 'EOF'
[Unit]
Description=Game Recommender Scraper
After=network.target

[Service]
Type=oneshot
User=www-data
WorkingDirectory=/opt/game-recommender
ExecStart=/opt/game-recommender/venv/bin/python run_scraper.py
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
EOF
```

启用并启动服务：

```bash
sudo systemctl daemon-reload
sudo systemctl enable game-recommender
sudo systemctl start game-recommender
```

### 4. 配置 Nginx

```bash
sudo tee /etc/nginx/sites-available/game-recommender > /dev/null << 'EOF'
server {
    listen 80;
    server_name your-domain.com;  # 替换为实际域名

    root /opt/game-recommender;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location /data/ {
        alias /opt/game-recommender/data/;
        autoindex off;
        add_header Access-Control-Allow-Origin *;
    }

    # 静态资源缓存
    location ~* \.(css|js|png|jpg|jpeg|gif|ico)$ {
        expires 7d;
        add_header Cache-Control "public, immutable";
    }
}
EOF

# 启用站点
sudo ln -s /etc/nginx/sites-available/game-recommender /etc/nginx/sites-enabled/
sudo rm -f /etc/nginx/sites-enabled/default

# 测试配置并重启
sudo nginx -t && sudo systemctl restart nginx
```

### 5. 防火墙设置（如使用 UFW）

```bash
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

### 6. SSL 加密（可选）

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com
```

---

## Docker 部署

### Dockerfile

在项目根目录创建 `Dockerfile`：

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# 安装依赖
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 复制项目文件
COPY . .

# 创建数据目录
RUN mkdir -p data

# 默认启动：先抓取数据，再启动 HTTP 服务
CMD ["sh", "-c", "python run_scraper.py && python -m http.server 8000 --directory /app"]

EXPOSE 8000
```

### docker-compose.yml

```yaml
version: "3.8"

services:
  game-recommender:
    build: .
    ports:
      - "8000:8000"
    volumes:
      - ./data:/app/data
    restart: unless-stopped
    command: >
      sh -c "python run_scraper.py && python -m http.server 8000 --directory /app"

  # 可选：独立的定时爬虫服务
  scraper-cron:
    build: .
    volumes:
      - ./data:/app/data
    environment:
      - CRON_SCHEDULE=0 3 * * *
    command: >
      sh -c "crond && bash -c 'echo \"${CRON_SCHEDULE:-0 3 * * *} python /app/run_scraper.py --output /app/data/games.json\" >> /etc/crontab && tail -f /dev/null'"
    restart: unless-stopped
```

### 启动

```bash
# 构建并启动
docker-compose up -d --build

# 查看日志
docker-compose logs -f

# 访问
# http://localhost:8000
```

---

## 定时任务配置

### 方式一：crontab 定时抓取

```bash
# 编辑 crontab
crontab -e
```

添加以下行（每天凌晨 3 点自动抓取数据）：

```cron
# 单机游戏推荐站 - 每日数据更新
0 3 * * * /opt/game-recommender/venv/bin/python /opt/game-recommender/run_scraper.py --output /opt/game-recommender/data/games.json >> /var/log/game-recommender.log 2>&1
```

设置日志文件并分配权限：

```bash
sudo touch /var/log/game-recommender.log
sudo chmod 644 /var/log/game-recommender.log
```

### 方式二：systemd Timer（推荐）

创建 Timer 单元：

```bash
sudo tee /etc/systemd/system/game-recommender.timer > /dev/null << 'EOF'
[Unit]
Description=Daily game data scraping

[Timer]
OnCalendar=*-*-* 03:00:00
Persistent=true
RandomizedDelaySec=300

[Install]
WantedBy=timers.target
EOF
```

```bash
# 启用并启动 Timer
sudo systemctl daemon-reload
sudo systemctl enable game-recommender.timer
sudo systemctl start game-recommender.timer

# 查看 Timer 状态
sudo systemctl list-timers --all

# 手动触发测试
sudo systemctl start game-recommender.service
```

### 方式三：GitHub Actions 自动部署

在仓库中创建 `.github/workflows/deploy.yml`：

```yaml
name: Update Game Data

on:
  schedule:
    # 每天 UTC 19:00 = 北京时间 03:00
    - cron: '0 19 * * *'
  workflow_dispatch:  # 支持手动触发

jobs:
  scrape:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run scraper
        run: python run_scraper.py

      - name: Commit and push if changed
        run: |
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git add data/games.json
          git diff --staged --quiet || git commit -m "chore: update game data"
          git push
```

---

## 常见问题

### 问题 1：抓取失败 / 返回空数据

- 检查目标网站是否修改了页面结构
- 调整 `--delay` 参数增大请求间隔
- 查看 `data/3dm_games.json` 和 `data/ali213_games.json` 原始数据

### 问题 2：前端页面 JSON 加载失败 (CORS)

- 直接打开 HTML 文件不会工作，必须通过 HTTP 服务器访问
- 使用 `python -m http.server 8080` 或 Nginx 部署

### 问题 3：定时任务不执行

- 检查日志：`journalctl -u game-recommender.service`
- 检查权限：确保脚本可执行，工作目录存在
- 检查 cron 日志：`grep CRON /var/log/syslog`

### 问题 4：GitHub Pages 无法读取 JSON

- 使用上述 GitHub Actions 方案，让 Actions 每次 push 后更新 JSON 文件
- 或将 JSON 内嵌到 HTML 中（适合数据量不大的场景）

---

## 监控与维护

### 健康检查

```bash
# 检查抓取结果是否有效
python3 -c "
import json
with open('data/games.json') as f:
    games = json.load(f)
print(f'Games count: {len(games)}')
assert len(games) > 0, 'No games found!'
print('OK')
"
```

### 日志轮转（可选）

```bash
sudo tee /etc/logrotate.d/game-recommender > /dev/null << 'EOF'
/var/log/game-recommender.log {
    daily
    rotate 7
    compress
    missingok
    notifempty
}
EOF
```
