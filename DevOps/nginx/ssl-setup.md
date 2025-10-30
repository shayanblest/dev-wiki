# 🔐 SSL Setup (Certbot + HTTPS) for Nginx

This guide explains how to enable free SSL (HTTPS) for your domains using **Certbot** and **Let’s Encrypt** on an Nginx server.

---

## 🧩 1. Install Certbot

```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx -y
```

---

## ⚙️ 2. Obtain SSL Certificates

Run the following command to automatically configure SSL for your domains:

```bash
sudo certbot --nginx -d example.com -d www.example.com -d example2.com -d www.example2.com
```

### What this does
- Validates domain ownership
- Automatically edits your Nginx config to enable HTTPS
- Asks if you want to redirect all HTTP traffic to HTTPS (choose **Redirect**)

---

## 🔁 3. Test Configuration

```bash
sudo nginx -t
sudo systemctl reload nginx
```

Now visit your domain in a browser:
```
https://example.com
https://example2.com
```
You should see a secure lock icon 🔒.

---

## ♻️ 4. Automatic Renewal

Certbot renews SSL certificates automatically every 60 days.  
To test renewal manually, run:

```bash
sudo certbot renew --dry-run
```

To view existing certificates:

```bash
sudo certbot certificates
```

---

## 🧱 5. Managing Multiple Domains

If you later add another domain, simply rerun certbot with the full list of domains, e.g.:

```bash
sudo certbot --nginx -d example.com -d www.example.com -d example3.com -d www.example3.com
```

This updates your existing certificate to include the new domain.

---

## 🔒 6. Enforce HTTPS (Manual Option)

If you prefer to edit manually, you can force HTTPS in Nginx like this:

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name example.com www.example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:4200;
    }
}
```

---

## 🧰 7. Useful Commands

| Purpose | Command |
|----------|----------|
| Test Nginx configuration | `sudo nginx -t` |
| Reload Nginx | `sudo systemctl reload nginx` |
| View certificates | `sudo certbot certificates` |
| Force renew all | `sudo certbot renew --force-renewal` |
| Check logs | `sudo tail -f /var/log/letsencrypt/letsencrypt.log` |

---

**Author:** Shayan  
**Last Updated:** 2025-10-30
