# Nginx Configuration for bridgebond.ai

Complete Nginx configurations for all Bridge Bond services on `bridgebond.ai` domain.

## DNS Records (Already Configured)

Based on your DNS setup:
- `api` → 51.20.185.71 (Production API)
- `staging-api` → 51.20.185.71 (Staging API)
- `staging-super-admin` → 51.20.185.71 (Staging Frontend)
- `super-admin` → 51.20.185.71 (Production Frontend)

## 1. Backend API - Staging (staging-api.bridgebond.ai)

**File:** `/etc/nginx/sites-available/staging-api.bridgebond.ai`

```nginx
# Upstream for staging backend
upstream backend_staging {
    server 127.0.0.1:2323;
    keepalive 64;
}

# HTTP - Redirect to HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name staging-api.bridgebond.ai;

    # Allow Let's Encrypt verification
    location /.well-known/acme-challenge/ {
        root /var/www/html;
        allow all;
    }

    # Redirect to HTTPS
    location / {
        return 301 https://$server_name$request_uri;
    }
}

# HTTPS Server
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name staging-api.bridgebond.ai;

    # SSL Certificate paths (configured by Certbot)
    ssl_certificate /etc/letsencrypt/live/staging-api.bridgebond.ai/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/staging-api.bridgebond.ai/privkey.pem;

    # SSL Configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    ssl_session_tickets off;
    ssl_stapling on;
    ssl_stapling_verify on;

    # Security Headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    add_header X-Frame-Options "DENY" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;

    # Logging
    access_log /var/log/nginx/staging-api-access.log;
    error_log /var/log/nginx/staging-api-error.log warn;

    # Client body size for file uploads
    client_max_body_size 50M;
    client_body_buffer_size 128k;

    # Timeouts
    proxy_connect_timeout 600s;
    proxy_send_timeout 600s;
    proxy_read_timeout 600s;
    send_timeout 600s;

    # Proxy all requests to Node.js backend
    location / {
        proxy_pass http://backend_staging;
        proxy_http_version 1.1;
        
        # WebSocket support
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        
        # Standard proxy headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host $server_name;
        
        # Proxy settings
        proxy_cache_bypass $http_upgrade;
        proxy_redirect off;
        proxy_buffering off;
    }

    # Health check endpoint
    location /health {
        access_log off;
        proxy_pass http://backend_staging/health;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }

    # Block access to hidden files
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }
}
```

## 2. Backend API - Production (api.bridgebond.ai)

**File:** `/etc/nginx/sites-available/api.bridgebond.ai`

```nginx
# Upstream for production backend
upstream backend_production {
    server 127.0.0.1:2323;
    keepalive 64;
}

# HTTP - Redirect to HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name api.bridgebond.ai;

    # Allow Let's Encrypt verification
    location /.well-known/acme-challenge/ {
        root /var/www/html;
        allow all;
    }

    # Redirect to HTTPS
    location / {
        return 301 https://$server_name$request_uri;
    }
}

# HTTPS Server
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name api.bridgebond.ai;

    # SSL Certificate paths (configured by Certbot)
    ssl_certificate /etc/letsencrypt/live/api.bridgebond.ai/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.bridgebond.ai/privkey.pem;

    # SSL Configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    ssl_session_tickets off;
    ssl_stapling on;
    ssl_stapling_verify on;

    # Security Headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    add_header X-Frame-Options "DENY" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;

    # Logging
    access_log /var/log/nginx/api-access.log;
    error_log /var/log/nginx/api-error.log warn;

    # Client body size for file uploads
    client_max_body_size 50M;
    client_body_buffer_size 128k;

    # Timeouts
    proxy_connect_timeout 600s;
    proxy_send_timeout 600s;
    proxy_read_timeout 600s;
    send_timeout 600s;

    # Proxy all requests to Node.js backend
    location / {
        proxy_pass http://backend_production;
        proxy_http_version 1.1;
        
        # WebSocket support
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        
        # Standard proxy headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host $server_name;
        
        # Proxy settings
        proxy_cache_bypass $http_upgrade;
        proxy_redirect off;
        proxy_buffering off;
    }

    # Health check endpoint
    location /health {
        access_log off;
        proxy_pass http://backend_production/health;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }

    # Block access to hidden files
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }
}
```

## 3. Frontend Super Admin - Staging (staging-super-admin.bridgebond.ai)

**File:** `/etc/nginx/sites-available/staging-super-admin.bridgebond.ai`

```nginx
# HTTP - Redirect to HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name staging-super-admin.bridgebond.ai;

    # Allow Let's Encrypt verification
    location /.well-known/acme-challenge/ {
        root /var/www/html;
        allow all;
    }

    # Redirect to HTTPS
    location / {
        return 301 https://$server_name$request_uri;
    }
}

# HTTPS Server
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name staging-super-admin.bridgebond.ai;

    # SSL Certificate paths (configured by Certbot)
    ssl_certificate /etc/letsencrypt/live/staging-super-admin.bridgebond.ai/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/staging-super-admin.bridgebond.ai/privkey.pem;

    # SSL Configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    ssl_session_tickets off;
    ssl_stapling on;
    ssl_stapling_verify on;

    # Security Headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Logging
    access_log /var/log/nginx/staging-super-admin-access.log;
    error_log /var/log/nginx/staging-super-admin-error.log warn;

    # Proxy to frontend (PM2 serve on port 5178)
    location / {
        proxy_pass http://localhost:5178;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        
        # Timeouts
        proxy_connect_timeout 600s;
        proxy_send_timeout 600s;
        proxy_read_timeout 600s;
    }

    # Client body size for file uploads
    client_max_body_size 50M;
}
```

