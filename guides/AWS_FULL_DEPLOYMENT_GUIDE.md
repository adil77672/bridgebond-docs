# Complete AWS Ubuntu Deployment Guide

**Full A-Z Setup for Backend (Node.js) & Frontend (React) with Multiple Projects**

This guide covers everything from SSH setup with multiple GitHub accounts to automated deployment scripts for staging and production environments.

## 📋 Table of Contents

1. [SSH Setup for Multiple GitHub Accounts](#1-ssh-setup-for-multiple-github-accounts)
2. [AWS EC2 Instance Setup](#2-aws-ec2-instance-setup)
3. [Initial Server Configuration](#3-initial-server-configuration)
4. [Install Required Software](#4-install-required-software)
5. [MongoDB Installation & Security](#5-mongodb-installation--security)
6. [PostgreSQL Setup (Alternative)](#6-postgresql-setup-alternative)
7. [PM2 Multi-Project Setup](#7-pm2-multi-project-setup)
8. [Nginx Configuration for Multiple Domains](#8-nginx-configuration-for-multiple-domains)
9. [SSL Certificate Setup](#9-ssl-certificate-setup)
10. [Project Structure & Cloning](#10-project-structure--cloning)
11. [Automated Deployment Scripts](#11-automated-deployment-scripts)
12. [Frontend Deployment (React)](#12-frontend-deployment-react)
13. [Edge Cases & Advanced Scenarios](#13-edge-cases--advanced-scenarios)
14. [Complete Command Reference](#14-complete-command-reference)
15. [Troubleshooting Common Issues](#15-troubleshooting-common-issues)
16. [Security Best Practices](#16-security-best-practices)
17. [Monitoring & Logging](#17-monitoring--logging)
18. [Quick Start Checklist](#18-quick-start-checklist)

---

## 1. SSH Setup for Multiple GitHub Accounts

### 1.1 Generate SSH Keys for Different Accounts

```bash
# Navigate to SSH directory
cd ~/.ssh

# Generate key for default account
ssh-keygen -t ed25519 -C "your-email@example.com" -f ~/.ssh/adil

# Generate key for second account (Adil-F)
ssh-keygen -t ed25519 -C "adil-f@example.com" -f ~/.ssh/adil-f

# Generate key for third account (Waleed)
ssh-keygen -t ed25519 -C "waleed@example.com" -f ~/.ssh/waleed

# Set correct permissions
chmod 600 ~/.ssh/adil ~/.ssh/adil-f ~/.ssh/waleed
chmod 644 ~/.ssh/adil.pub ~/.ssh/adil-f.pub ~/.ssh/waleed.pub
```

### 1.2 Create SSH Config File

```bash
# Create config file
touch ~/.ssh/config
chmod 600 ~/.ssh/config

# Edit config
nano ~/.ssh/config
```

Paste this configuration:

```ssh
# Default GitHub account
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/adil
    IdentitiesOnly yes

# Custom GitHub account (Adil-F)
Host github.com-adil-f
    HostName github.com
    User git
    IdentityFile ~/.ssh/adil-f
    IdentitiesOnly yes

# Custom GitHub account (Rana Waleed)
Host github.com-waleed
    HostName github.com
    User git
    IdentityFile ~/.ssh/waleed
    IdentitiesOnly yes
```

**Save:** `Ctrl + O`, `Enter`, `Ctrl + X`

### 1.3 Add SSH Keys to GitHub

```bash
# Display public keys
cat ~/.ssh/adil.pub
cat ~/.ssh/adil-f.pub
cat ~/.ssh/waleed.pub
```

**For each key:**
1. Copy the public key output
2. Go to GitHub → Settings → SSH and GPG keys → New SSH key
3. Paste the public key and save

### 1.4 Test SSH Connections

```bash
# Test default account
ssh -T git@github.com

# Test Adil-F account
ssh -T git@github.com-adil-f

# Test Waleed account
ssh -T git@github.com-waleed
```

You should see: `Hi [username]! You've successfully authenticated...`

---

## 2. AWS EC2 Instance Setup

### 2.1 Create EC2 Instance

1. **Login to AWS Console**
   - Go to https://console.aws.amazon.com
   - Navigate to **EC2 Dashboard**

2. **Launch Instance**
   - Click **"Launch Instance"**
   - **Name**: `bridge-bond-production`

3. **Choose AMI**
   - Select **Ubuntu Server 22.04 LTS** or **24.04 LTS**
   - Architecture: **64-bit (x86)**

4. **Instance Type**
   - **Production**: `t3.medium` (2 vCPU, 4GB RAM)
   - **Development**: `t3.small` (2 vCPU, 2GB RAM)

5. **Key Pair**
   - Create new key pair: `bridge-bond-key.pem`
   - **⚠️ SAVE THIS FILE SECURELY!**

6. **Network Settings - Security Group (CRITICAL!)**

   **Inbound Rules:**

   | Type    | Protocol | Port Range | Source    | Description        |
   |---------|----------|------------|-----------|-------------------|
   | SSH     | TCP      | 22         | My IP     | SSH access        |
   | HTTP    | TCP      | 80         | 0.0.0.0/0 | HTTP web traffic  |
   | HTTPS   | TCP      | 443        | 0.0.0.0/0 | HTTPS web traffic |
   | Custom  | TCP      | 2323       | 0.0.0.0/0 | Backend staging   |
   | Custom  | TCP      | 2324       | 0.0.0.0/0 | Backend production|

   **⚠️ IMPORTANT:** Without port 80 & 443 open, SSL certificates will FAIL!

7. **Storage**
   - Root volume: 30-50 GB (gp3 SSD)

8. **Launch Instance**
   - Wait for instance to be "Running"
   - Note the **Public IPv4 address**

---

## 3. Initial Server Configuration

### 3.1 Connect to Server

```bash
# Set permissions for key file
chmod 400 /path/to/bridge-bond-key.pem

# Connect to server (replace with YOUR IP)
ssh -i /path/to/bridge-bond-key.pem ubuntu@YOUR_SERVER_IP

# Example:
ssh -i ~/aws-keys/bridge-bond-key.pem ubuntu@51.20.185.71
```

### 3.2 Update System

```bash
# Update package list
sudo apt update

# Upgrade all packages
sudo apt upgrade -y

# Install essential tools
sudo apt install -y \
    build-essential \
    curl \
    wget \
    git \
    vim \
    htop \
    unzip \
    ufw \
    fail2ban \
    software-properties-common
```

### 3.3 Configure Timezone & Hostname

```bash
# Set timezone
sudo timedatectl set-timezone UTC

# Verify
timedatectl

# Set hostname
sudo hostnamectl set-hostname bridge-bond-api

# Add to hosts file
echo "127.0.0.1 bridge-bond-api" | sudo tee -a /etc/hosts

# Verify
hostname
```

---

## 4. Install Required Software

### 4.1 Install Node.js 20 LTS

```bash
# Add NodeSource repository
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

# Install Node.js
sudo apt install -y nodejs

# Verify installation
node --version   # Should show v20.x.x
npm --version    # Should show 10.x.x
```

### 4.2 Install PM2 Globally

```bash
# Install PM2
sudo npm install -g pm2

# Verify
pm2 --version

# Setup PM2 startup script
pm2 startup systemd

# Run the command it shows (it will look like):
# sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u ubuntu --hp /home/ubuntu

# Then save
pm2 save
```

### 4.3 Install Nginx

```bash
# Install Nginx
sudo apt install -y nginx

# Start Nginx
sudo systemctl start nginx

# Enable on boot
sudo systemctl enable nginx

# Check status
sudo systemctl status nginx
```

### 4.4 Install Certbot for SSL

```bash
# Install Certbot and Nginx plugin
sudo apt install -y certbot python3-certbot-nginx

# Verify
certbot --version
```

---

## 5. MongoDB Installation & Security

### 5.1 Install MongoDB 7.0

```bash
# Import MongoDB GPG key
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
  sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor

# Add MongoDB repository
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | \
  sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

# Update package list
sudo apt update

# Install MongoDB
sudo apt install -y mongodb-org

# Start MongoDB
sudo systemctl start mongod

# Enable on boot
sudo systemctl enable mongod

# Check status
sudo systemctl status mongod
```

### 5.2 Secure MongoDB with Authentication

```bash
# Connect to MongoDB
mongosh
```

In MongoDB shell, run:

```javascript
// Switch to admin database
use admin

// Create admin user (CHANGE PASSWORD!)
db.createUser({
  user: "admin",
  pwd: "YourStrongAdminPassword123!",
  roles: [ 
    { role: "userAdminAnyDatabase", db: "admin" }, 
    "readWriteAnyDatabase" 
  ]
})

// Create staging database and user
use bridgebond_staging
db.createUser({
  user: "bridgebond_staging",
  pwd: "YourStrongStagingPassword456!",
  roles: [ { role: "readWrite", db: "bridgebond_staging" } ]
})

// Create production database and user
use bridgebond_production
db.createUser({
  user: "bridgebond_prod",
  pwd: "YourStrongProdPassword789!",
  roles: [ { role: "readWrite", db: "bridgebond_production" } ]
})

// Exit
exit
```

### 5.3 Enable MongoDB Authentication

```bash
# Edit MongoDB config
sudo nano /etc/mongod.conf
```

Find `#security:` and change to:

```yaml
security:
  authorization: enabled
```

**Save:** `Ctrl + O`, `Enter`, `Ctrl + X`

```bash
# Restart MongoDB
sudo systemctl restart mongod

# Test connection
mongosh -u bridgebond_staging -p YourStrongStagingPassword456! --authenticationDatabase bridgebond_staging
```

Type `exit` to leave.

---

## 6. PostgreSQL Setup (Alternative)

### 6.1 Install PostgreSQL

```bash
# Install PostgreSQL
sudo apt install -y postgresql postgresql-contrib

# Start PostgreSQL
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Check status
sudo systemctl status postgresql
```

### 6.2 Create Databases and Users

```bash
# Switch to postgres user
sudo -u postgres psql
```

In PostgreSQL shell:

```sql
-- Create staging database and user
CREATE DATABASE bridgebond_staging;
CREATE USER bridgebond_staging WITH ENCRYPTED PASSWORD 'YourStrongStagingPassword456!';
GRANT ALL PRIVILEGES ON DATABASE bridgebond_staging TO bridgebond_staging;

-- Create production database and user
CREATE DATABASE bridgebond_production;
CREATE USER bridgebond_prod WITH ENCRYPTED PASSWORD 'YourStrongProdPassword789!';
GRANT ALL PRIVILEGES ON DATABASE bridgebond_production TO bridgebond_prod;

-- Exit
\q
```

### 6.3 Configure PostgreSQL for Remote Access (if needed)

```bash
# Edit PostgreSQL config
sudo nano /etc/postgresql/14/main/postgresql.conf
```

Find and change:

```conf
listen_addresses = 'localhost'  # For local only
# OR
listen_addresses = '*'          # For remote access (less secure)
```

```bash
# Edit pg_hba.conf for authentication
sudo nano /etc/postgresql/14/main/pg_hba.conf
```

Add:

```conf
# Local connections
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```

```bash
# Restart PostgreSQL
sudo systemctl restart postgresql
```

### 6.4 PostgreSQL Environment Variables

In your `.env` files:

```env
# PostgreSQL Connection
DATABASE_URL=postgresql://bridgebond_staging:YourStrongStagingPassword456!@localhost:5432/bridgebond_staging

# Or separate variables
DB_HOST=localhost
DB_PORT=5432
DB_NAME=bridgebond_staging
DB_USER=bridgebond_staging
DB_PASSWORD=YourStrongStagingPassword456!
```

---

## 7. PM2 Multi-Project Setup

### 7.1 Create Project Directory Structure

```bash
# Create directory structure
cd ~
mkdir -p bridge-bond/staging
mkdir -p bridge-bond/production
mkdir -p react-apps/staging
mkdir -p react-apps/production

# Verify structure
tree -L 2 bridge-bond react-apps
```

### 7.2 Global PM2 Ecosystem Configuration

Create a master ecosystem file for managing all projects:

```bash
# Create global ecosystem config
nano ~/ecosystem.config.js
```

Paste this configuration:

```javascript
module.exports = {
  apps: [
    // Backend Staging
    {
      name: 'backend-staging',
      script: './bridge-bond/staging/bridge-bond-api/src/index.js',
      instances: 1,
      exec_mode: 'cluster',
      autorestart: true,
      watch: false,
      max_memory_restart: '1G',
      env: {
        NODE_ENV: 'staging',
        PORT: 2323
      },
      error_file: './bridge-bond/staging/logs/err.log',
      out_file: './bridge-bond/staging/logs/out.log',
      log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
      time: true
    },
    
    // Backend Production
    {
      name: 'backend-production',
      script: './bridge-bond/production/bridge-bond-api/src/index.js',
      instances: 2,
      exec_mode: 'cluster',
      autorestart: true,
      watch: false,
      max_memory_restart: '1G',
      env: {
        NODE_ENV: 'production',
        PORT: 2324
      },
      error_file: './bridge-bond/production/logs/err.log',
      out_file: './bridge-bond/production/logs/out.log',
      log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
      time: true
    }
    
    // Add more apps as needed
  ]
};
```

**Save:** `Ctrl + O`, `Enter`, `Ctrl + X`

### 7.3 Create Log Directories

```bash
# Create log directories
mkdir -p ~/bridge-bond/staging/logs
mkdir -p ~/bridge-bond/production/logs
```

---

## 8. Nginx Configuration for Multiple Domains

### 8.1 Backend Staging Configuration

```bash
# Create Nginx config for staging API
sudo nano /etc/nginx/sites-available/api-staging.bridgebond.ai
```

Paste this:

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
    server_name api-staging.bridgebond.ai;

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
    server_name api-staging.bridgebond.ai;

    # SSL Configuration (will be auto-configured by Certbot)
    ssl_certificate /etc/letsencrypt/live/api-staging.bridgebond.ai/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api-staging.bridgebond.ai/privkey.pem;
    
    # SSL Settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256';
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    # Security Headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Logging
    access_log /var/log/nginx/api-staging-access.log;
    error_log /var/log/nginx/api-staging-error.log;

    # Client settings
    client_max_body_size 50M;

    # Proxy to Node.js backend
    location / {
        proxy_pass http://backend_staging;
        proxy_http_version 1.1;
        
        # WebSocket support
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        
        # Headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        proxy_cache_bypass $http_upgrade;
        proxy_redirect off;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # Health check
    location /health {
        access_log off;
        proxy_pass http://backend_staging/health;
    }
}
```

### 8.2 Backend Production Configuration

```bash
# Create Nginx config for production API
sudo nano /etc/nginx/sites-available/api.bridgebond.ai
```

Paste similar config but change:
- `server_name` to `api.bridgebond.ai`
- `upstream` to `backend_production` pointing to port `2324`
- SSL paths to `api.bridgebond.ai`
- Log files to `api-production-*.log`

```nginx
# Upstream for production backend
upstream backend_production {
    server 127.0.0.1:2324;
    keepalive 64;
}

# HTTP - Redirect to HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name api.bridgebond.ai;

    location /.well-known/acme-challenge/ {
        root /var/www/html;
        allow all;
    }

    location / {
        return 301 https://$server_name$request_uri;
    }
}

# HTTPS Server
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name api.bridgebond.ai;

    ssl_certificate /etc/letsencrypt/live/api.bridgebond.ai/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.bridgebond.ai/privkey.pem;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256';
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;

    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;

    access_log /var/log/nginx/api-production-access.log;
    error_log /var/log/nginx/api-production-error.log;

    client_max_body_size 50M;

    location / {
        proxy_pass http://backend_production;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### 8.3 Enable Sites

```bash
# Enable staging site
sudo ln -s /etc/nginx/sites-available/api-staging.bridgebond.ai /etc/nginx/sites-enabled/

# Enable production site
sudo ln -s /etc/nginx/sites-available/api.bridgebond.ai /etc/nginx/sites-enabled/

# Remove default site (optional)
sudo rm /etc/nginx/sites-enabled/default

# Test configuration
sudo nginx -t

# DON'T reload yet - need SSL first!
```

---

## 9. SSL Certificate Setup

### 9.1 Verify DNS Configuration

Before getting SSL, ensure your domains point to your server:

**In your DNS provider (GoDaddy, Cloudflare, etc.), add:**

```
Type: A
Name: api-staging
Value: YOUR_SERVER_IP
TTL: 3600

Type: A
Name: api
Value: YOUR_SERVER_IP
TTL: 3600
```

**Verify DNS propagation:**

```bash
# From your server
dig api-staging.bridgebond.ai
dig api.bridgebond.ai

# Should return your server IP
```

### 9.2 Verify Port 80 is Open

```bash
# Check if Nginx is listening on port 80
sudo ss -tulnp | grep :80

# Expected output:
# tcp   LISTEN  0.0.0.0:80   nginx
```

**Test from outside (from your local machine):**

```bash
curl -I http://api-staging.bridgebond.ai
curl -I http://api.bridgebond.ai
```

**If timeout → AWS Security Group is blocking port 80!**

### 9.3 Get SSL Certificates

```bash
# Stop Nginx temporarily for initial cert
sudo systemctl stop nginx

# Get certificate for staging
sudo certbot certonly --standalone -d api-staging.bridgebond.ai

# Get certificate for production
sudo certbot certonly --standalone -d api.bridgebond.ai

# Follow prompts:
# - Enter email
# - Agree to terms (Y)
# - Share email? (N is fine)
```

You should see:

```
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/api-staging.bridgebond.ai/fullchain.pem
Key is saved at: /etc/letsencrypt/live/api-staging.bridgebond.ai/privkey.pem
```

### 9.4 Start Nginx with SSL

```bash
# Test Nginx config
sudo nginx -t

# Start Nginx
sudo systemctl start nginx

# Check status
sudo systemctl status nginx
```

### 9.5 Setup Auto-Renewal

```bash
# Test renewal
sudo certbot renew --dry-run

# Check certbot timer
sudo systemctl status certbot.timer
```

---

## 10. Project Structure & Cloning

### 10.1 Clone Backend Projects

```bash
# Navigate to staging directory
cd ~/bridge-bond/staging

# Clone using SSH (Adil-F account example)
git clone git@github.com-adil-f:Adil7767/bridge-bond-api.git

# Navigate to production directory
cd ~/bridge-bond/production

# Clone production (you can use same or different repo/branch)
git clone git@github.com-adil-f:Adil7767/bridge-bond-api.git

# Or clone a specific branch
git clone -b production git@github.com-adil-f:Adil7767/bridge-bond-api.git
```

### 10.2 Setup Environment Variables

**For Staging:**

```bash
cd ~/bridge-bond/staging/bridge-bond-api
nano .env
```

Paste:

```env
# Environment
NODE_ENV=staging

# Server
PORT=2323
HOST=0.0.0.0
BASE_URL=https://api-staging.bridgebond.ai

# Database
MONGODB_URL_STAGING=mongodb://bridgebond_staging:YourStrongStagingPassword456!@localhost:27017/bridgebond_staging?authSource=bridgebond_staging
IS_LIVE=false

# JWT Configuration
JWT_SECRET=your-staging-jwt-secret-min-32-chars
JWT_ACCESS_EXPIRATION_MINUTES=2880
JWT_REFRESH_EXPIRATION_DAYS=5

# Email Configuration
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USERNAME=your-email@gmail.com
SMTP_PASSWORD=your-gmail-app-password
EMAIL_FROM=noreply-staging@bridgebond.ai

# Add other services...
```

**For Production:**

```bash
cd ~/bridge-bond/production/bridge-bond-api
nano .env
```

Paste (with production values):

```env
NODE_ENV=production
PORT=2324
HOST=0.0.0.0
BASE_URL=https://api.bridgebond.ai
MONGODB_URL_LIVE=mongodb://bridgebond_prod:YourStrongProdPassword789!@localhost:27017/bridgebond_production?authSource=bridgebond_production
IS_LIVE=true
JWT_SECRET=your-production-jwt-secret-min-32-chars-DIFFERENT-FROM-STAGING
# ... rest of production config
```

**Secure .env files:**

```bash
chmod 600 ~/bridge-bond/staging/bridge-bond-api/.env
chmod 600 ~/bridge-bond/production/bridge-bond-api/.env
```

### 10.3 Install Dependencies

```bash
# Staging
cd ~/bridge-bond/staging/bridge-bond-api
npm install

# Production
cd ~/bridge-bond/production/bridge-bond-api
npm install
```

---

## 11. Automated Deployment Scripts

### 11.1 Staging Deployment Script

```bash
# Create staging deployment script
nano ~/deploy-staging.sh
```

Paste:

```bash
#!/bin/bash
# Staging Deployment Script for Bridge Bond Backend
# This script pulls latest code, installs dependencies, and restarts the app

set -e  # Exit on any error

echo "🚀 Starting Staging Deployment..."
echo "================================"

# Configuration
PROJECT_DIR="/home/ubuntu/bridge-bond/staging/bridge-bond-api"
APP_NAME="backend-staging"
BRANCH="staging"  # or "main" depending on your setup

# Navigate to project directory
cd $PROJECT_DIR
echo "📂 Current directory: $(pwd)"

# Stash any local changes (if any)
echo "💾 Stashing local changes..."
git stash

# Fetch latest changes
echo "🔄 Fetching latest changes from GitHub..."
git fetch origin

# Checkout and pull the branch
echo "🔄 Pulling latest code from $BRANCH branch..."
git checkout $BRANCH
git pull origin $BRANCH

# Install/Update dependencies
echo "📦 Installing dependencies..."
npm install --production

# Run database migrations (if you have any)
# npm run migrate

# Build if needed (TypeScript projects)
# npm run build

# Restart PM2 process
echo "🔄 Restarting PM2 process..."
pm2 restart $APP_NAME

# Check status
echo "✅ Deployment complete! Checking status..."
pm2 status $APP_NAME

# Show recent logs
echo "📋 Recent logs:"
pm2 logs $APP_NAME --lines 20 --nostream

echo "================================"
echo "✅ Staging deployment successful!"
echo "🌐 Access at: https://api-staging.bridgebond.ai"
```

Make executable:

```bash
chmod +x ~/deploy-staging.sh
```

### 11.2 Production Deployment Script

```bash
# Create production deployment script
nano ~/deploy-production.sh
```

Paste:

```bash
#!/bin/bash
# Production Deployment Script for Bridge Bond Backend
# IMPORTANT: Always test in staging before deploying to production!

set -e  # Exit on any error

echo "🚀 Starting Production Deployment..."
echo "===================================="
echo "⚠️  WARNING: Deploying to PRODUCTION!"
echo "===================================="
read -p "Are you sure you want to continue? (yes/no): " confirm

if [ "$confirm" != "yes" ]; then
    echo "❌ Deployment cancelled."
    exit 1
fi

# Configuration
PROJECT_DIR="/home/ubuntu/bridge-bond/production/bridge-bond-api"
APP_NAME="backend-production"
BRANCH="production"  # or "main"

# Navigate to project directory
cd $PROJECT_DIR
echo "📂 Current directory: $(pwd)"

# Backup current version
echo "💾 Creating backup..."
BACKUP_DIR="/home/ubuntu/backups/production-$(date +%Y%m%d-%H%M%S)"
mkdir -p $BACKUP_DIR
cp -r $PROJECT_DIR $BACKUP_DIR
echo "✅ Backup created at: $BACKUP_DIR"

# Stash any local changes
echo "💾 Stashing local changes..."
git stash

# Fetch latest changes
echo "🔄 Fetching latest changes from GitHub..."
git fetch origin

# Checkout and pull the branch
echo "🔄 Pulling latest code from $BRANCH branch..."
git checkout $BRANCH
git pull origin $BRANCH

# Show what changed
echo "📝 Recent commits:"
git log -5 --oneline

read -p "Proceed with these changes? (yes/no): " proceed
if [ "$proceed" != "yes" ]; then
    echo "❌ Deployment cancelled."
    exit 1
fi

# Install/Update dependencies
echo "📦 Installing dependencies..."
npm install --production

# Run tests (if you have any)
# echo "🧪 Running tests..."
# npm test

# Run database migrations (if needed)
# npm run migrate:production

# Build if needed
# npm run build

# Reload PM2 process (zero-downtime)
echo "🔄 Reloading PM2 process (zero-downtime)..."
pm2 reload $APP_NAME --update-env

# Wait a moment
sleep 3

# Check status
echo "✅ Deployment complete! Checking status..."
pm2 status $APP_NAME

# Show recent logs
echo "📋 Recent logs:"
pm2 logs $APP_NAME --lines 20 --nostream

# Health check
echo "🏥 Running health check..."
sleep 2
HEALTH_CHECK=$(curl -s -o /dev/null -w "%{http_code}" https://api.bridgebond.ai/health || echo "failed")

if [ "$HEALTH_CHECK" = "200" ]; then
    echo "✅ Health check passed!"
else
    echo "⚠️  Health check failed! Status: $HEALTH_CHECK"
    echo "🔙 Consider rolling back using: pm2 restart $APP_NAME"
fi

echo "===================================="
echo "✅ Production deployment successful!"
echo "🌐 Access at: https://api.bridgebond.ai"
echo "💾 Backup available at: $BACKUP_DIR"
```

Make executable:

```bash
chmod +x ~/deploy-production.sh
```

### 11.3 Quick Deploy & Restart Script

```bash
# Create quick restart script
nano ~/restart-apps.sh
```

Paste:

```bash
#!/bin/bash
# Quick restart script for all applications

echo "🔄 Restarting all applications..."

# Restart staging
pm2 restart backend-staging
echo "✅ Staging restarted"

# Restart production
pm2 restart backend-production
echo "✅ Production restarted"

# Show status
pm2 status

# Show logs
echo "📋 Recent logs:"
pm2 logs --lines 10 --nostream
```

Make executable:

```bash
chmod +x ~/restart-apps.sh
```

### 11.4 Start All Applications

```bash
# Start all applications using PM2
cd ~
pm2 start ecosystem.config.js

# Save PM2 configuration
pm2 save

# Check status
pm2 status
```

---

## 12. Frontend Deployment (React)

### 12.1 Clone React Projects

```bash
# Navigate to react-apps directory
cd ~/react-apps/staging

# Clone frontend repo
git clone git@github.com-adil-f:YourOrg/bridge-bond-frontend.git

# For production
cd ~/react-apps/production
git clone git@github.com-adil-f:YourOrg/bridge-bond-frontend.git
```

### 12.2 Build React Apps

```bash
# Staging build
cd ~/react-apps/staging/bridge-bond-frontend
npm install
npm run build

# Production build
cd ~/react-apps/production/bridge-bond-frontend
npm install
npm run build
```

### 12.3 Nginx Configuration for React Apps

**For Staging Frontend (staging.bridgebond.ai):**

```bash
sudo nano /etc/nginx/sites-available/staging.bridgebond.ai
```

Paste:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name staging.bridgebond.ai;
    
    location /.well-known/acme-challenge/ {
        root /var/www/html;
        allow all;
    }
    
    location / {
        return 301 https://$server_name$request_uri;
    }
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name staging.bridgebond.ai;
    ssl_certificate /etc/letsencrypt/live/staging.bridgebond.ai/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/staging.bridgebond.ai/privkey.pem;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256';
    root /home/ubuntu/react-apps/staging/bridge-bond-frontend/build;
    index index.html;

    # Gzip compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # Cache static assets
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

**For Production Frontend (bridgebond.ai or www.bridgebond.ai):**

```bash
sudo nano /etc/nginx/sites-available/bridgebond.ai
```

Paste similar config with production paths.

**Enable sites:**

```bash
sudo ln -s /etc/nginx/sites-available/staging.bridgebond.ai /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/bridgebond.ai /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

**Get SSL for frontend domains:**

```bash
sudo certbot --nginx -d staging.bridgebond.ai
sudo certbot --nginx -d bridgebond.ai -d www.bridgebond.ai
```

### 12.4 Frontend Deployment Script

```bash
nano ~/deploy-super-admin-staging.sh
```

```bash
#!/bin/bash
set -e

echo "🚀 Deploying Frontend Staging..."

PROJECT_DIR="/home/ubuntu/react-apps/staging/bridge-bond-frontend"
cd $PROJECT_DIR

git stash
git pull origin staging
npm install
npm run build

echo "✅ Frontend staging deployed!"
echo "🌐 Access at: https://staging.bridgebond.ai"
```

```bash
chmod +x ~/deploy-super-admin-staging.sh
```

---

## 13. Edge Cases & Advanced Scenarios

### 13.1 Converting PuTTY Keys (.ppk) to OpenSSH (.pem)

**On Windows with PuTTYgen:**

```bash
# Using PuTTYgen command line
puttygen adgpt.ppk -O private-openssh -o adgpt.pem
```

**Or using PuTTYgen GUI:**
1. Open PuTTYgen
2. Load your .ppk file
3. Go to Conversions → Export OpenSSH key
4. Save as .pem

**Set permissions:**

```bash
chmod 600 adgpt.pem
ssh -i ~/Downloads/adgpt.pem ubuntu@23.23.170.164
```

### 13.2 Running Multiple Apps on Different Ports

**PM2 with npm start on specific ports:**

```bash
# Frontend on port 3000
pm2 start npm --name "frontend.adgpt.com" -- start -- --port 3000

# Main app on port 3001
pm2 start npm --name "app.adgpt.com" -- start -- --port 3001

# Staging on port 3002
pm2 start npm --name "app2.adgpt.com" -- start -- --port 3002
```

**Important Notes:**
- The `--` before `start` separates PM2 args from npm args
- The second `--` before `--port` separates npm script from its args
- Works with `next dev`, `vite`, or any dev server that accepts port args

### 13.3 PM2 Ecosystem for Multiple Ports

```bash
nano ~/ecosystem-multi.config.js
```

```javascript
module.exports = {
  apps: [
    {
      name: 'frontend.adgpt.com',
      cwd: '/home/ubuntu/projects/frontend',
      script: 'npm',
      args: 'start -- --port 3000',
      env: {
        NODE_ENV: 'production',
        PORT: 3000
      }
    },
    {
      name: 'app.adgpt.com',
      cwd: '/home/ubuntu/projects/main-app',
      script: 'npm',
      args: 'start -- --port 3001',
      env: {
        NODE_ENV: 'production',
        PORT: 3001
      }
    },
    {
      name: 'app2.adgpt.com',
      cwd: '/home/ubuntu/projects/staging-app',
      script: 'npm',
      args: 'start -- --port 3002',
      env: {
        NODE_ENV: 'staging',
        PORT: 3002
      }
    }
  ]
};
```

Start all:

```bash
pm2 start ecosystem-multi.config.js
pm2 save
```

### 13.4 Nginx Configuration for Multiple Domains

**Production API (api.adgpt.com):**

```bash
sudo nano /etc/nginx/sites-available/api.adgpt.com
```

```nginx
upstream backend_production {
    server 127.0.0.1:3001;
    keepalive 64;
}

server {
    listen 80;
    server_name api.adgpt.com;
    location / {
        return 301 https://$server_name$request_uri;
    }
}

server {
    listen 443 ssl http2;
    server_name api.adgpt.com;
    ssl_certificate /etc/letsencrypt/live/api.adgpt.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.adgpt.com/privkey.pem;

    location / {
        proxy_pass http://backend_production;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

**Staging API (api2.adgpt.com):**

```bash
sudo nano /etc/nginx/sites-available/api2.adgpt.com
```

```nginx
upstream backend_staging {
    server 127.0.0.1:3002;
    keepalive 64;
}

server {
    listen 80;
    server_name api2.adgpt.com;
    location / {
        return 301 https://$server_name$request_uri;
    }
}

server {
    listen 443 ssl http2;
    server_name api2.adgpt.com;
    ssl_certificate /etc/letsencrypt/live/api2.adgpt.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api2.adgpt.com/privkey.pem;

    location / {
        proxy_pass http://backend_staging;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

**Frontend (frontend.adgpt.com):**

```bash
sudo nano /etc/nginx/sites-available/frontend.adgpt.com
```

```nginx
upstream frontend_app {
    server 127.0.0.1:3000;
    keepalive 64;
}

server {
    listen 80;
    server_name frontend.adgpt.com;
    location / {
        return 301 https://$server_name$request_uri;
    }
}

server {
    listen 443 ssl http2;
    server_name frontend.adgpt.com;
    ssl_certificate /etc/letsencrypt/live/frontend.adgpt.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/frontend.adgpt.com/privkey.pem;

    location / {
        proxy_pass http://frontend_app;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

**Enable all sites:**

```bash
sudo ln -s /etc/nginx/sites-available/api.adgpt.com /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/api2.adgpt.com /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/frontend.adgpt.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

**Get SSL for all domains:**

```bash
sudo certbot --nginx -d api.adgpt.com
sudo certbot --nginx -d api2.adgpt.com
sudo certbot --nginx -d frontend.adgpt.com
```

### 13.5 Complete Deployment Script with Error Handling

```bash
nano ~/deploy-with-checks.sh
```

```bash
#!/bin/bash
# Advanced Deployment Script with Error Handling

set -e  # Exit on error

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Configuration
PROJECT_DIR="/home/ubuntu/bridge-bond/staging/bridge-bond-api"
APP_NAME="backend-staging"
BRANCH="staging"
LOG_FILE="/home/ubuntu/deployment.log"

# Functions
log() {
    echo -e "${GREEN}[$(date +'%Y-%m-%d %H:%M:%S')]${NC} $1" | tee -a $LOG_FILE
}

error() {
    echo -e "${RED}[ERROR]${NC} $1" | tee -a $LOG_FILE
    exit 1
}

warning() {
    echo -e "${YELLOW}[WARNING]${NC} $1" | tee -a $LOG_FILE
}

# Check if app is running
check_app_status() {
    if pm2 describe $APP_NAME > /dev/null 2>&1; then
        log "✅ App $APP_NAME is running"
        return 0
    else
        warning "⚠️  App $APP_NAME is not running"
        return 1
    fi
}

# Backup function
create_backup() {
    log "💾 Creating backup..."
    BACKUP_DIR="/home/ubuntu/backups/$(date +%Y%m%d-%H%M%S)"
    mkdir -p $BACKUP_DIR
    cp -r $PROJECT_DIR $BACKUP_DIR
    log "✅ Backup created at: $BACKUP_DIR"
}

# Health check
health_check() {
    log "🏥 Running health check..."
    sleep 3
    
    HEALTH_URL="http://localhost:2323/health"
    HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" $HEALTH_URL || echo "failed")
    
    if [ "$HTTP_CODE" = "200" ]; then
        log "✅ Health check passed! (HTTP $HTTP_CODE)"
        return 0
    else
        error "❌ Health check failed! (HTTP $HTTP_CODE)"
        return 1
    fi
}

# Main deployment
main() {
    log "🚀 Starting deployment for $APP_NAME"
    log "================================"
    
    # Check if directory exists
    if [ ! -d "$PROJECT_DIR" ]; then
        error "Project directory not found: $PROJECT_DIR"
    fi
    
    cd $PROJECT_DIR
    log "📂 Working directory: $(pwd)"
    
    # Check git status
    if [ -d ".git" ]; then
        log "🔍 Checking git status..."
        git status --short
    else
        error "Not a git repository!"
    fi
    
    # Create backup
    create_backup
    
    # Stash changes
    log "💾 Stashing local changes..."
    git stash
    
    # Fetch and pull
    log "🔄 Fetching latest changes..."
    git fetch origin
    
    log "🔄 Pulling from $BRANCH..."
    git checkout $BRANCH
    git pull origin $BRANCH
    
    # Show recent commits
    log "📝 Recent commits:"
    git log -3 --oneline
    
    # Check for package.json changes
    if git diff HEAD@{1} HEAD --name-only | grep -q "package.json"; then
        log "📦 package.json changed, installing dependencies..."
        npm install --production
    else
        log "✅ No dependency changes detected"
    fi
    
    # Check if app was running
    WAS_RUNNING=false
    if check_app_status; then
        WAS_RUNNING=true
        log "🔄 Reloading PM2 process..."
        pm2 reload $APP_NAME --update-env
    else
        log "▶️  Starting PM2 process..."
        pm2 start ecosystem.config.js --only $APP_NAME
    fi
    
    # Wait and check status
    sleep 2
    pm2 status $APP_NAME
    
    # Run health check
    if ! health_check; then
        warning "Health check failed. Rolling back..."
        if [ "$WAS_RUNNING" = true ]; then
            pm2 restart $APP_NAME
        fi
        error "Deployment failed!"
    fi
    
    # Show logs
    log "📋 Recent logs:"
    pm2 logs $APP_NAME --lines 15 --nostream
    
    log "================================"
    log "✅ Deployment completed successfully!"
    log "🌐 Check: https://api-staging.bridgebond.ai"
}

# Run main function
main
```

Make executable:

```bash
chmod +x ~/deploy-with-checks.sh
```

### 13.6 Rollback Script

```bash
nano ~/rollback.sh
```

```bash
#!/bin/bash
# Rollback Script

set -e

APP_NAME=$1
BACKUP_DIR=$2

if [ -z "$APP_NAME" ] || [ -z "$BACKUP_DIR" ]; then
    echo "Usage: ./rollback.sh <app-name> <backup-directory>"
    echo "Example: ./rollback.sh backend-staging /home/ubuntu/backups/20241114-143022"
    exit 1
fi

echo "🔙 Rolling back $APP_NAME from $BACKUP_DIR"
read -p "Are you sure? (yes/no): " confirm

if [ "$confirm" != "yes" ]; then
    echo "❌ Rollback cancelled"
    exit 0
fi

# Stop app
pm2 stop $APP_NAME

# Restore backup
PROJECT_DIR=$(pm2 describe $APP_NAME | grep 'exec cwd' | awk '{print $4}')
echo "📂 Restoring to: $PROJECT_DIR"

# Backup current state first
EMERGENCY_BACKUP="/home/ubuntu/backups/emergency-$(date +%Y%m%d-%H%M%S)"
cp -r $PROJECT_DIR $EMERGENCY_BACKUP
echo "💾 Emergency backup created: $EMERGENCY_BACKUP"

# Restore
rm -rf $PROJECT_DIR
cp -r $BACKUP_DIR $PROJECT_DIR

# Restart app
pm2 restart $APP_NAME

echo "✅ Rollback complete!"
pm2 logs $APP_NAME --lines 20
```

Make executable:

```bash
chmod +x ~/rollback.sh
```

**Usage:**

```bash
./rollback.sh backend-staging /home/ubuntu/backups/20241114-143022
```

### 13.7 AWS CLI Configuration (for automated backups to S3)

```bash
# Install AWS CLI
sudo apt install -y awscli

# Configure AWS CLI
aws configure

# Enter your credentials:
# AWS Access Key ID: [your-key]
# AWS Secret Access Key: [your-secret]
# Default region: us-east-1
# Default output format: json
```

**Backup to S3 Script:**

```bash
nano ~/backup-to-s3.sh
```

```bash
#!/bin/bash
# Backup to AWS S3

DATE=$(date +%Y%m%d-%H%M%S)
BACKUP_DIR="/home/ubuntu/backups"
S3_BUCKET="s3://your-backup-bucket"

# MongoDB backup
mongodump --uri="mongodb://bridgebond_staging:password@localhost:27017/bridgebond_staging" \
  --out=$BACKUP_DIR/mongo-$DATE

# PostgreSQL backup (if using)
# pg_dump -U bridgebond_staging bridgebond_staging > $BACKUP_DIR/postgres-$DATE.sql

# Compress
tar -czf $BACKUP_DIR/backup-$DATE.tar.gz -C $BACKUP_DIR mongo-$DATE

# Upload to S3
aws s3 cp $BACKUP_DIR/backup-$DATE.tar.gz $S3_BUCKET/

# Cleanup local backups older than 7 days
find $BACKUP_DIR -name "*.tar.gz" -mtime +7 -delete

echo "✅ Backup completed and uploaded to S3"
```

```bash
chmod +x ~/backup-to-s3.sh
```

**Schedule with cron:**

```bash
crontab -e
```

Add:

```bash
# Daily backup at 2 AM
0 2 * * * /home/ubuntu/backup-to-s3.sh >> /home/ubuntu/backup.log 2>&1
```

---

## 14. Complete Command Reference

### 14.1 PM2 Commands

```bash
# Start applications
pm2 start ecosystem.config.js
pm2 start ecosystem.config.js --only backend-staging
pm2 start npm --name "app-name" -- start -- --port 3000

# Stop/Restart
pm2 stop backend-staging
pm2 restart backend-staging
pm2 reload backend-staging  # Zero-downtime reload
pm2 delete backend-staging

# Logs
pm2 logs backend-staging
pm2 logs backend-staging --lines 100
pm2 logs backend-staging --err  # Error logs only
pm2 flush  # Clear all logs

# Monitoring
pm2 monit
pm2 status
pm2 list
pm2 describe backend-staging

# Save configuration
pm2 save
pm2 startup  # Generate startup script
pm2 unstartup  # Remove startup script
```

### 14.2 Nginx Commands

```bash
# Test configuration
sudo nginx -t

# Reload (no downtime)
sudo systemctl reload nginx

# Restart
sudo systemctl restart nginx

# Start/Stop
sudo systemctl start nginx
sudo systemctl stop nginx

# Status
sudo systemctl status nginx

# View logs
sudo tail -f /var/log/nginx/error.log
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/api-staging-error.log
```

### 14.3 SSL/Certbot Commands

```bash
# Get certificate
sudo certbot --nginx -d api.bridgebond.ai
sudo certbot certonly --standalone -d api.bridgebond.ai

# List certificates
sudo certbot certificates

# Renew all certificates
sudo certbot renew

# Test renewal
sudo certbot renew --dry-run

# Revoke certificate
sudo certbot revoke --cert-name api.bridgebond.ai

# Check auto-renewal timer
sudo systemctl status certbot.timer
```

### 14.4 Git Commands

```bash
# Clone with specific SSH key
git clone git@github.com-adil-f:Adil7767/bridge-bond-api.git

# Update
git pull origin staging

# Check current branch
git branch

# Switch branch
git checkout staging

# View commit history
git log --oneline -10

# Stash changes
git stash
git stash pop
git stash list
```

### 14.5 Database Commands

**MongoDB:**

```bash
# Connect
mongosh -u bridgebond_staging -p password --authenticationDatabase bridgebond_staging

# Backup
mongodump --uri="mongodb://user:pass@localhost:27017/dbname"

# Restore
mongorestore --uri="mongodb://user:pass@localhost:27017/dbname" --archive=backup.gz --gzip

# Check status
sudo systemctl status mongod
sudo systemctl restart mongod
```

**PostgreSQL:**

```bash
# Connect
psql -U bridgebond_staging -d bridgebond_staging

# Backup
pg_dump -U username dbname > backup.sql

# Restore
psql -U username dbname < backup.sql

# List databases
psql -U postgres -c "\l"

# Check status
sudo systemctl status postgresql
sudo systemctl restart postgresql
```

### 14.6 System Commands

```bash
# Check disk usage
df -h

# Check memory
free -h

# Check CPU/Process usage
htop
top

# Check port usage
sudo netstat -tulpn | grep :2323
sudo ss -tulnp | grep :2323

# Check running services
sudo systemctl list-units --type=service --state=running

# Reboot server
sudo reboot

# Check system logs
sudo journalctl -xe
sudo journalctl -u nginx -f
sudo journalctl -u mongod -f
```

---

## 15. Troubleshooting Common Issues

### Issue 1: SSL Certificate Fails (Timeout during connect)

**Problem:** Certbot failed to authenticate - Timeout during connect

**Solutions:**

```bash
# 1. Check DNS is pointing to server
dig api.bridgebond.ai
# Should return your server IP

# 2. Check port 80 is open in AWS Security Group
# Go to AWS Console → EC2 → Security Groups → Add inbound rule for port 80

# 3. Check Nginx is listening on port 80
sudo ss -tulnp | grep :80

# 4. Test from outside
curl -I http://api.bridgebond.ai
# Should NOT timeout

# 5. Check UFW firewall
sudo ufw status
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# 6. Try standalone mode
sudo systemctl stop nginx
sudo certbot certonly --standalone -d api.bridgebond.ai
sudo systemctl start nginx
```

### Issue 2: PM2 App Won't Start

**Problem:** App shows "errored" or "stopped" status

**Solutions:**

```bash
# 1. Check logs
pm2 logs backend-staging --err --lines 50

# 2. Check if port is already in use
sudo netstat -tulpn | grep :2323

# 3. Kill process on port
sudo kill -9 $(sudo lsof -t -i:2323)

# 4. Check environment variables
pm2 describe backend-staging | grep env

# 5. Delete and restart
pm2 delete backend-staging
pm2 start ecosystem.config.js --only backend-staging

# 6. Check file permissions
ls -la ~/bridge-bond/staging/bridge-bond-api/.env

# 7. Test app manually (without PM2)
cd ~/bridge-bond/staging/bridge-bond-api
PORT=2323 node src/index.js
```

### Issue 3: Nginx 502 Bad Gateway

**Problem:** Nginx shows 502 error

**Solutions:**

```bash
# 1. Check if backend is running
pm2 status backend-staging

# 2. Check Nginx error logs
sudo tail -50 /var/log/nginx/error.log

# 3. Test backend directly
curl http://localhost:2323

# 4. Check Nginx upstream configuration
sudo nginx -t
sudo nano /etc/nginx/sites-available/api-staging.bridgebond.ai
# Verify upstream points to correct port

# 5. Restart both services
pm2 restart backend-staging
sudo systemctl restart nginx
```

### Issue 4: MongoDB Connection Failed

**Problem:** App can't connect to MongoDB

**Solutions:**

```bash
# 1. Check MongoDB is running
sudo systemctl status mongod

# 2. Test connection manually
mongosh -u bridgebond_staging -p password --authenticationDatabase bridgebond_staging

# 3. Check MongoDB logs
sudo tail -50 /var/log/mongodb/mongod.log

# 4. Verify authentication is enabled
sudo nano /etc/mongod.conf
# Check: security.authorization: enabled

# 5. Check connection string in .env
cat ~/bridge-bond/staging/bridge-bond-api/.env | grep MONGODB

# 6. Restart MongoDB
sudo systemctl restart mongod
```

### Issue 5: Git Clone Fails (SSH Issues)

**Problem:** Permission denied (publickey) or Could not resolve hostname

**Solutions:**

```bash
# 1. Test SSH connection
ssh -T git@github.com-adil-f

# 2. Check SSH config
cat ~/.ssh/config

# 3. Check SSH key permissions
ls -la ~/.ssh/
chmod 600 ~/.ssh/adil-f
chmod 600 ~/.ssh/config

# 4. Add SSH key to agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/adil-f

# 5. Verify key is added to GitHub
cat ~/.ssh/adil-f.pub
# Copy and add to GitHub Settings → SSH Keys

# 6. Use correct hostname
git clone git@github.com-adil-f:Adil7767/bridge-bond-api.git
# NOT: git@github.com:Adil7767/bridge-bond-api.git
```

### Issue 6: Deployment Script Fails

**Problem:** Deployment script exits with errors

**Solutions:**

```bash
# 1. Run script with debug mode
bash -x ~/deploy-staging.sh

# 2. Check if git is clean
cd ~/bridge-bond/staging/bridge-bond-api
git status

# 3. Reset to remote
git fetch origin
git reset --hard origin/staging

# 4. Check permissions
ls -la ~/deploy-staging.sh
chmod +x ~/deploy-staging.sh

# 5. Run commands manually one by one
cd ~/bridge-bond/staging/bridge-bond-api
git pull origin staging
npm install
pm2 restart backend-staging
```

### Issue 7: High Memory Usage

**Problem:** Server runs out of memory

**Solutions:**

```bash
# 1. Check memory usage
free -h
htop

# 2. Check which process uses memory
pm2 monit

# 3. Reduce PM2 instances in ecosystem.config.js
nano ~/ecosystem.config.js
# Change: instances: 1 (instead of 2 or more)

# 4. Set memory limit
# In ecosystem.config.js:
max_memory_restart: '500M'

# 5. Clear PM2 logs
pm2 flush

# 6. Clear system cache
sudo sync
echo 3 | sudo tee /proc/sys/vm/drop_caches

# 7. Upgrade server instance type
# AWS Console → EC2 → Instance → Change instance type
```

### Issue 8: Port Already in Use

**Problem:** Error: listen EADDRINUSE: address already in use :::2323

**Solutions:**

```bash
# 1. Find process using port
sudo lsof -i :2323
sudo netstat -tulpn | grep :2323

# 2. Kill the process
sudo kill -9 <PID>
# Or kill all on port
sudo kill -9 $(sudo lsof -t -i:2323)

# 3. Check PM2 list
pm2 list
# Delete duplicates
pm2 delete 0
pm2 delete backend-staging

# 4. Restart with correct port
PORT=2323 pm2 start ecosystem.config.js --only backend-staging
```

---

## 16. Security Best Practices

### 16.1 SSH Security

```bash
# 1. Change SSH port (optional)
sudo nano /etc/ssh/sshd_config
# Change: Port 22 → Port 2222

# 2. Disable root login
# PermitRootLogin no

# 3. Disable password authentication
# PasswordAuthentication no

# 4. Restart SSH
sudo systemctl restart sshd

# ⚠️ Test new SSH settings before closing session!
# Open new terminal and test: ssh -p 2222 -i key.pem ubuntu@server
```

### 16.2 Firewall Configuration

```bash
# Enable UFW
sudo ufw enable

# Allow SSH (change port if modified)
sudo ufw allow 22/tcp

# Allow HTTP/HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Allow specific IPs only for admin tasks
sudo ufw allow from YOUR_IP to any port 22

# Check status
sudo ufw status verbose
```

### 16.3 Fail2Ban Setup

```bash
# Install
sudo apt install -y fail2ban

# Configure
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local

# Add SSH protection:
[sshd]
enabled = true
port = 22
maxretry = 3
bantime = 3600

# Restart
sudo systemctl restart fail2ban

# Check status
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

### 16.4 Environment Variables Security

```bash
# Never commit .env files
echo ".env" >> .gitignore

# Set strict permissions
chmod 600 .env

# Use different secrets for staging/production

# Generate strong secrets:
openssl rand -base64 32
```

### 16.5 Regular Updates

```bash
# Enable automatic security updates
sudo apt install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades

# Manual updates
sudo apt update
sudo apt upgrade -y

# Update npm packages
npm outdated
npm update
```

---

## 17. Monitoring & Logging

### 17.1 PM2 Monitoring

```bash
# Real-time monitoring
pm2 monit

# Install PM2 log rotation
pm2 install pm2-logrotate

# Configure log rotation
pm2 set pm2-logrotate:max_size 10M
pm2 set pm2-logrotate:retain 7
pm2 set pm2-logrotate:compress true

# View logs
pm2 logs --lines 100
```

### 17.2 Setup Log Rotation

```bash
# Create logrotate config
sudo nano /etc/logrotate.d/bridge-bond
```

```bash
/home/ubuntu/bridge-bond/*/logs/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 ubuntu ubuntu
    sharedscripts
}
```

### 17.3 System Monitoring Script

```bash
nano ~/monitor.sh
```

```bash
#!/bin/bash

echo "=== System Resources ==="
echo "Memory Usage:"
free -h | grep "Mem:"
echo ""

echo "Disk Usage:"
df -h | grep "/$"
echo ""

echo "CPU Load:"
uptime
echo ""

echo "=== PM2 Status ==="
pm2 status
echo ""

echo "=== Nginx Status ==="
sudo systemctl status nginx --no-pager
echo ""

echo "=== Database Status ==="
sudo systemctl status mongod --no-pager
# Or for PostgreSQL:
# sudo systemctl status postgresql --no-pager
```

```bash
chmod +x ~/monitor.sh
```

---

## 18. Quick Start Checklist

### New Project Setup Checklist

- [ ] AWS EC2 instance created with proper security groups (ports 22, 80, 443 open)
- [ ] SSH key configured and able to connect
- [ ] System updated: `sudo apt update && sudo apt upgrade -y`
- [ ] Node.js installed: `node --version`
- [ ] PM2 installed: `pm2 --version`
- [ ] Nginx installed: `nginx -v`
- [ ] Database installed (MongoDB or PostgreSQL)
- [ ] SSH keys generated for GitHub accounts
- [ ] Projects cloned in proper directory structure
- [ ] .env files created with correct values
- [ ] DNS A records pointing to server IP
- [ ] Nginx configurations created for each domain
- [ ] SSL certificates obtained: `sudo certbot certificates`
- [ ] PM2 ecosystem configured
- [ ] Apps started and running: `pm2 status`
- [ ] Deployment scripts created and tested
- [ ] Firewall configured: `sudo ufw status`
- [ ] Automated backups configured
- [ ] Monitoring setup complete

---

## 19. Final Directory Structure

After complete setup, your directory structure should look like:

```
/home/ubuntu/
├── bridge-bond/
│   ├── staging/
│   │   ├── bridge-bond-api/
│   │   │   ├── src/
│   │   │   ├── .env
│   │   │   ├── ecosystem.config.js
│   │   │   └── package.json
│   │   └── logs/
│   │       ├── err.log
│   │       └── out.log
│   └── production/
│       ├── bridge-bond-api/
│       │   ├── src/
│       │   ├── .env
│       │   └── package.json
│       └── logs/
├── react-apps/
│   ├── staging/
│   │   └── bridge-bond-frontend/
│   │       └── build/
│   └── production/
│       └── bridge-bond-frontend/
│           └── build/
├── backups/
│   ├── 20241114-120000/
│   └── 20241114-140000/
├── ecosystem.config.js
├── deploy-staging.sh
├── deploy-production.sh
├── deploy-with-checks.sh
├── rollback.sh
├── backup-to-s3.sh
├── restart-apps.sh
└── monitor.sh
```

---

## 20. Support & Resources

### Useful Commands Reference Card

```bash
# Quick status check
pm2 status && sudo systemctl status nginx && sudo systemctl status mongod

# Quick restart all
pm2 restart all && sudo systemctl reload nginx

# View all logs
pm2 logs --lines 50

# Check what's using ports
sudo netstat -tulpn | grep -E ':(2323|2324|80|443)'

# Test SSL certificates
sudo certbot certificates

# Check system resources
htop
```

### Important Files to Backup

- `.env` files
- `ecosystem.config.js`
- Nginx configs in `/etc/nginx/sites-available/`
- Database dumps
- SSH keys in `~/.ssh/`

### Before Going Live

- [ ] Test all endpoints
- [ ] Run health checks
- [ ] Check SSL certificates expiry
- [ ] Verify database backups working
- [ ] Test deployment scripts
- [ ] Monitor logs for errors
- [ ] Load test your applications
- [ ] Setup monitoring/alerts

---

## Conclusion

This guide covers complete deployment setup for:

✅ Multiple GitHub accounts with SSH  
✅ PM2 process management for multiple apps  
✅ Nginx reverse proxy with multiple domains  
✅ SSL certificates with auto-renewal  
✅ MongoDB and PostgreSQL setup  
✅ Automated deployment scripts  
✅ Error handling and rollback procedures  
✅ Security hardening  
✅ Monitoring and logging  
✅ Edge cases (PuTTY keys, multiple ports, etc.)

### Need Help?

- Check logs: `pm2 logs` and `sudo tail -f /var/log/nginx/error.log`
- Test configurations: `sudo nginx -t` and `pm2 describe <app-name>`
- Review this guide's troubleshooting section

### Remember:

- Always test in staging before production
- Keep backups before major changes
- Monitor your applications regularly
- Keep systems and dependencies updated

**Good luck with your deployment! 🚀**

---

**Last Updated**: 2024  
**Maintained by**: Bridge Bond Team

