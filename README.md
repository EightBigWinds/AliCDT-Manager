# 🛡️ AliCDT-Manager

**阿里云 CDT 流量监控与自动化管理控制台**

[![Docker](https://img.shields.io/badge/Docker-ghcr.io-blue?logo=docker)](https://github.com/lillinlin/AliCDT-Manager/pkgs/container/alicdt-manager)
[![GitHub](https://img.shields.io/badge/GitHub-lillinlin-black?logo=github)](https://github.com/lillinlin/AliCDT-Manager)

</div>

---

## ✨ 功能
- 支持 AMD64 / ARM64 架构
- 多账户聚合监控，CDT 流量实时展示
- 抢占式实例保活：被回收自动拉起
- 流量熔断：超阈值自动停机
- 余额待还熔断：到设置的待还余额值自动停机
- 抢占型实例类型库存不足提醒
- 停机模式有节省停机和普通停机，默认节省，可自选择
- 定时开关机计划
- Telegram 告警通知
- 账单统计（待还款金额，国际站准确）
- 每1~2分钟自动同步数据，有特殊情况可点击立即同步

## 🔑 所需 RAM 权限
https://ram.console.alibabacloud.com/users
```bash
AliyunECSFullAccess
```
```bash
AliyunCDTFullAccess
```
```bash
AliyunBSSFullAccess
```

## 🚀 一键安装

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/lillinlin/AliCDT-Manager/main/install.sh)
```

- docker-compose.yml 默认端口为 ports: "127.0.0.1:8000:8000" 在安装完成需要配置 Nginx 反代通过域名访问
- 如需 IP:端口 测试，可采用手动部署后手动编辑 docker-compose.yml


## 🛠 手动部署

```bash
mkdir -p /app/alicdt-manager/data && cd /app/alicdt-manager
```
```bash
echo "SECRET_KEY=$(cat /dev/urandom | tr -dc 'a-zA-Z0-9' | head -c 48)" > .env
```
```bash
curl -fsSL https://raw.githubusercontent.com/lillinlin/AliCDT-Manager/main/docker-compose.yml -o docker-compose.yml
```
```bash
docker compose up -d
```

## ✨ 界面截图
![1](READMEimages/1.png)  
![2](READMEimages/2.png)  
![3](READMEimages/3.png)  
![5](READMEimages/5.png)  

## Nginx 配置示例

请手动填写 
- 端口
- 域名
- Pem证书路径
- Key证书路径

```bash
nano /etc/nginx/sites-available/alicdt-manager.conf
```
- Nginx 配置示例
```bash
server {
    listen 端口 ssl;
    server_name 域名;

    ssl_certificate     Pem证书路径;
    ssl_certificate_key Key证书路径;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers off;

    gzip on;
    gzip_types text/css application/json application/javascript text/javascript;
    gzip_min_length 1024;
    gzip_comp_level 6;

    keepalive_timeout 64;

    proxy_http_version 1.1;
    proxy_set_header Connection "";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    proxy_connect_timeout 10s;
    proxy_send_timeout 60s;
    proxy_read_timeout 120s;

    location ^~ /data/ {
        return 404;
    }

    location ~ (^|/)\. {
        return 404;
    }

    location / {
        proxy_pass http://alicdt_backend;
    }
}

upstream alicdt_backend {
    server 127.0.0.1:8000;
    keepalive 64;
}
```
- 填好上方配置后
```bash
ln -s /etc/nginx/sites-available/alicdt-manager.conf /etc/nginx/sites-enabled/
```
```bash
nginx -t && systemctl reload nginx
```
---

## 📋 常用命令

```bash
# 重启服务
cd /app/alicdt-manager && docker compose restart

# 更新到最新镜像
cd /app/alicdt-manager && docker compose pull && docker compose up -d

# 停止服务/卸载（保留数据）
cd /app/alicdt-manager && docker compose down

# 彻底卸载
cd /app/alicdt-manager && docker compose down && rm -rf /app/alicdt-manager
```


## Tech Stack

- Backend: Python + FastAPI + APScheduler + SQLite
- Frontend: Vue + TailwindCSS


## Nodeseek
https://www.nodeseek.com/post-737919-1