## 4. Frontend Super Admin - Production (super-admin.bridgebond.ai)

**File:** `/etc/nginx/sites-available/super-admin.bridgebond.ai`

```nginx
# HTTP - Redirect to HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name super-admin.bridgebond.ai;

    # Allow Let's Encrypt verification
    location /.well-known/acme-challenge/ {
        root /var/www/html;
        allow all;
    }

    # Redirect to HTTPS
    location / {
        return 301 https://$server_name$request_uri;
    }
}

# HTTPS Server
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name super-admin.bridgebond.ai;

    # SSL Certificate paths (configured by Certbot)
    ssl_certificate /etc/letsencrypt/live/super-admin.bridgebond.ai/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/super-admin.bridgebond.ai/privkey.pem;

    # SSL Configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    ssl_session_tickets off;
    ssl_stapling on;
    ssl_stapling_verify on;

    # Security Headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Logging
    access_log /var/log/nginx/super-admin-access.log;
    error_log /var/log/nginx/super-admin-error.log warn;

    # Proxy to frontend (PM2 serve on port 5173)
    location / {
        proxy_pass http://localhost:5173;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        
        # Timeouts
        proxy_connect_timeout 600s;
        proxy_send_timeout 600s;
        proxy_read_timeout 600s;
    }

    # Client body size for file uploads
    client_max_body_size 50M;
}
```

## Installation Commands

Run these commands on your server to set up all configurations:

```bash
# 1. Create all Nginx configuration files
sudo nano /etc/nginx/sites-available/staging-api.bridgebond.ai
# Paste staging API config above

sudo nano /etc/nginx/sites-available/api.bridgebond.ai
# Paste production API config above

sudo nano /etc/nginx/sites-available/staging-super-admin.bridgebond.ai
# Paste staging frontend config above

sudo nano /etc/nginx/sites-available/super-admin.bridgebond.ai
# Paste production frontend config above

# 2. Enable all sites
sudo ln -sf /etc/nginx/sites-available/staging-api.bridgebond.ai /etc/nginx/sites-enabled/
sudo ln -sf /etc/nginx/sites-available/api.bridgebond.ai /etc/nginx/sites-enabled/
sudo ln -sf /etc/nginx/sites-available/staging-super-admin.bridgebond.ai /etc/nginx/sites-enabled/
sudo ln -sf /etc/nginx/sites-available/super-admin.bridgebond.ai /etc/nginx/sites-enabled/

# 3. Remove old configurations (if any)
sudo rm -f /etc/nginx/sites-enabled/*bridgebond.io*
sudo rm -f /etc/nginx/sites-enabled/*bridgebond.io*

# 4. Test Nginx configuration
sudo nginx -t

# 5. If test passes, reload Nginx
sudo systemctl reload nginx
```

## SSL Certificate Setup

After configuring Nginx, get SSL certificates for all domains:

```bash
# Get SSL certificates for all domains
sudo certbot --nginx -d staging-api.bridgebond.ai -d api.bridgebond.ai -d staging-super-admin.bridgebond.ai -d super-admin.bridgebond.ai

# Or get them individually:
sudo certbot --nginx -d staging-api.bridgebond.ai
sudo certbot --nginx -d api.bridgebond.ai
sudo certbot --nginx -d staging-super-admin.bridgebond.ai
sudo certbot --nginx -d super-admin.bridgebond.ai

# Verify auto-renewal
sudo certbot renew --dry-run
```

## Verify Configuration

```bash
# Check Nginx status
sudo systemctl status nginx

# Test all domains
curl -I https://staging-api.bridgebond.ai
curl -I https://api.bridgebond.ai
curl -I https://staging-super-admin.bridgebond.ai
curl -I https://super-admin.bridgebond.ai

# View logs
sudo tail -f /var/log/nginx/staging-api-access.log
sudo tail -f /var/log/nginx/api-access.log
```

## Port Configuration Summary

| Service | Domain | Port | PM2 App |
|---------|--------|------|---------|
| Staging API | staging-api.bridgebond.ai | 2323 | backend-staging |
| Production API | api.bridgebond.ai | 2323 | backend-prod |
| Staging Frontend | staging-super-admin.bridgebond.ai | 5178 | staging-superadmin |
| Production Frontend | super-admin.bridgebond.ai | 5173 | production-superadmin |

## Troubleshooting

### If Nginx test fails:
```bash
# Check syntax errors
sudo nginx -t

# View error log
sudo tail -50 /var/log/nginx/error.log
```

### If SSL certificate fails:
```bash
# Make sure DNS is pointing correctly
dig staging-api.bridgebond.ai
dig api.bridgebond.ai

# Make sure port 80 is open
sudo ufw status
sudo ufw allow 80/tcp
```

### If domain doesn't load:
```bash
# Check if PM2 apps are running
pm2 list

# Check if ports are listening
sudo netstat -tulpn | grep 2323
sudo netstat -tulpn | grep 5178
sudo netstat -tulpn | grep 5173
```

