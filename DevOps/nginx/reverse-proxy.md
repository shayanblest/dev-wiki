# 🚀 Nginx Reverse Proxy + SSL Setup Guide

This guide explains how to set up multiple domains on a single server using **Nginx** as a reverse proxy, and enable **free SSL (HTTPS)** with **Certbot (Let’s Encrypt)**.

---

## 🧩 1. Install Nginx and Certbot

```bash
sudo apt update
sudo apt install nginx certbot python3-certbot-nginx -y
```

---

## ⚙️ 2. Create Reverse Proxy Configuration

Go to your Nginx configuration directory:

```bash
cd /etc/nginx/sites-available/
```

Create a new config file for your domain (for example: `example.com.conf`):

```bash
sudo nano example.com.conf
```

Add the following content:

```nginx
server {
    listen 80;
    server_name example.com www.example.com;

    location / {
        proxy_pass http://127.0.0.1:4200;  # your local app port
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Enable the site and reload Nginx:

```bash
sudo ln -s /etc/nginx/sites-available/example.com.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## 🌍 3. DNS Configuration

In your DNS provider panel, add the following **A records** for each domain:

```
A   example.com        <your_server_ip>
A   www.example.com    <your_server_ip>
```

If you have multiple domains (e.g. `site1.com`, `site2.com`), repeat this step for each domain.

---

## 🧱 4. Multiple Domains on One Server

### Option A – Same app for multiple domains

If all domains point to the same app (e.g. `example.com` and `example2.com` both proxy to `127.0.0.1:4200`):

Edit your config and add all domain names under `server_name`:

```nginx
server {
    listen 80;
    server_name example.com www.example.com example2.com www.example2.com;

    location / {
        proxy_pass http://127.0.0.1:4200;
        ...
    }
}
```

### Option B – Different apps per domain

Create separate config files:
- `/etc/nginx/sites-available/example.com.conf` → `proxy_pass http://127.0.0.1:4200;`
- `/etc/nginx/sites-available/example2.com.conf` → `proxy_pass http://127.0.0.1:4300;`

Then enable both with symlinks:
```bash
sudo ln -s /etc/nginx/sites-available/example2.com.conf /etc/nginx/sites-enabled/
sudo systemctl reload nginx
```

---

## 🔐 5. Enable SSL (HTTPS)

Run Certbot for all your domains:

```bash
sudo certbot --nginx -d example.com -d www.example.com -d example2.com -d www.example2.com
```

Follow the prompts:
- Enter your email
- Agree to terms
- Choose **Redirect all HTTP to HTTPS**

---

## 🧾 6. Verify and Test

Check syntax and reload Nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

Then visit:
- https://example.com  
- https://example2.com  

Both should show a secure lock 🔒

---

## ♻️ 7. Auto-Renew SSL Certificates

Certbot automatically renews certificates.  
To test renewal manually:

```bash
sudo certbot renew --dry-run
```

To view existing certificates:

```bash
sudo certbot certificates
```

---

## 🧰 8. Useful Commands

| Purpose | Command |
|----------|----------|
| Test Nginx configuration | `sudo nginx -t` |
| Reload Nginx | `sudo systemctl reload nginx` |
| Restart Nginx | `sudo systemctl restart nginx` |
| Check logs | `sudo tail -f /var/log/nginx/error.log` |
| View Certbot certificates | `sudo certbot certificates` |

---

**Author:** Shayan  
**Last Updated:** 2025-10-30
