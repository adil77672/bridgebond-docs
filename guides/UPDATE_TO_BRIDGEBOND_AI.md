# Update to bridgebond.ai Domain - Quick Guide

This guide helps you update all configurations from `bridgebond.io` to `bridgebond.ai`.

## DNS Records (Already Configured ✅)

Your DNS is already set up correctly:
- `api` → 51.20.185.71 (Production API)
- `staging-api` → 51.20.185.71 (Staging API)
- `staging-super-admin` → 51.20.185.71 (Staging Frontend)
- `super-admin` → 51.20.185.71 (Production Frontend)

## Quick Setup on Server

### Option 1: Use Automated Script (Recommended)

```bash
# Copy the script to server (from your local machine)
scp -i your-key.pem scripts/nginx/update-bridgebond-ai-config.sh ubuntu@your-server-ip:~/

# On server, run with sudo
sudo bash ~/update-bridgebond-ai-config.sh
```

### Option 2: Manual Setup

Run these commands on your server:

```bash
# 1. Backup existing configs
sudo mkdir -p /etc/nginx/backup-$(date +%Y%m%d)
sudo cp -r /etc/nginx/sites-available/* /etc/nginx/backup-$(date +%Y%m%d)/

# 2. Create staging API config
sudo nano /etc/nginx/sites-available/staging-api.bridgebond.ai
# Copy config from NGINX_BRIDGEBOND_AI_CONFIG.md

# 3. Create production API config
sudo nano /etc/nginx/sites-available/api.bridgebond.ai
# Copy config from NGINX_BRIDGEBOND_AI_CONFIG.md

# 4. Create staging frontend config
sudo nano /etc/nginx/sites-available/staging-super-admin.bridgebond.ai
# Copy config from NGINX_BRIDGEBOND_AI_CONFIG.md

# 5. Create production frontend config
sudo nano /etc/nginx/sites-available/super-admin.bridgebond.ai
# Copy config from NGINX_BRIDGEBOND_AI_CONFIG.md

# 6. Enable all sites
sudo ln -sf /etc/nginx/sites-available/staging-api.bridgebond.ai /etc/nginx/sites-enabled/
sudo ln -sf /etc/nginx/sites-available/api.bridgebond.ai /etc/nginx/sites-enabled/
sudo ln -sf /etc/nginx/sites-available/staging-super-admin.bridgebond.ai /etc/nginx/sites-enabled/
sudo ln -sf /etc/nginx/sites-available/super-admin.bridgebond.ai /etc/nginx/sites-enabled/

# 7. Remove old bridgebond.io configs
sudo rm -f /etc/nginx/sites-enabled/*bridgebond.io*

# 8. Test and reload
sudo nginx -t
sudo systemctl reload nginx
```

## Get SSL Certificates

After Nginx is configured, get SSL certificates:

```bash
# Get all certificates at once
sudo certbot --nginx -d staging-api.bridgebond.ai -d api.bridgebond.ai -d staging-super-admin.bridgebond.ai -d super-admin.bridgebond.ai

# Or individually:
sudo certbot --nginx -d staging-api.bridgebond.ai
sudo certbot --nginx -d api.bridgebond.ai
sudo certbot --nginx -d staging-super-admin.bridgebond.ai
sudo certbot --nginx -d super-admin.bridgebond.ai
```

## Update Environment Variables

### Frontend (.env files)

**Staging:** `~/bridge-bond/projects/bridgebond_super_Admin/.env.staging`
```env
VITE_API_URL=https://staging-api.bridgebond.ai/v1
```

**Production:** `~/bridge-bond/projects/bridgebond_super_Admin/.env.production`
```env
VITE_API_URL=https://api.bridgebond.ai/v1
```

### Backend (.env files)

**Staging:** `~/bridge-bond/projects/bridge-bond-api/.env`
```env
BASE_URL=https://staging-api.bridgebond.ai
```

**Production:** `~/bridge-bond/projects/bridge-bond-api/.env`
```env
BASE_URL=https://api.bridgebond.ai
```

## Verify Everything Works

```bash
# Test all domains
curl -I https://staging-api.bridgebond.ai
curl -I https://api.bridgebond.ai
curl -I https://staging-super-admin.bridgebond.ai
curl -I https://super-admin.bridgebond.ai

# Check PM2 apps
pm2 list

# Check Nginx status
sudo systemctl status nginx

# View logs
sudo tail -f /var/log/nginx/staging-api-access.log
sudo tail -f /var/log/nginx/api-access.log
```

## Domain Summary

| Service | Domain | Port | PM2 App |
|---------|--------|------|---------|
| Staging API | staging-api.bridgebond.ai | 2323 | backend-staging |
| Production API | api.bridgebond.ai | 2323 | backend-prod |
| Staging Frontend | staging-super-admin.bridgebond.ai | 5178 | staging-superadmin |
| Production Frontend | super-admin.bridgebond.ai | 5173 | production-superadmin |

## Troubleshooting

### If Nginx test fails:
```bash
sudo nginx -t
sudo tail -50 /var/log/nginx/error.log
```

### If SSL fails:
```bash
# Check DNS
dig staging-api.bridgebond.ai
dig api.bridgebond.ai

# Check port 80 is open
sudo ufw status
sudo ufw allow 80/tcp
```

### If domains don't load:
```bash
# Check PM2
pm2 list

# Check ports
sudo netstat -tulpn | grep 2323
sudo netstat -tulpn | grep 5178
sudo netstat -tulpn | grep 5173
```

## Complete Configuration Files

See `docs/guides/NGINX_BRIDGEBOND_AI_CONFIG.md` for complete Nginx configurations.

