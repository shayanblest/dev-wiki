# راهنمای نهایی راه‌اندازی Reverse Proxy زرین‌پال با Docker (Host Network)

این مستند شامل مراحل راه‌اندازی Nginx بر روی کانتینر داکر به منظور پروکسی کردن درخواست‌های زرین‌پال در سرور ایران است.

---

## ۱. ساختار پوشه‌بندی
ابتدا پوشه‌های لازم را ایجاد کنید:
```bash
mkdir -p zarinpal-proxy/nginx/conf.d
cd zarinpal-proxy
```

---

## ۲. فایل `docker-compose.yml`
فایل را در ریشه `zarinpal-proxy` بسازید:
```yaml
services:
  reverse-proxy:
    image: nginx:1.27-alpine
    container_name: reverse-proxy
    restart: unless-stopped
    network_mode: host
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d:ro
      - /etc/letsencrypt:/etc/letsencrypt:ro
      - /var/www/certbot:/var/www/certbot:ro
```

---

## ۳. مرحله اول: گرفتن گواهینامه SSL (فقط HTTP)
ابتدا یک کانفیگ موقت برای تایید Let's Encrypt ایجاد کنید:
`nano nginx/conf.d/zarinpal.conf`

```nginx
server {
    listen 80;
    server_name zpapi.onesecondexchange.com zpsandbox.onesecondexchange.com;

    location ^~ /.well-known/acme-challenge/ {
        root /var/www/certbot;
        default_type "text/plain";
    }

    location / {
        return 200 'Nginx is up';
    }
}
```

پوشه‌ی چالش را بسازید و داکر را بالا بیاورید:
```bash
sudo mkdir -p /var/www/certbot
docker compose up -d
```

حالا دستور دریافت Cert را اجرا کنید:
```bash
sudo certbot certonly --webroot -w /var/www/certbot \
-d zpapi.onesecondexchange.com \
-d zpsandbox.onesecondexchange.com
```

---

## ۴. مرحله دوم: کانفیگ نهایی (HTTP & HTTPS)
بعد از دریافت موفقیت‌آمیز گواهینامه‌ها، فایل `nginx/conf.d/zarinpal.conf` را کاملاً با محتوای زیر جایگزین کنید:

```nginx
resolver 1.1.1.1 8.8.8.8 valid=300s ipv6=off;

# پروکسی به سمت سرور اصلی زرین‌پال
upstream zarinpal_prod {
    server api.zarinpal.com:443;
    keepalive 32;
}

upstream zarinpal_sandbox {
    server sandbox.zarinpal.com:443;
    keepalive 32;
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name zpapi.onesecondexchange.com zpsandbox.onesecondexchange.com;
    
    location ^~ /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}

# HTTPS - Production API
server {
    listen 443 ssl;
    http2 on;
    server_name zpapi.onesecondexchange.com;

    ssl_certificate /etc/letsencrypt/live/zpapi.onesecondexchange.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/zpapi.onesecondexchange.com/privkey.pem;

    location / {
        proxy_pass https://zarinpal_prod;
        proxy_http_version 1.1;
        proxy_ssl_server_name on;
        proxy_ssl_name api.zarinpal.com;
        proxy_set_header Host api.zarinpal.com;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Connection "";
    }
}

# HTTPS - Sandbox API
server {
    listen 443 ssl;
    http2 on;
    server_name zpsandbox.onesecondexchange.com;

    ssl_certificate /etc/letsencrypt/live/zpapi.onesecondexchange.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/zpapi.onesecondexchange.com/privkey.pem;

    location / {
        proxy_pass https://zarinpal_sandbox;
        proxy_http_version 1.1;
        proxy_ssl_server_name on;
        proxy_ssl_name sandbox.zarinpal.com;
        proxy_set_header Host sandbox.zarinpal.com;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Connection "";
    }
}
```

---

## ۵. راه‌اندازی نهایی
کانتینر را ری‌استارت کنید تا تنظیمات جدید اعمال شود:
```bash
docker compose restart reverse-proxy
```

---

## ۶. تمدید خودکار (Auto-Renewal)
برای اینکه بعد از تمدید خودکار گواهینامه، Nginx در داکر متوجه تغییرات شود، اسکریپت زیر را بسازید:
```bash
sudo mkdir -p /etc/letsencrypt/renewal-hooks/deploy
sudo nano /etc/letsencrypt/renewal-hooks/deploy/reload-nginx.sh
```
محتوای فایل:
```bash
#!/bin/bash
docker exec reverse-proxy nginx -s reload
```
دسترسی اجرا بدهید:
```bash
sudo chmod +x /etc/letsencrypt/renewal-hooks/deploy/reload-nginx.sh
```

---
**نکته:** اگر پورت ۸۰ یا ۴۴۳ توسط سرویس دیگری (مثل Apache یا Nginx قدیمی روی Host) اشغال شده باشد، کانتینر بالا نخواهد آمد. قبل از شروع، از آزاد بودن این پورت‌ها مطمئن شوید.
