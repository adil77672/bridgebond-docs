# AWS Deployment Guide - Bridgebond Super Admin

## Overview

This comprehensive guide covers deploying the Bridgebond Super Admin application to AWS EC2 using PM2 for process management, Nginx as a reverse proxy, and Let's Encrypt for SSL certificates.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Server Setup](#server-setup)
3. [Environment Setup Guide](#environment-setup-guide)
4. [Application Deployment](#application-deployment)
5. [Nginx Configuration](#nginx-configuration)
6. [SSL Certificate Setup](#ssl-certificate-setup)
7. [PM2 Management](#pm2-management)
8. [Deployment Scripts](#deployment-scripts)
9. [Security Considerations](#security-considerations)
10. [Troubleshooting](#troubleshooting)
11. [Monitoring](#monitoring)
12. [CI/CD Integration](#cicd-integration)
13. [Backup Strategy](#backup-strategy)

---

## Prerequisites

Before starting the deployment, ensure you have:

- **AWS EC2 instance** (Ubuntu 20.04+ recommended)
- **SSH access** to the EC2 instance
- **Domain name configured** (optional, for custom domain)
- **Node.js 18+** and **Yarn** installed on the server
- **PM2 installed** globally
- **Git repository access** with appropriate SSH keys configured

---

## Server Setup

### 1. Connect to EC2 Instance

```bash
ssh -i your-key.pem ubuntu@your-ec2-ip
```

### 2. Install Node.js and Yarn

```bash
# Install Node.js 18.x
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install Yarn
npm install -g yarn

# Verify installations
node --version
yarn --version
```

**Expected Output:**
```
v18.x.x
1.22.x
```

### 3. Install PM2

```bash
# Install PM2 globally
npm install -g pm2

# Setup PM2 to start on system boot
pm2 startup

# Follow the instructions to run the generated command
# It will look like:
# sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u ubuntu --hp /home/ubuntu

# Save PM2 process list
pm2 save
```

### 4. Install Nginx (for reverse proxy)

```bash
# Update package list
sudo apt update

# Install Nginx
sudo apt install nginx -y

# Start and enable Nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# Verify Nginx is running
sudo systemctl status nginx
```

### 5. Install Certbot (for SSL certificates)

```bash
# Install Certbot and Nginx plugin
sudo apt install certbot python3-certbot-nginx -y

# Verify installation
certbot --version
```

### 6. Configure Firewall (UFW)

```bash
# Check UFW status
sudo ufw status

# Allow HTTP, HTTPS, and SSH
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Enable UFW
sudo ufw enable

# Verify rules
sudo ufw status verbose
```

---

## Environment Setup Guide

### Overview

This section explains how to configure environment variables for different deployment environments (Development, Staging, Production, Test).

### Environment Files

The application uses environment-specific configuration files:

- `.env.development` - Local development
- `.env.staging` - Staging environment
- `.env.production` - Production environment
- `.env.test` - Test environment
- `.env.example` - Template file

### Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `VITE_API_URL` | Base URL for the API | `https://api.bridgebond.ai/v1` |
| `VITE_ONESIGNAL_APP_ID` | OneSignal App ID for push notifications | `97518421-dcf2-446b-a170-f5c07c462bab` |

### Environment-Specific Configuration

#### Development Environment

**File:** `.env.development`

```env
# Development Environment Variables
VITE_API_URL=http://localhost:2323/v1
VITE_ONESIGNAL_APP_ID=97518421-dcf2-446b-a170-f5c07c462bab
```

**Usage:**

```bash
yarn dev
# or
yarn build:development
```

#### Staging Environment

**File:** `.env.staging`

```env
# Staging Environment Variables
VITE_API_URL=https://staging-api.bridgebond.ai/v1
VITE_ONESIGNAL_APP_ID=97518421-dcf2-446b-a170-f5c07c462bab
```

**Usage:**

```bash
yarn build:staging
```

**PM2 Configuration:**
- Port: `5178`
- API URL: `https://staging-api.bridgebond.ai/v1`
- Nginx: `staging-admin.bridgebond.ai`

#### Production Environment

**File:** `.env.production`

```env
# Production Environment Variables
VITE_API_URL=https://api.bridgebond.ai/v1
VITE_ONESIGNAL_APP_ID=97518421-dcf2-446b-a170-f5c07c462bab
```

**Usage:**

```bash
yarn build:production
```

**PM2 Configuration:**
- Port: `5173`
- API URL: `https://api.bridgebond.ai/v1`
- Nginx: `admin.bridgebond.ai`

#### Test Environment

**File:** `.env.test`

```env
# Test Environment Variables
VITE_API_URL=https://test-api.bridgebond.ai/v1
VITE_ONESIGNAL_APP_ID=97518421-dcf2-446b-a170-f5c07c462bab
```

**Usage:**

```bash
yarn build --mode test
```

### API URLs by Environment

| Environment | API Base URL | Domain | Port |
|------------|--------------|--------|------|
| Development | `http://localhost:2323/v1` | `localhost:5173` | 5178 |
| Staging | `https://staging-api.bridgebond.ai/v1` | `staging-admin.bridgebond.ai` | 5178 |
| Production | `https://api.bridgebond.ai/v1` | `admin.bridgebond.ai` | 5173 |
| Test | `https://test-api.bridgebond.ai/v1` | `test-admin.bridgebond.ai` | 5178 |

### Setup Instructions

#### 1. Create Environment Files

Copy the example file and update values:

```bash
# For staging
cp .env.example .env.staging
nano .env.staging

# For production
cp .env.example .env.production
nano .env.production
```

#### 2. Update Values

Edit the environment files with your specific values:

```env
VITE_API_URL=https://your-api-url.com/v1
VITE_ONESIGNAL_APP_ID=your-onesignal-app-id
```

#### 3. Build for Specific Environment

```bash
# Development
yarn build:development

# Staging
yarn build:staging

# Production
yarn build:production
```

### Vite Environment Variable Rules

1. **Prefix Required**: All environment variables must be prefixed with `VITE_` to be exposed to the client
2. **Build Time**: Environment variables are embedded at build time, not runtime
3. **Mode-Based**: Vite automatically loads `.env.[mode]` files based on the `--mode` flag

### PM2 Environment Configuration

The `ecosystem.config.cjs` file includes environment variables for each PM2 process:

```javascript
{
  name: 'staging-superadmin',
  env: {
    PORT: 5178,
    NODE_ENV: 'staging',
    VITE_API_URL: 'https://staging-api.bridgebond.ai/v1',
  }
}
```

**Note:** For PM2, you can also override environment variables:

```bash
VITE_API_URL=https://custom-api.com/v1 pm2 start ecosystem.config.cjs --only staging-superadmin
```

### Vite Configuration

The `vite.config.js` file handles environment-based API URLs:

```javascript
export default defineConfig(({ mode }) => {
  const apiUrl = process.env.VITE_API_URL || 
    (mode === 'staging' ? 'https://staging-api.bridgebond.ai/v1' :
     mode === 'production' ? 'https://api.bridgebond.ai/v1' :
     'http://localhost:2323/v1');

  return {
    define: {
      'import.meta.env.VITE_API_URL': JSON.stringify(apiUrl),
    },
  };
});
```

### Security Best Practices for Environment Variables

#### 1. Never Commit .env Files

Ensure `.env*` files are in `.gitignore`:

```gitignore
.env
.env.local
.env.*.local
.env.staging
.env.production
.env.test
```

#### 2. Use .env.example as Template

Keep `.env.example` in version control with placeholder values:

```env
VITE_API_URL=your-api-url-here
VITE_ONESIGNAL_APP_ID=your-onesignal-app-id
```

#### 3. Restrict File Permissions

On production servers:

```bash
chmod 600 .env.production
chmod 600 .env.staging
```

#### 4. Use AWS Secrets Manager (Production)

For production, consider using AWS Secrets Manager:

```bash
# Install AWS CLI
aws secretsmanager get-secret-value --secret-id prod/api-keys

# Or use environment variables from CI/CD
```

---

## Application Deployment

### 1. Clone Repository

```bash
# Create application directory
mkdir -p ~/bridge-bond/projects
cd ~/bridge-bond/projects

# Clone your repository
git clone https://github.com/your-org/bridgebond-super-admin.git bridgebond_super_Admin

# Navigate to project directory
cd bridgebond_super_Admin
```

**Note:** If using SSH with multiple GitHub accounts, use the appropriate SSH host:

```bash
# For default account
git clone git@github.com:your-org/bridgebond-super-admin.git bridgebond_super_Admin

# For specific account (if configured in ~/.ssh/config)
git clone git@github.com-adil-f:your-org/bridgebond-super-admin.git bridgebond_super_Admin
```

### 2. Install Dependencies

```bash
cd ~/bridge-bond/projects/bridgebond_super_Admin

# Install dependencies
yarn install
```

### 3. Configure Environment Variables

Create environment files based on your deployment environment:

#### For Staging:

```bash
cp .env.example .env.staging
nano .env.staging
```

Update with:

```env
VITE_API_URL=https://staging-api.bridgebond.ai/v1
VITE_ONESIGNAL_APP_ID=97518421-dcf2-446b-a170-f5c07c462bab
```

#### For Production:

```bash
cp .env.example .env.production
nano .env.production
```

Update with:

```env
VITE_API_URL=https://api.bridgebond.ai/v1
VITE_ONESIGNAL_APP_ID=97518421-dcf2-446b-a170-f5c07c462bab
```

**Set proper permissions:**

```bash
chmod 600 .env.staging
chmod 600 .env.production
```

### 4. Build Application

```bash
# For staging
yarn build:staging

# For production
yarn build:production
```

The build output will be in the `dist/` directory.

### 5. Configure PM2 Ecosystem File

Create or update `ecosystem.config.cjs`:

```javascript
module.exports = {
  apps: [
    {
      name: 'staging-superadmin',
      script: 'serve',
      args: '-s dist -l 5178',
      env: {
        PORT: 5178,
        NODE_ENV: 'staging',
        VITE_API_URL: 'https://staging-api.bridgebond.ai/v1',
      },
      error_file: './logs/staging-err.log',
      out_file: './logs/staging-out.log',
      log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
      instances: 1,
      exec_mode: 'fork',
      autorestart: true,
      watch: false,
      max_memory_restart: '1G',
    },
    {
      name: 'production-superadmin',
      script: 'serve',
      args: '-s dist -l 5173',
      env: {
        PORT: 5173,
        NODE_ENV: 'production',
        VITE_API_URL: 'https://api.bridgebond.ai/v1',
      },
      error_file: './logs/prod-err.log',
      out_file: './logs/prod-out.log',
      log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
      instances: 1,
      exec_mode: 'fork',
      autorestart: true,
      watch: false,
      max_memory_restart: '1G',
    },
  ],
};
```

**Note:** You'll need to install `serve` globally:

```bash
npm install -g serve
```

### 6. Start with PM2

```bash
# Create logs directory
mkdir -p logs

# Start staging
pm2 start ecosystem.config.cjs --only staging-superadmin

# Or start production
pm2 start ecosystem.config.cjs --only production-superadmin

# Save PM2 process list
pm2 save
```

### 7. Verify PM2 Status

```bash
# View all processes
pm2 list

# View logs
pm2 logs staging-superadmin
pm2 logs production-superadmin

# Check if app is responding
curl http://localhost:5178  # Staging
curl http://localhost:5173  # Production
```

---

## Nginx Configuration

### 1. Create Nginx Configuration

Create Nginx configuration file:

```bash
sudo nano /etc/nginx/sites-available/bridgebond-superadmin
```

### 2. Add Configuration

Paste the following configuration:

```nginx
# Staging Configuration
server {
    listen 80;
    server_name staging-admin.bridgebond.ai;

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

# Production Configuration
server {
    listen 80;
    server_name admin.bridgebond.ai;

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

### 3. Enable the Site

```bash
# Create symbolic link
sudo ln -s /etc/nginx/sites-available/bridgebond-superadmin /etc/nginx/sites-enabled/

# Test Nginx configuration
sudo nginx -t

# If test passes, reload Nginx
sudo systemctl reload nginx
```

### 4. Verify Nginx Status

```bash
# Check Nginx status
sudo systemctl status nginx

# View Nginx error logs
sudo tail -f /var/log/nginx/error.log

# View access logs
sudo tail -f /var/log/nginx/access.log
```

---

## SSL Certificate Setup

### 1. Configure DNS Records

Before obtaining SSL certificates, ensure your DNS records are configured:

**For Staging:**
- Type: `A`
- Name: `staging-admin`
- Value: `YOUR_EC2_PUBLIC_IP`
- TTL: `3600`

**For Production:**
- Type: `A`
- Name: `admin`
- Value: `YOUR_EC2_PUBLIC_IP`
- TTL: `3600`

**Verify DNS:**

```bash
# Check DNS resolution
dig staging-admin.bridgebond.ai
dig admin.bridgebond.ai

# Or use ping
ping staging-admin.bridgebond.ai
ping admin.bridgebond.ai
```

### 2. Ensure Port 80 is Open

**Check AWS EC2 Security Group:**
- Go to AWS Console → EC2 → Security Groups
- Find your instance's security group
- Ensure inbound rule for HTTP (port 80) exists:
  - Type: `HTTP`
  - Port: `80`
  - Source: `0.0.0.0/0`

**Check UFW Firewall:**

```bash
sudo ufw status
# If active, ensure port 80 is allowed
sudo ufw allow 80/tcp
```

### 3. Obtain SSL Certificates

```bash
# Get SSL certificates for both domains
sudo certbot --nginx -d staging-admin.bridgebond.ai -d admin.bridgebond.ai

# Follow the prompts:
# - Enter your email address
# - Agree to terms (Y)
# - Share email? (N is fine)
```

**Expected Output:**
```
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/staging-admin.bridgebond.ai/fullchain.pem
Key is saved at: /etc/letsencrypt/live/staging-admin.bridgebond.ai/privkey.pem
```

### 4. Verify SSL Configuration

Certbot automatically updates your Nginx configuration. Verify:

```bash
# Test Nginx configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx

# Test SSL certificates
sudo certbot certificates
```

### 5. Setup Auto-Renewal

Certbot automatically sets up auto-renewal. Verify:

```bash
# Test auto-renewal
sudo certbot renew --dry-run

# Check certbot timer status
sudo systemctl status certbot.timer
```

### 6. Verify HTTPS Access

```bash
# Test from server
curl -I https://staging-admin.bridgebond.ai
curl -I https://admin.bridgebond.ai

# Or open in browser
# https://staging-admin.bridgebond.ai
# https://admin.bridgebond.ai
```

---

## PM2 Management

### Basic Commands

```bash
# View all processes
pm2 list

# View logs
pm2 logs staging-superadmin
pm2 logs production-superadmin

# View logs with lines limit
pm2 logs staging-superadmin --lines 50

# Restart application
pm2 restart staging-superadmin
pm2 restart production-superadmin

# Stop application
pm2 stop staging-superadmin

# Delete from PM2
pm2 delete staging-superadmin

# Monitor resources
pm2 monit

# View detailed info
pm2 show staging-superadmin
```

### Advanced Commands

```bash
# Reload application (zero-downtime)
pm2 reload staging-superadmin

# Restart all applications
pm2 restart all

# Stop all applications
pm2 stop all

# Delete all applications
pm2 delete all

# Save current process list
pm2 save

# Resurrect saved processes
pm2 resurrect
```

### Log Management

```bash
# View error logs only
pm2 logs staging-superadmin --err

# View output logs only
pm2 logs staging-superadmin --out

# Clear logs
pm2 flush

# Install log rotation
pm2 install pm2-logrotate

# Configure log rotation
pm2 set pm2-logrotate:max_size 10M
pm2 set pm2-logrotate:retain 7
pm2 set pm2-logrotate:compress true
```

---

## Deployment Scripts

The project includes organized deployment scripts for both **frontend** and **backend** applications. Scripts are located in the `scripts/` directory with separate folders for frontend and backend.

### Directory Structure

```
scripts/
├── frontend/
│   ├── deploy-staging.sh
│   ├── deploy-production.sh
│   └── README.md
├── backend/
│   ├── deploy-staging.sh
│   ├── deploy-production.sh
│   └── README.md
├── install-deploy-scripts.sh
└── README.md
```

### Installation

To install all deployment scripts to your home directory:

```bash
# From the project root directory
cd ~/bridge-bond
./scripts/install-deploy-scripts.sh
```

This will:
- Copy all scripts to `~/deploy-scripts/`
- Create convenient symlinks in `~/`
- Make all scripts executable

### Frontend Deployment Scripts

Located in `scripts/super-admin/`:

**Staging:**
- Pulls latest code from `staging` branch
- Installs dependencies with `yarn install`
- Builds for staging with `yarn build:staging`
- Restarts PM2 `staging-superadmin` process

**Production:**
- Requires confirmation before proceeding
- Pulls latest code from `main` branch
- Installs dependencies with `yarn install`
- Builds for production with `yarn build:production`
- Restarts PM2 `production-superadmin` process

**Usage:**
```bash
# Using symlinks (recommended)
~/deploy-super-admin-staging.sh
~/deploy-super-admin-production.sh

# Or using full paths
~/deploy-scripts/super-admin/deploy-super-admin-staging.sh
~/deploy-scripts/super-admin/deploy-super-admin-production.sh
```

**Configuration:**
- Project: `$HOME/bridge-bond/projects/bridgebond_super_Admin`
- Staging Branch: `staging`
- Production Branch: `main`
- PM2 Apps: `staging-superadmin` (Port 5178), `production-superadmin` (Port 5173)

### Backend Deployment Scripts

Located in `scripts/backend/`:

**Staging:**
- Pulls latest code from `staging` branch
- Installs dependencies with `npm install`
- Restarts PM2 `backend-staging` process

**Production:**
- Requires confirmation before proceeding
- Pulls latest code from `main` branch
- Installs dependencies with `npm install`
- Restarts PM2 `backend-prod` process

**Usage:**
```bash
# Using symlinks (recommended)
~/deploy-backend-staging.sh
~/deploy-backend-production.sh

# Or using full paths
~/deploy-scripts/backend/deploy-staging.sh
~/deploy-scripts/backend/deploy-production.sh
```

**Configuration:**
- Project: `$HOME/bridge-bond/projects/bridge-bond-api`
- Staging Branch: `staging`
- Production Branch: `main`
- PM2 Apps: `backend-staging` (Port 2323), `backend-prod` (Port 2324)

### Quick Deployment Examples

**Deploy Everything to Staging:**
```bash
# Deploy backend first
~/deploy-backend-staging.sh

# Then deploy Super Admin
~/deploy-super-admin-staging.sh
```

**Deploy Everything to Production:**
```bash
# Deploy backend first (requires confirmation)
~/deploy-backend-production.sh

# Then deploy Super Admin (requires confirmation)
~/deploy-super-admin-production.sh
```

### Customizing Scripts

Edit the scripts to change:
- `PROJECT_DIR` - Path to your project
- `BRANCH` - Git branch name
- `PM2_APP_NAME` - PM2 process name

**Example:**
```bash
# Edit Super Admin staging script
nano ~/deploy-scripts/super-admin/deploy-super-admin-staging.sh

# Edit backend production script
nano ~/deploy-scripts/backend/deploy-production.sh
```

### Script Features

All deployment scripts include:
- ✅ Color-coded output (green/yellow/red)
- ✅ Error handling (exits on error)
- ✅ Automatic PM2 management (start if not running, restart if running)
- ✅ Status display after deployment
- ✅ Log directory creation
- ✅ PM2 state saving
- ✅ Production confirmation prompts

### Documentation

For detailed information about each script:
- Super Admin: See `scripts/super-admin/README.md`
- Backend: See `scripts/backend/README.md`
- General: See `scripts/README.md`

---

## Security Considerations

### 1. Firewall Configuration

```bash
# Allow HTTP, HTTPS, and SSH
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Enable UFW
sudo ufw enable

# Verify rules
sudo ufw status verbose
```

### 2. Environment Variables Security

- **Never commit `.env` files** to version control
- **Use AWS Secrets Manager** or Parameter Store for sensitive values in production
- **Restrict file permissions:**

```bash
chmod 600 .env.production
chmod 600 .env.staging
```

### 3. PM2 Logs

Logs are stored in `./logs/` directory. Ensure proper log rotation:

```bash
# PM2 handles log rotation automatically, but you can configure it
pm2 install pm2-logrotate
pm2 set pm2-logrotate:max_size 10M
pm2 set pm2-logrotate:retain 7
pm2 set pm2-logrotate:compress true
```

### 4. SSH Security

```bash
# Disable password authentication (use keys only)
sudo nano /etc/ssh/sshd_config

# Set:
PasswordAuthentication no
PubkeyAuthentication yes

# Restart SSH
sudo systemctl restart sshd
```

### 5. Regular Updates

```bash
# Update system packages regularly
sudo apt update
sudo apt upgrade -y

# Update Node.js and npm
npm install -g npm@latest
```

---

## Troubleshooting

### Application Not Starting

```bash
# Check PM2 logs
pm2 logs staging-superadmin --lines 50

# Check if port is in use
sudo netstat -tulpn | grep 5178
sudo netstat -tulpn | grep 5173

# Check PM2 status
pm2 status

# Check if serve is installed
which serve
npm list -g serve
```

### Nginx Issues

```bash
# Test Nginx configuration
sudo nginx -t

# Check Nginx error logs
sudo tail -f /var/log/nginx/error.log

# Check Nginx access logs
sudo tail -f /var/log/nginx/access.log

# Restart Nginx
sudo systemctl restart nginx

# Check if Nginx is running
sudo systemctl status nginx
```

### Build Issues

```bash
# Clear node_modules and reinstall
rm -rf node_modules dist
yarn install
yarn build:staging

# Check Node.js version
node --version

# Check Yarn version
yarn --version
```

### SSL Certificate Issues

```bash
# Check certificate status
sudo certbot certificates

# Test certificate renewal
sudo certbot renew --dry-run

# Check Certbot logs
sudo tail -f /var/log/letsencrypt/letsencrypt.log

# Re-obtain certificate if needed
sudo certbot --nginx -d staging-admin.bridgebond.ai --force-renewal
```

### Environment Variables Not Loading

1. **Check file name**: Must be `.env.[mode]` (e.g., `.env.staging`)
2. **Check build command**: Must use `--mode` flag (e.g., `yarn build:staging`)
3. **Check variable prefix**: Must start with `VITE_`
4. **Rebuild application**: Environment variables are embedded at build time

### API URL Not Updating

1. **Rebuild application**: Environment variables are embedded at build time
2. **Check vite.config.js**: Ensure it's reading the correct environment
3. **Check PM2 env**: Verify `ecosystem.config.cjs` has correct values

### Verify Environment Variables

Check what's being used in the browser console:

```javascript
console.log(import.meta.env.VITE_API_URL);
```

---

## Monitoring

### PM2 Monitoring

```bash
# Real-time monitoring
pm2 monit

# View process info
pm2 info staging-superadmin

# View process metrics
pm2 show staging-superadmin
```

### Application Health Check

Create a simple health check endpoint or monitor the application directly:

```bash
# Check if application is responding
curl http://localhost:5178  # Staging
curl http://localhost:5173   # Production

# Check via domain
curl https://staging-admin.bridgebond.ai
curl https://admin.bridgebond.ai
```

### System Monitoring

```bash
# Check system resources
htop

# Check disk space
df -h

# Check memory usage
free -h

# Check CPU usage
top
```

### Log Monitoring

```bash
# PM2 logs
pm2 logs staging-superadmin --lines 100

# Nginx logs
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log

# System logs
sudo journalctl -u nginx -f
```

---

## CI/CD Integration

### GitHub Actions Example

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to AWS

on:
  push:
    branches:
      - main
      - staging

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to staging
        if: github.ref == 'refs/heads/staging'
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.AWS_HOST }}
          username: ${{ secrets.AWS_USER }}
          key: ${{ secrets.AWS_SSH_KEY }}
          script: |
            cd ~/bridge-bond/projects/bridgebond_super_Admin
            git pull
            yarn install
            yarn build:staging
            pm2 restart staging-superadmin
      
      - name: Deploy to production
        if: github.ref == 'refs/heads/main'
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.AWS_HOST }}
          username: ${{ secrets.AWS_USER }}
          key: ${{ secrets.AWS_SSH_KEY }}
          script: |
            cd ~/bridge-bond/projects/bridgebond_super_Admin
            git pull
            yarn install
            yarn build:production
            pm2 restart production-superadmin
```

### AWS CodePipeline

Set environment variables in the build stage:

```json
{
  "environmentVariables": [
    {
      "name": "VITE_API_URL",
      "value": "https://staging-api.bridgebond.ai/v1"
    },
    {
      "name": "VITE_ONESIGNAL_APP_ID",
      "value": "97518421-dcf2-446b-a170-f5c07c462bab"
    }
  ]
}
```

---

## Backup Strategy

### 1. Application Files

```bash
# Backup application directory
tar -czf backup-$(date +%Y%m%d).tar.gz ~/bridge-bond/projects/bridgebond_super_Admin

# Backup to S3 (if AWS CLI configured)
aws s3 cp backup-$(date +%Y%m%d).tar.gz s3://your-backup-bucket/
```

### 2. PM2 Configuration

```bash
# Backup PM2 process list
pm2 save

# Copy PM2 dump file
cp ~/.pm2/dump.pm2 /backup/location/
```

### 3. Environment Files

```bash
# Backup environment files
cp .env.staging ~/backups/.env.staging.$(date +%Y%m%d)
cp .env.production ~/backups/.env.production.$(date +%Y%m%d)
```

### 4. Automated Backup Script

Create `backup.sh`:

```bash
#!/bin/bash

BACKUP_DIR="$HOME/backups"
DATE=$(date +%Y%m%d_%H%M%S)
PROJECT_DIR="$HOME/bridge-bond/projects/bridgebond_super_Admin"

mkdir -p "$BACKUP_DIR"

# Backup application
tar -czf "$BACKUP_DIR/app-backup-$DATE.tar.gz" "$PROJECT_DIR"

# Backup environment files
cp "$PROJECT_DIR/.env.staging" "$BACKUP_DIR/.env.staging.$DATE"
cp "$PROJECT_DIR/.env.production" "$BACKUP_DIR/.env.production.$DATE"

# Backup PM2 config
pm2 save
cp ~/.pm2/dump.pm2 "$BACKUP_DIR/pm2-dump.$DATE"

echo "Backup completed: $BACKUP_DIR"
```

Make it executable and add to crontab:

```bash
chmod +x backup.sh

# Add to crontab (daily at 2 AM)
crontab -e
# Add: 0 2 * * * /path/to/backup.sh
```

---

## Updating the Application

### 1. Pull Latest Changes

```bash
cd ~/bridge-bond/projects/bridgebond_super_Admin

# For staging
git checkout staging
git pull origin staging

# For production
git checkout main
git pull origin main
```

### 2. Install New Dependencies (if any)

```bash
yarn install
```

### 3. Rebuild Application

```bash
# For staging
yarn build:staging

# For production
yarn build:production
```

### 4. Restart PM2

```bash
pm2 restart staging-superadmin

# or

pm2 restart production-superadmin
```

### 5. Verify Deployment

```bash
# Check PM2 status
pm2 status

# Check application logs
pm2 logs staging-superadmin --lines 20

# Test application
curl https://staging-admin.bridgebond.ai
```

---

## Migration Guide

### Updating API URLs

1. Update `.env.[environment]` file
2. Update `ecosystem.config.cjs` if using PM2
3. Rebuild application: `yarn build:[environment]`
4. Restart PM2: `pm2 restart [app-name]`

### Adding New Environment

1. Create `.env.[new-env]` file
2. Add build script to `package.json`:

   ```json
   "build:new-env": "vite build --mode new-env"
   ```

3. Add PM2 config to `ecosystem.config.cjs`
4. Update Nginx configuration
5. Update documentation

---

## Additional Resources

- [PM2 Documentation](https://pm2.keymetrics.io/docs/usage/quick-start/)
- [Nginx Documentation](https://nginx.org/en/docs/)
- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [Let's Encrypt Documentation](https://letsencrypt.org/docs/)
- [Vite Environment Variables](https://vitejs.dev/guide/env-and-mode.html)
- [PM2 Environment Variables](https://pm2.keymetrics.io/docs/usage/environment/)
- [AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/)

---

## Support

For issues or questions, contact the development team or refer to the project's issue tracker.

---

## Quick Reference Commands

```bash
# PM2
pm2 list
pm2 logs staging-superadmin
pm2 restart staging-superadmin
pm2 monit

# Nginx
sudo nginx -t
sudo systemctl reload nginx
sudo tail -f /var/log/nginx/error.log

# SSL
sudo certbot certificates
sudo certbot renew --dry-run

# Deployment
./deploy-staging.sh
./deploy-production.sh

# System
htop
df -h
free -h
```

---

**Last Updated:** 2025-01-13
**Version:** 1.0.0

