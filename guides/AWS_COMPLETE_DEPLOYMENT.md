# Complete AWS Ubuntu Deployment Guide for api.bridgebond.ai

This is a comprehensive, step-by-step guide to deploy Bridge Bond API on AWS EC2 Ubuntu server with Nginx, SSL, PM2, and all necessary components.

## 📋 Table of Contents

1. [AWS EC2 Instance Setup](#1-aws-ec2-instance-setup)
2. [Initial Server Connection](#2-initial-server-connection)
3. [System Updates & Essential Tools](#3-system-updates--essential-tools)
4. [Git Installation & Repository Setup](#4-git-installation--repository-setup)
5. [Node.js & npm Installation](#5-nodejs--npm-installation)
6. [MongoDB Installation & Configuration](#6-mongodb-installation--configuration)
7. [PM2 Installation & Configuration](#7-pm2-installation--configuration)
8. [Application Setup](#8-application-setup)
9. [Nginx Installation & Configuration](#9-nginx-installation--configuration)
10. [SSL Certificate Setup (Let's Encrypt)](#10-ssl-certificate-setup-lets-encrypt)
11. [Firewall Configuration (UFW)](#11-firewall-configuration-ufw)
12. [DNS Configuration](#12-dns-configuration)
13. [Testing & Verification](#13-testing--verification)
14. [Monitoring & Maintenance](#14-monitoring--maintenance)
15. [Troubleshooting](#15-troubleshooting)
16. [Real-World Deployment Issues & Solutions](#16-real-world-deployment-issues--solutions)

---

## 1. AWS EC2 Instance Setup

### Step 1.1: Create EC2 Instance

1. **Login to AWS Console**
   - Go to https://console.aws.amazon.com
   - Navigate to **EC2 Dashboard**

2. **Launch Instance**
   - Click **"Launch Instance"** (orange button)
   - **Name**: `bridge-bond-api-server`

3. **Choose AMI (Amazon Machine Image)**
   - Select **Ubuntu Server 22.04 LTS** (or latest LTS)
   - Architecture: **64-bit (x86)**

4. **Instance Type**
   - **Recommended**: `t3.medium` (2 vCPU, 4 GB RAM)
   - **Budget option**: `t3.small` (2 vCPU, 2 GB RAM)
   - **Minimum**: `t3.micro` (2 vCPU, 1 GB RAM) - for testing only

5. **Key Pair (IMPORTANT!)**
   - Click **"Create new key pair"**
   - **Name**: `bridge-bond-key`
   - **Key pair type**: RSA
   - **Private key file format**: `.pem`
   - Click **"Create key pair"**
   - **⚠️ DOWNLOAD AND SAVE THIS FILE SECURELY** - You'll need it to SSH into your server!

6. **Network Settings**
   - Click **"Edit"** in Network settings
   - **VPC**: Default VPC (or your custom VPC)
   - **Subnet**: Public subnet
   - **Auto-assign Public IP**: Enable
   - **Security Group**: Create new security group
     - **Security group name**: `bridge-bond-sg`
     - **Description**: Security group for Bridge Bond API
     - **Inbound Rules**:
       ```
       Type          Protocol    Port Range    Source              Description
       SSH           TCP         22            My IP              SSH access
       HTTP          TCP         80            0.0.0.0/0          Web traffic
       HTTPS         TCP         443           0.0.0.0/0          Secure web traffic
       Custom TCP    TCP         2323          0.0.0.0/0          Node.js app (optional, for direct access)
       ```
     - **Outbound Rules**: All traffic (default)

7. **Storage**
   - **Root volume**: 30 GB gp3 SSD
   - **Volume type**: gp3
   - **Delete on termination**: Uncheck if you want to keep data

8. **Launch Instance**
   - Review all settings
   - Click **"Launch Instance"**
   - Wait 2-3 minutes for instance to start

### Step 1.2: Get Instance Details

1. Go to **EC2 Dashboard → Instances**
2. Click on your instance
3. Note the following:
   - **Public IPv4 address** (e.g., `54.123.45.67`)
   - **Instance ID** (e.g., `i-0123456789abcdef0`)
   - **Security Group ID**

### Step 1.3: Allocate Elastic IP (Optional but Recommended)

For a static IP address:

1. Go to **EC2 → Elastic IPs**
2. Click **"Allocate Elastic IP address"**
3. Click **"Allocate"**
4. Select the Elastic IP → **Actions → Associate Elastic IP address**
5. Select your instance and click **"Associate"**

**Note**: Use this Elastic IP for your DNS A record.

---

## 2. Initial Server Connection

### Step 2.1: Connect via SSH (Mac/Linux)

```bash
# Navigate to where you saved your key
cd ~/Downloads  # or wherever you saved it

# Set correct permissions (IMPORTANT!)
chmod 400 bridge-bond-key.pem

# Connect to server (replace with YOUR IP)
ssh -i bridge-bond-key.pem ubuntu@YOUR_SERVER_IP

# Example:
ssh -i bridge-bond-key.pem ubuntu@54.123.45.67
```

### Step 2.2: Connect via SSH (Windows)

**Option 1: Using Windows Terminal/PowerShell**
```powershell
# Navigate to key location
cd C:\Users\YourName\Downloads

# Connect
ssh -i bridge-bond-key.pem ubuntu@YOUR_SERVER_IP
```

**Option 2: Using PuTTY**
1. Download PuTTY and PuTTYgen
2. Convert `.pem` to `.ppk` using PuTTYgen
3. Use PuTTY to connect with the `.ppk` file

### Step 2.3: Verify Connection

You should see:
```
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-xxx-generic x86_64)

ubuntu@ip-172-31-xx-xx:~$
```

✅ **Checkpoint**: You're now connected to your server!

---

## 3. System Updates & Essential Tools

### Step 3.1: Update System Packages

```bash
# Update package lists
sudo apt update

# Upgrade all packages (takes 5-10 minutes)
sudo apt upgrade -y

# Install essential build tools and utilities
sudo apt install -y \
    build-essential \
    curl \
    wget \
    git \
    vim \
    nano \
    htop \
    unzip \
    zip \
    ufw \
    fail2ban \
    software-properties-common \
    apt-transport-https \
    ca-certificates \
    gnupg \
    lsb-release
```

### Step 3.2: Configure Timezone

```bash
# Set timezone to UTC (or your preferred timezone)
sudo timedatectl set-timezone UTC

# Verify
timedatectl
```

### Step 3.3: Configure Hostname

```bash
# Set hostname
sudo hostnamectl set-hostname bridge-bond-api

# Add to hosts file
echo "127.0.0.1 bridge-bond-api" | sudo tee -a /etc/hosts

# Verify
hostname
```

### Step 3.4: Setup Automatic Security Updates

```bash
# Enable automatic security updates
sudo dpkg-reconfigure -plow unattended-upgrades

# Verify
sudo systemctl status unattended-upgrades
```

### Step 3.5: Configure Fail2Ban (Brute Force Protection)

```bash
# Enable and start fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

# Check status
sudo fail2ban-client status
```

✅ **Checkpoint**: System is updated and secured!

---

## 4. Git Installation & Repository Setup

### Step 4.1: Install Git

```bash
# Git is usually pre-installed, verify
git --version

# If not installed
sudo apt install -y git
```

### Step 4.2: Configure Git

```bash
# Set your Git identity
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set default branch name
git config --global init.defaultBranch main

# Configure line endings (for cross-platform)
git config --global core.autocrlf input
```

### Step 4.3: Generate SSH Key for Git (Optional)

```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"

# Press Enter to accept default location
# Press Enter for no passphrase (or set one)

# Start SSH agent
eval "$(ssh-agent -s)"

# Add key to agent
ssh-add ~/.ssh/id_ed25519

# Display public key (add this to GitHub/GitLab)
cat ~/.ssh/id_ed25519.pub
```

**To add to GitHub:**
1. Copy the output from `cat ~/.ssh/id_ed25519.pub`
2. Go to GitHub → Settings → SSH and GPG keys → New SSH key
3. Paste and save

### Step 4.4: Configure Multiple GitHub Accounts (Advanced)

If you need to use multiple GitHub accounts from the same server, configure SSH with multiple keys:

```bash
# Navigate to .ssh directory
cd ~/.ssh

# List existing keys
ls -la

# If you have keys with different names (e.g., adil, adil-f, waleed)
# Create or edit SSH config
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

**Set proper permissions:**
```bash
# Set permissions for private keys
chmod 600 ~/.ssh/adil ~/.ssh/adil-f ~/.ssh/waleed

# Set permissions for public keys
chmod 644 ~/.ssh/adil.pub ~/.ssh/adil-f.pub ~/.ssh/waleed.pub

# Set permissions for config
chmod 600 ~/.ssh/config
```

**Test each SSH connection:**
```bash
# Test default account
ssh -T git@github.com

# Test Adil-F account
ssh -T git@github.com-adil-f

# Test Waleed account
ssh -T git@github.com-waleed
```

**Usage when cloning repositories:**
```bash
# Using default account
git clone git@github.com:username/repo.git

# Using Adil-F account
git clone git@github.com-adil-f:username/repo.git

# Using Waleed account
git clone git@github.com-waleed:username/repo.git
```

**Note:** Make sure each public key (`.pub` file) is added to the corresponding GitHub account's SSH keys.

### Step 4.5: Clone Repository

```bash
# Navigate to home directory
cd ~

# Clone repository (HTTPS)
git clone https://github.com/ITSOLSTUDIOLTD/bridgebond-backend.git bridge-bond

# Or using SSH (if configured)
# git clone git@github.com:ITSOLSTUDIOLTD/bridgebond-backend.git bridge-bond

# Navigate to project
cd bridge-bond

# Verify files
ls -la
```

✅ **Checkpoint**: Repository cloned successfully!

---

## 5. Node.js & npm Installation

### Step 5.1: Install Node.js 20 LTS

```bash
# Add NodeSource repository for Node.js 20
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

# Install Node.js
sudo apt install -y nodejs

# Verify installation
node --version   # Should show v20.x.x
npm --version    # Should show 10.x.x
```

### Step 5.2: Install Yarn (Optional)

```bash
# Install Yarn
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt update
sudo apt install -y yarn

# Verify
yarn --version
```

### Step 5.3: Update npm to Latest

```bash
# Update npm to latest version
sudo npm install -g npm@latest

# Verify
npm --version
```

✅ **Checkpoint**: Node.js and npm installed!

---

## 6. MongoDB Installation & Configuration

### Step 6.1: Install MongoDB 7.0

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

# Enable MongoDB on boot
sudo systemctl enable mongod

# Check status
sudo systemctl status mongod
```

You should see **"active (running)"** in green.

### Step 6.2: Create MongoDB Admin User

```bash
# Connect to MongoDB
mongosh
```

In MongoDB shell, run:

```javascript
// Switch to admin database
use admin

// Create admin user (CHANGE THE PASSWORD!)
db.createUser({
  user: "admin",
  pwd: "YourStrongAdminPassword123!",
  roles: [ { role: "userAdminAnyDatabase", db: "admin" }, "readWriteAnyDatabase" ]
})

// Switch to your app database
use bridgebond

// Create app user (CHANGE THE PASSWORD!)
db.createUser({
  user: "bridgebond",
  pwd: "YourStrongAppPassword456!",
  roles: [ { role: "readWrite", db: "bridgebond" } ]
})

// Exit MongoDB shell
exit
```

### Step 6.3: Enable MongoDB Authentication

```bash
# Edit MongoDB config
sudo nano /etc/mongod.conf
```

Find the `#security:` section and change it to:

```yaml
security:
  authorization: enabled
```

**To save in nano:**
- Press `Ctrl + X`
- Press `Y` (yes)
- Press `Enter`

```bash
# Restart MongoDB
sudo systemctl restart mongod

# Test connection with your new credentials
mongosh -u bridgebond -p YourStrongAppPassword456! --authenticationDatabase bridgebond
```

If it connects, type `exit` to leave.

### Step 6.4: Secure MongoDB (Bind to Localhost Only)

```bash
# Edit MongoDB config
sudo nano /etc/mongod.conf
```

Ensure the `net` section looks like:

```yaml
net:
  port: 27017
  bindIp: 127.0.0.1  # Only localhost
```

```bash
# Restart MongoDB
sudo systemctl restart mongod
```

✅ **Checkpoint**: MongoDB installed and secured!

---

## 7. PM2 Installation & Configuration

### Step 7.1: Install PM2

```bash
# Install PM2 globally
sudo npm install -g pm2

# Verify
pm2 --version
```

### Step 7.2: Setup PM2 Startup Script

```bash
# Generate startup script
pm2 startup systemd
```

You'll see a command to run. **Copy and paste that command** (it will look like):

```bash
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u ubuntu --hp /home/ubuntu
```

Run it, then:

```bash
# Save PM2 configuration
pm2 save
```

### Step 7.3: Configure PM2 Log Rotation

```bash
# Install PM2 log rotation module
pm2 install pm2-logrotate

# Configure log rotation
pm2 set pm2-logrotate:max_size 10M
pm2 set pm2-logrotate:retain 7
pm2 set pm2-logrotate:compress true
pm2 set pm2-logrotate:dateFormat YYYY-MM-DD_HH-mm-ss
```

✅ **Checkpoint**: PM2 installed and configured!

---

## 8. Application Setup

### Step 8.1: Install Dependencies

```bash
# Navigate to project directory
cd ~/bridge-bond

# Install dependencies
npm install

# Or with yarn
# yarn install
```

### Step 8.2: Create Environment File

```bash
# Create .env file
nano .env
```

Paste this template and **FILL IN YOUR REAL VALUES**:

```env
# Environment
NODE_ENV=production

# Server
PORT=2323
HOST=0.0.0.0
BASE_URL=https://api.bridgebond.ai

# Database (USE YOUR PASSWORD FROM STEP 6.2!)
MONGODB_URL_STAGING=mongodb://bridgebond:YourStrongAppPassword456!@localhost:27017/bridgebond?authSource=bridgebond
MONGODB_URL_LIVE=mongodb://bridgebond:YourStrongAppPassword456!@localhost:27017/bridgebond?authSource=bridgebond
IS_LIVE=true

# JWT Configuration (GENERATE STRONG SECRETS - minimum 32 characters!)
JWT_SECRET=your-very-strong-secret-minimum-32-chars-long-change-this-immediately
JWT_ACCESS_EXPIRATION_MINUTES=2880
JWT_REFRESH_EXPIRATION_DAYS=5
JWT_RESET_PASSWORD_EXPIRATION_MINUTES=10
JWT_VERIFY_EMAIL_EXPIRATION_MINUTES=10

# Email Configuration
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_SERVICE=Gmail
SMTP_USERNAME=your-email@gmail.com
SMTP_PASSWORD=your-gmail-app-password
EMAIL_FROM=noreply@bridgebond.ai

# Cloudinary (if using)
CLOUDINARY_CLOUD_NAME=your-cloud-name
CLOUDINARY_API_KEY=your-api-key
CLOUDINARY_API_SECRET=your-api-secret

# AWS S3 (if using)
AWS_S3_BUCKET=your-bucket-name
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
AWS_REGION=us-east-1

# OneSignal (if using)
ONESIGNAL_APP_ID=your-app-id
ONESIGNAL_REST_API_KEY=your-rest-api-key

# Stripe (if using)
STRIPE_SECRET_KEY=your-secret-key
STRIPE_PUBLISHABLE_KEY=your-publishable-key
STRIPE_WEBHOOK_SECRET=your-webhook-secret

# Merge.dev (if using)
MERGE_API_KEY=your-merge-api-key
MERGE_WEBHOOK_SECRET=your-webhook-secret
```

**To save in nano:**
- Press `Ctrl + X`
- Press `Y` (yes)
- Press `Enter`

### Step 8.3: Secure Environment File

```bash
# Set proper permissions
chmod 600 .env

# Verify
ls -la .env
```

### Step 8.4: Create Logs Directory

```bash
# Create logs directory for PM2
mkdir -p logs

# Verify
ls -la logs
```

### Step 8.5: Initialize Database

```bash
# Create super admin (first time only)
npm run create-superadmin

# Or seed database (development only - NOT recommended for production)
# npm run seed
```

### Step 8.6: Start Application with PM2

**Important:** PM2 doesn't have a `--port` flag. The port must be set via environment variable or in `ecosystem.config.json`.

**Option 1: Using ecosystem.config.json (Recommended)**

```bash
# Start application using ecosystem.config.json
pm2 start ecosystem.config.json --only backend-staging

# Or start all apps
pm2 start ecosystem.config.json

# Check status
pm2 status

# View logs
pm2 logs backend-staging --lines 50

# Save PM2 configuration
pm2 save
```

**Option 2: Direct start with environment variable**

```bash
# Start with specific port (overrides any default in code)
PORT=2323 pm2 start src/index.js --name backend-staging

# This sets process.env.PORT=2323, so your app will run on port 2323
# regardless of what's in package.json or default code
```

**Verify it's running:**
```bash
# Check PM2 status
pm2 list

# Should show:
# ┌─────┬──────────────────┬────────┬──────┬────────┬────────┐
# │ id  │ name             │ status │ mode │ pid    │ port   │
# ├─────┼──────────────────┼────────┼──────┼────────┼────────┤
# │ 0   │ backend-staging  │ online │ fork │ 13245  │ 2323   │
# └─────┴──────────────────┴────────┴──────┴────────┴────────┘

# Test local connection
curl http://localhost:2323
```

**Note:** If your code uses `const PORT = process.env.PORT || 3000;`, the PORT in `ecosystem.config.json` will override the default. The `--port` flag doesn't exist in PM2 - always use environment variables.

### Step 8.7: Verify Application is Running

```bash
# Test local connection
curl http://localhost:2323

# Test health endpoint (if available)
curl http://localhost:2323/health

# Check PM2 status
pm2 list

# View real-time logs
pm2 logs backend-staging
```

✅ **Checkpoint**: Application is running on port 2323!

---

## 9. Nginx Installation & Configuration

### Step 9.1: Install Nginx

```bash
# Install Nginx
sudo apt install -y nginx

# Start Nginx
sudo systemctl start nginx

# Enable Nginx on boot
sudo systemctl enable nginx

# Check status
sudo systemctl status nginx
```

### Step 9.2: Create Nginx Configuration for api.bridgebond.ai

```bash
# Create Nginx configuration file
sudo nano /etc/nginx/sites-available/api.bridgebond.ai
```

Paste the following complete configuration:

```nginx
# Upstream configuration for Bridge Bond API
upstream bridge_bond_api {
    server 127.0.0.1:2323;
    keepalive 64;
}

# HTTP Server - Redirect to HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name api.bridgebond.ai;

    # Allow Let's Encrypt verification
    location /.well-known/acme-challenge/ {
        root /var/www/html;
        allow all;
    }

    # Redirect all other HTTP traffic to HTTPS
    location / {
        return 301 https://$server_name$request_uri;
    }
}

# HTTPS Server - Main Configuration
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name api.bridgebond.ai;

    # SSL Certificate paths (will be configured by Certbot)
    ssl_certificate /etc/letsencrypt/live/api.bridgebond.ai/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.bridgebond.ai/privkey.pem;
    
    # SSL Configuration - Modern and Secure
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
    add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;

    # Logging
    access_log /var/log/nginx/api.bridgebond.ai-access.log;
    error_log /var/log/nginx/api.bridgebond.ai-error.log warn;

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
        proxy_pass http://bridge_bond_api;
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

    # Health check endpoint (optional)
    location /health {
        access_log off;
        proxy_pass http://bridge_bond_api/health;
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

**To save in nano:**
- Press `Ctrl + X`
- Press `Y` (yes)
- Press `Enter`

### Step 9.3: Enable the Site

```bash
# Create symbolic link to enable the site
sudo ln -s /etc/nginx/sites-available/api.bridgebond.ai /etc/nginx/sites-enabled/

# Remove default site (optional)
sudo rm /etc/nginx/sites-enabled/default

# Test Nginx configuration
sudo nginx -t
```

You should see:
```
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

**⚠️ Don't reload Nginx yet!** We need SSL certificates first.

✅ **Checkpoint**: Nginx configured (but not active yet)!

---

## 10. SSL Certificate Setup (Let's Encrypt)

### Step 10.1: Configure DNS First (CRITICAL!)

**Before getting SSL, your domain MUST point to your server!**

1. Go to your DNS provider (GoDaddy, Namecheap, Cloudflare, Route53, etc.)
2. Add an **A record**:
   ```
   Type: A
   Name: api
   Value: YOUR_SERVER_IP (or Elastic IP)
   TTL: 3600 (or Auto)
   ```

3. **Verify DNS is working:**
   ```bash
   # Check if domain resolves to your server
   ping api.bridgebond.ai
   
   # Or use dig
   dig api.bridgebond.ai
   
   # Or use nslookup
   nslookup api.bridgebond.ai
   ```

   Wait 5-10 minutes for DNS to propagate if it doesn't work immediately.

### Step 10.2: Install Certbot

```bash
# Install Certbot and Nginx plugin
sudo apt install -y certbot python3-certbot-nginx

# Verify installation
certbot --version
```

### Step 10.3: Obtain SSL Certificate

**⚠️ CRITICAL PRE-FLIGHT CHECKLIST - Verify all before running Certbot:**

1. **DNS is configured correctly:**
   ```bash
   dig api.bridgebond.ai +short
   # Should return your server IP (e.g., 51.20.185.71)
   ```

2. **EC2 Security Group allows port 80:**
   - Go to AWS Console → EC2 → Security Groups
   - Ensure inbound rule: HTTP (TCP 80) from 0.0.0.0/0 exists
   - See troubleshooting section if you get "Timeout during connect" error

3. **UFW firewall allows port 80 (if active):**
   ```bash
   sudo ufw status
   sudo ufw allow 80/tcp  # If needed
   ```

4. **Nginx is running:**
   ```bash
   sudo systemctl status nginx
   # Should show "active (running)"
   ```

5. **Nginx is listening on port 80:**
   ```bash
   sudo netstat -tulpn | grep :80
   # Should show nginx listening
   ```

6. **Test from outside (from your local machine):**
   ```bash
   curl -I http://api.bridgebond.ai
   # Should return HTTP headers, not timeout
   ```

**If all checks pass, proceed:**

```bash
# Get SSL certificate (Certbot will automatically configure Nginx)
sudo certbot --nginx -d api.bridgebond.ai

# Follow the prompts:
# - Enter your email address
# - Agree to terms (Y)
# - Share email? (N is fine, or Y if you want)
# - Redirect HTTP to HTTPS? (2 - Redirect - recommended)
```

You should see:
```
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/api.bridgebond.ai/fullchain.pem
Key is saved at: /etc/letsencrypt/live/api.bridgebond.ai/privkey.pem
```

### Step 10.4: Verify SSL Certificate

```bash
# Check certificate status
sudo certbot certificates

# Test auto-renewal
sudo certbot renew --dry-run

# Check certbot timer status
sudo systemctl status certbot.timer
```

Certbot automatically renews certificates. The timer runs twice daily.

### Step 10.5: Reload Nginx

```bash
# Test Nginx configuration again
sudo nginx -t

# Reload Nginx to apply SSL configuration
sudo systemctl reload nginx

# Check status
sudo systemctl status nginx
```

✅ **Checkpoint**: SSL certificate installed and configured!

---

## 11. Firewall Configuration (UFW)

### Step 11.1: Configure UFW Firewall

```bash
# Check UFW status
sudo ufw status

# Set default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (IMPORTANT - do this first!)
sudo ufw allow 22/tcp

# Allow HTTP and HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Enable UFW
sudo ufw enable

# Type 'y' and press Enter when prompted

# Check status
sudo ufw status verbose
```

You should see:
```
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
```

### Step 11.2: Optional - Restrict SSH to Your IP

For better security, you can restrict SSH access to your IP only:

```bash
# Get your current IP
curl ifconfig.me

# Allow SSH only from your IP (replace with YOUR_IP)
sudo ufw delete allow 22/tcp
sudo ufw allow from YOUR_IP_ADDRESS to any port 22

# Verify
sudo ufw status numbered
```

✅ **Checkpoint**: Firewall configured!

---

## 12. DNS Configuration

### Step 12.1: Final DNS Verification

```bash
# Check DNS resolution
dig api.bridgebond.ai +short

# Should return your server's IP address

# Test from command line
curl -I https://api.bridgebond.ai
```

### Step 12.2: DNS Propagation Check

Use online tools to verify DNS propagation:
- https://dnschecker.org
- https://www.whatsmydns.net

Enter `api.bridgebond.ai` and check if it resolves to your server IP globally.

✅ **Checkpoint**: DNS configured correctly!

---

## 13. Testing & Verification

### Step 13.1: Test Application

```bash
# 1. Check if your app is running
pm2 status

# 2. Check app logs
pm2 logs backend-staging --lines 30

# 3. Test local connection
curl http://localhost:2323

# 4. Check Nginx status
sudo systemctl status nginx

# 5. Test domain (HTTP - should redirect to HTTPS)
curl -I http://api.bridgebond.ai

# 6. Test domain (HTTPS)
curl -I https://api.bridgebond.ai

# 7. Test API endpoint
curl https://api.bridgebond.ai/v1/docs
```

### Step 13.2: Test from Browser

1. Open your browser
2. Visit: `https://api.bridgebond.ai`
3. Visit: `https://api.bridgebond.ai/v1/docs` (API documentation)
4. Check SSL certificate (lock icon in browser)

### Step 13.3: Verify All Services

```bash
# Check all services status
echo "=== PM2 Status ==="
pm2 status

echo "=== Nginx Status ==="
sudo systemctl status nginx --no-pager

echo "=== MongoDB Status ==="
sudo systemctl status mongod --no-pager

echo "=== SSL Certificate ==="
sudo certbot certificates

echo "=== Firewall Status ==="
sudo ufw status
```

✅ **Checkpoint**: Everything is working!

---

## 14. Monitoring & Maintenance

### Step 14.1: PM2 Monitoring

```bash
# Real-time monitoring
pm2 monit

# Show process info
pm2 info backend-staging

# Show process tree
pm2 prettylist
```

### Step 14.2: Log Management

```bash
# View application logs
pm2 logs backend-staging

# View Nginx access logs
sudo tail -f /var/log/nginx/api.bridgebond.ai-access.log

# View Nginx error logs
sudo tail -f /var/log/nginx/api.bridgebond.ai-error.log

# View MongoDB logs
sudo tail -f /var/log/mongodb/mongod.log
```

### Step 14.3: System Monitoring

```bash
# Monitor system resources
htop

# Check disk usage
df -h

# Check memory usage
free -h

# Check network connections
sudo netstat -tulpn | grep 2323
```

### Step 14.4: Backup MongoDB

Create a backup script:

```bash
# Create backup directory
mkdir -p ~/backups/mongodb

# Create backup script
nano ~/backup-mongodb.sh
```

Paste:

```bash
#!/bin/bash
BACKUP_DIR="/home/ubuntu/backups/mongodb"
DATE=$(date +%Y%m%d_%H%M%S)
mkdir -p $BACKUP_DIR

# Backup database
mongodump --uri="mongodb://bridgebond:YourStrongAppPassword456!@localhost:27017/bridgebond?authSource=bridgebond" \
  --out=$BACKUP_DIR/$DATE

# Compress backup
tar -czf $BACKUP_DIR/$DATE.tar.gz -C $BACKUP_DIR $DATE
rm -rf $BACKUP_DIR/$DATE

# Delete backups older than 7 days
find $BACKUP_DIR -name "*.tar.gz" -mtime +7 -delete

echo "Backup completed: $BACKUP_DIR/$DATE.tar.gz"
```

```bash
# Make executable
chmod +x ~/backup-mongodb.sh

# Test backup
~/backup-mongodb.sh

# Setup daily backup (runs at 2 AM)
crontab -e

# Add this line:
0 2 * * * /home/ubuntu/backup-mongodb.sh >> /home/ubuntu/backup.log 2>&1
```

### Step 14.5: Update Application

```bash
# Navigate to project
cd ~/bridge-bond

# Pull latest code
git pull origin main

# Install new dependencies
npm install

# Restart application (zero-downtime for cluster mode)
pm2 reload backend-staging

# Or restart
pm2 restart backend-staging

# Check logs
pm2 logs backend-staging --lines 50
```

---

## 15. Troubleshooting

### Issue: Application Won't Start

```bash
# Check PM2 logs
pm2 logs backend-staging --err --lines 100

# Check if port is in use
sudo lsof -i :2323

# Check environment variables
pm2 env 0  # Replace 0 with app ID

# Restart application
pm2 restart backend-staging
```

### Issue: Nginx 502 Bad Gateway

```bash
# Check if app is running
pm2 status

# Check app logs
pm2 logs backend-staging

# Test local connection
curl http://localhost:2323

# Check Nginx error logs
sudo tail -50 /var/log/nginx/api.bridgebond.ai-error.log

# Restart Nginx
sudo systemctl restart nginx
```

### Issue: SSL Certificate Not Working / Certbot Timeout

**Common Error**: `Timeout during connect (likely firewall problem)`

This means Let's Encrypt cannot reach your server to verify domain ownership. Follow these steps:

#### Step 1: Verify DNS Configuration

```bash
# Check if DNS is pointing correctly
dig api.bridgebond.ai +short

# Should return your server's IP (e.g., 51.20.185.71)
# If it returns a different IP or nothing, fix DNS first

# Check DNS from multiple locations
nslookup api.bridgebond.ai

# Use online tools:
# - https://dnschecker.org
# - https://www.whatsmydns.net
```

**If DNS is incorrect:**
1. Go to your DNS provider
2. Add/update A record: `api.bridgebond.ai → YOUR_SERVER_IP`
3. Wait 5-10 minutes for propagation

#### Step 2: Check EC2 Security Group

**This is the most common cause!** Your Security Group must allow HTTP (port 80) and HTTPS (port 443).

```bash
# Find your Security Group ID
curl http://169.254.169.254/latest/meta-data/security-groups

# Or check in AWS Console:
# EC2 → Instances → Your Instance → Security → Security Groups
```

**In AWS Console:**
1. Go to **EC2 Dashboard → Instances → Select your instance**
2. Scroll to **Security → Security Groups**
3. Click the security group name
4. Go to **Inbound rules → Edit inbound rules**
5. **Add these rules** (if not present):

| Type  | Protocol | Port Range | Source    | Description        |
|-------|----------|------------|-----------|-------------------|
| HTTP  | TCP      | 80         | 0.0.0.0/0 | Let's Encrypt     |
| HTTPS | TCP      | 443        | 0.0.0.0/0 | HTTPS traffic     |

6. Click **"Save rules"**

**Verify with AWS CLI (if installed):**
```bash
# List security groups
aws ec2 describe-security-groups --group-ids <your-sg-id>

# Should show rules allowing ports 80 and 443
```

#### Step 3: Check Ubuntu Firewall (UFW)

```bash
# Check UFW status
sudo ufw status

# If active, allow Nginx Full (opens ports 80 and 443)
sudo ufw allow 'Nginx Full'

# Or manually allow ports
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Reload firewall
sudo ufw reload

# Verify
sudo ufw status verbose
```

**Expected output:**
```
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
```

#### Step 4: Verify Nginx is Listening on Port 80

```bash
# Check if Nginx is listening on port 80
sudo netstat -tulpn | grep nginx

# Should show:
# tcp        0      0 0.0.0.0:80          0.0.0.0:*          LISTEN      <nginx_pid>/nginx
# tcp6       0      0 :::80               :::*               LISTEN      <nginx_pid>/nginx

# If not, check Nginx status
sudo systemctl status nginx

# If not running, start it
sudo systemctl start nginx
sudo systemctl enable nginx
```

#### Step 5: Verify Nginx Configuration

```bash
# Test Nginx configuration
sudo nginx -t

# Should show: "nginx: configuration file /etc/nginx/nginx.conf test is successful"

# Check if your site config exists
ls -la /etc/nginx/sites-available/api.bridgebond.ai
ls -la /etc/nginx/sites-enabled/api.bridgebond.ai

# If missing, create it (see Step 9.2 in this guide)
# Then reload Nginx
sudo systemctl reload nginx
```

#### Step 6: Test Connectivity from Outside

**From your local machine (or another server):**

```bash
# Test HTTP connection
curl -I http://api.bridgebond.ai

# Should return HTTP headers (200, 301, or 302)
# If timeout → Security Group or firewall still blocking

# Test with verbose output
curl -v http://api.bridgebond.ai

# Test from multiple locations
# Use: https://www.yougetsignal.com/tools/open-ports/
# Enter your server IP and port 80
```

**If you get a response** → Port 80 is open, proceed to Step 7.

**If timeout** → Go back to Steps 2-3 (Security Group and UFW).

#### Step 7: Retry Certbot

```bash
# Try Certbot again
sudo certbot --nginx -d api.bridgebond.ai

# Follow prompts:
# - Enter email address
# - Agree to terms (Y)
# - Share email? (N or Y)
# - Redirect HTTP to HTTPS? (2 - Redirect)
```

**If successful**, you'll see:
```
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/api.bridgebond.ai/fullchain.pem
Key is saved at: /etc/letsencrypt/live/api.bridgebond.ai/privkey.pem
```

#### Step 8: Verify SSL Certificate

```bash
# Check certificate status
sudo certbot certificates

# Test from browser
# Visit: https://api.bridgebond.ai
# Should show lock icon

# Test from command line
curl -I https://api.bridgebond.ai

# Check certificate details
openssl s_client -connect api.bridgebond.ai:443 -servername api.bridgebond.ai
```

#### Additional Troubleshooting

**If Certbot still fails:**

```bash
# Check Certbot logs
sudo tail -50 /var/log/letsencrypt/letsencrypt.log

# Try standalone mode (requires stopping Nginx temporarily)
sudo systemctl stop nginx
sudo certbot certonly --standalone -d api.bridgebond.ai
sudo systemctl start nginx

# Then manually configure Nginx SSL (see Step 9.2)

# Check for rate limiting
# Let's Encrypt has rate limits (50 certs per week per domain)
# If exceeded, wait or use staging server:
sudo certbot --nginx -d api.bridgebond.ai --staging
```

**Common Certbot Errors:**

| Error | Solution |
|-------|----------|
| `Timeout during connect` | Check Security Group (port 80) and UFW firewall |
| `DNS problem: NXDOMAIN` | Fix DNS A record pointing to server IP |
| `Rate limit exceeded` | Wait or use `--staging` flag for testing |
| `Connection refused` | Ensure Nginx is running and listening on port 80 |
| `Invalid response` | Check Nginx configuration and server blocks |

**Quick Checklist:**
- [ ] DNS A record points to server IP
- [ ] EC2 Security Group allows port 80 (0.0.0.0/0)
- [ ] EC2 Security Group allows port 443 (0.0.0.0/0)
- [ ] UFW allows ports 80 and 443 (or 'Nginx Full')
- [ ] Nginx is running: `sudo systemctl status nginx`
- [ ] Nginx is listening on port 80: `sudo netstat -tulpn | grep :80`
- [ ] Nginx config is valid: `sudo nginx -t`
- [ ] Can reach server from outside: `curl http://api.bridgebond.ai`
- [ ] Domain resolves correctly: `dig api.bridgebond.ai`

### Issue: MongoDB Connection Failed

```bash
# Check MongoDB status
sudo systemctl status mongod

# Check MongoDB logs
sudo tail -50 /var/log/mongodb/mongod.log

# Test connection
mongosh -u bridgebond -p YourStrongAppPassword456! --authenticationDatabase bridgebond

# Restart MongoDB
sudo systemctl restart mongod
```

### Issue: Domain Not Resolving

```bash
# Check DNS
dig api.bridgebond.ai

# Check from different location
curl -I https://api.bridgebond.ai

# Verify DNS records in your DNS provider
# Wait for DNS propagation (can take up to 48 hours, usually 5-10 minutes)
```

### Issue: Port 2323 Not Accessible

```bash
# Check if app is listening
sudo netstat -tulpn | grep 2323

# Check firewall
sudo ufw status

# Check AWS Security Group
# Go to EC2 Console → Security Groups → Edit inbound rules
# Ensure port 2323 is open (if needed for direct access)
```

### Issue: High Memory Usage

```bash
# Check memory usage
free -h
pm2 monit

# Restart application
pm2 restart backend-staging

# Check for memory leaks in logs
pm2 logs backend-staging --err
```

---

## 16. Real-World Deployment Issues & Solutions

This section documents actual issues encountered during deployment of `api.bridgebond.ai` and their solutions.

### Issue 1: Certbot Timeout - Security Group Not Configured

**Problem:**
```
Error: Timeout during connect (likely firewall problem)
```

**Root Cause:**
- EC2 Security Group was missing inbound rules for ports 80 and 443
- Let's Encrypt couldn't reach the server to verify domain ownership

**Solution:**
1. **Find Security Group:**
   ```bash
   curl http://169.254.169.254/latest/meta-data/security-groups
   ```

2. **In AWS Console:**
   - EC2 → Instances → Select instance → Security → Security Groups
   - Edit Inbound Rules
   - Add:
     - HTTP (TCP 80) from 0.0.0.0/0
     - HTTPS (TCP 443) from 0.0.0.0/0
   - Save rules

3. **Verify from outside:**
   ```bash
   curl -I http://api.bridgebond.ai
   # Should return HTTP headers, not timeout
   ```

4. **Retry Certbot:**
   ```bash
   sudo certbot --nginx -d api.bridgebond.ai
   ```

**Reference:** Based on deployment experience with `api.bridgebond.ai`

---

### Issue 2: DNS Resolution Delay

**Problem:**
- Domain `api.bridgebond.ai` not resolving immediately after DNS configuration
- Certbot failing with DNS-related errors

**Root Cause:**
- DNS propagation takes time (5-10 minutes, sometimes up to 48 hours)
- DNS A record not properly configured

**Solution:**
1. **Verify DNS configuration:**
   ```bash
   dig api.bridgebond.ai +short
   # Should return server IP (e.g., 51.20.185.71)
   ```

2. **Check DNS propagation:**
   - Use https://dnschecker.org
   - Enter `api.bridgebond.ai` and check global resolution
   - Wait for all locations to show correct IP

3. **Verify A record in DNS provider:**
   ```
   Type: A
   Name: api
   Value: YOUR_SERVER_IP
   TTL: 3600
   ```

4. **Wait and retry:**
   - Wait 5-10 minutes after DNS changes
   - Retry Certbot after DNS propagates globally

**Reference:** Based on deployment experience with `api.bridgebond.ai`

---

### Issue 3: UFW Firewall Blocking Traffic

**Problem:**
- UFW firewall was active but not configured for HTTP/HTTPS
- External connections timing out

**Root Cause:**
- UFW default deny incoming policy blocked port 80
- Nginx couldn't receive external connections

**Solution:**
```bash
# Check UFW status
sudo ufw status

# Allow Nginx Full (opens ports 80 and 443)
sudo ufw allow 'Nginx Full'

# Or manually allow ports
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Reload firewall
sudo ufw reload

# Verify
sudo ufw status verbose
```

**Expected Output:**
```
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
```

**Reference:** Based on deployment experience with `api.bridgebond.ai`

---

### Issue 4: Nginx Not Listening on Port 80

**Problem:**
- Nginx running but not listening on port 80
- Certbot unable to verify domain

**Root Cause:**
- Nginx configuration not properly set up
- Site configuration not enabled

**Solution:**
```bash
# Check if Nginx is listening
sudo netstat -tulpn | grep :80

# If not listening, check Nginx status
sudo systemctl status nginx

# Check if site config exists
ls -la /etc/nginx/sites-available/api.bridgebond.ai
ls -la /etc/nginx/sites-enabled/api.bridgebond.ai

# If missing, create and enable (see Step 9.2)
sudo ln -s /etc/nginx/sites-available/api.bridgebond.ai /etc/nginx/sites-enabled/

# Test and reload Nginx
sudo nginx -t
sudo systemctl reload nginx

# Verify listening
sudo netstat -tulpn | grep :80
```

**Expected Output:**
```
tcp        0      0 0.0.0.0:80          0.0.0.0:*          LISTEN      <nginx_pid>/nginx
tcp6       0      0 :::80               :::*               LISTEN      <nginx_pid>/nginx
```

**Reference:** Based on deployment experience with `api.bridgebond.ai`

---

### Issue 5: Domain Mismatch (api.bridgebond.ai vs api.bridgebond.ai)

**Problem:**
- Configuration files using `.io` but actual domain is `.ai`
- SSL certificate failing due to domain mismatch

**Root Cause:**
- Documentation examples used `.io` domain
- Actual deployment used `.ai` domain

**Solution:**
1. **Update all configuration files:**
   ```bash
   # Update Nginx config
   sudo nano /etc/nginx/sites-available/api.bridgebond.ai
   # Change server_name to: api.bridgebond.ai
   
   # Update .env file
   nano ~/bridge-bond/.env
   # Change BASE_URL to: https://api.bridgebond.ai
   ```

2. **Update Certbot command:**
   ```bash
   sudo certbot --nginx -d api.bridgebond.ai
   # Use .ai, not .io
   ```

3. **Verify DNS:**
   ```bash
   dig api.bridgebond.ai +short
   # Should return server IP
   ```

**Note:** Always verify your actual domain before deployment. Replace `.io` with `.ai` (or your actual domain) in all commands and configurations.

**Reference:** Based on deployment experience with `api.bridgebond.ai`

---

### Issue 6: PM2 App Not Starting After Reboot

**Problem:**
- Application not starting automatically after server reboot
- PM2 processes lost after restart

**Root Cause:**
- PM2 startup script not properly configured
- PM2 save not executed

**Solution:**
```bash
# Generate PM2 startup script
pm2 startup systemd

# Copy and run the command it outputs (usually requires sudo)
# Example:
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u ubuntu --hp /home/ubuntu

# Save current PM2 process list
pm2 save

# Verify
pm2 list
```

**Test:**
```bash
# Reboot server
sudo reboot

# After reboot, SSH back in and check
pm2 list
# Should show your apps running
```

**Reference:** Based on deployment experience with `api.bridgebond.ai`

---

### Issue 7: MongoDB Authentication Failing

**Problem:**
- Application unable to connect to MongoDB
- Authentication errors in logs

**Root Cause:**
- MongoDB authentication not enabled
- Incorrect connection string in .env file
- Wrong password or username

**Solution:**
```bash
# Check MongoDB status
sudo systemctl status mongod

# Test connection manually
mongosh -u bridgebond -p YOUR_PASSWORD --authenticationDatabase bridgebond

# If fails, check MongoDB config
sudo nano /etc/mongod.conf
# Ensure: security: authorization: enabled

# Verify .env connection string format
cat ~/bridge-bond/.env | grep MONGODB
# Should be: mongodb://username:password@localhost:27017/database?authSource=database

# Restart MongoDB
sudo systemctl restart mongod

# Test application connection
pm2 logs backend-staging | grep -i mongo
```

**Reference:** Based on deployment experience with `api.bridgebond.ai`

---

### Issue 8: Nginx 502 Bad Gateway

**Problem:**
- Nginx returning 502 Bad Gateway error
- Application not reachable through domain

**Root Cause:**
- Node.js application not running
- Application not listening on port 2323
- Wrong upstream configuration in Nginx

**Solution:**
```bash
# Check if app is running
pm2 status

# If not running, start it
pm2 start ecosystem.config.json --only backend-staging

# Test local connection
curl http://localhost:2323

# Check Nginx error logs
sudo tail -50 /var/log/nginx/api.bridgebond.ai-error.log

# Verify Nginx upstream configuration
sudo nano /etc/nginx/sites-available/api.bridgebond.ai
# Ensure: proxy_pass http://127.0.0.1:2323; (or upstream name)

# Test and reload Nginx
sudo nginx -t
sudo systemctl reload nginx
```

**Reference:** Based on deployment experience with `api.bridgebond.ai`

---

### Issue 9: Certbot Timeout - Security Group Not Open

**Problem:**
```
Error: Timeout during connect (likely firewall problem)
Detail: 51.20.185.71: Fetching http://api.bridgebond.ai/.well-known/acme-challenge/...: 
Timeout during connect (likely firewall problem)
```

**Root Cause:**
- EC2 Security Group missing inbound rules for ports 80 and 443
- Let's Encrypt cannot reach server to verify domain ownership
- Even if DNS is correct (api.bridgebond.ai → 51.20.185.71), firewall blocks the connection
- UFW may be inactive, but Security Group is still blocking

**Solution:**

1. **Find Security Group:**
   ```bash
   # Try to get Security Group (may return empty if metadata service restricted)
   curl http://169.254.169.254/latest/meta-data/security-groups
   
   # If empty, check in AWS Console:
   # EC2 → Instances → Your Instance → Security → Security Groups
   ```

2. **In AWS Console - Add Inbound Rules:**
   - Go to **EC2 Dashboard → Instances → Select your instance**
   - Scroll to **Security → Security Groups**
   - Click the security group name
   - Go to **Inbound rules → Edit inbound rules**
   - **Add these rules** (if not present):

   | Type  | Protocol | Port Range | Source    | Description        |
   |-------|----------|------------|-----------|-------------------|
   | HTTP  | TCP      | 80         | 0.0.0.0/0 | Let's Encrypt     |
   | HTTPS | TCP      | 443        | 0.0.0.0/0 | HTTPS traffic     |

   - Click **"Save rules"**

3. **Verify UFW Status:**
   ```bash
   sudo ufw status
   
   # If active, allow Nginx
   sudo ufw allow 'Nginx Full'
   sudo ufw reload
   
   # If inactive (as in your case), UFW is not the issue
   ```

4. **Verify Nginx is Running and Listening:**
   ```bash
   # Check Nginx status
   sudo systemctl status nginx
   
   # If not running, start it
   sudo systemctl start nginx
   sudo systemctl enable nginx
   
   # Verify Nginx is listening on port 80
   sudo netstat -tulpn | grep :80
   # OR
   sudo ss -tulnp | grep :80
   
   # Should show:
   # tcp        0      0 0.0.0.0:80          0.0.0.0:*          LISTEN      <nginx_pid>/nginx
   ```

5. **Create Basic Nginx Config for Certbot:**
   ```bash
   # Create Nginx config file
   sudo nano /etc/nginx/sites-available/api.bridgebond.ai
   ```

   Paste this minimal configuration:

   ```nginx
   server {
       listen 80;
       server_name api.bridgebond.ai;
       
       root /var/www/html;
       
       # Allow Let's Encrypt verification
       location /.well-known/acme-challenge/ {
           allow all;
       }
       
       # Proxy to Node.js app
       location / {
           proxy_pass http://localhost:2323;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }
   }
   ```

   ```bash
   # Enable the site
   sudo ln -s /etc/nginx/sites-available/api.bridgebond.ai /etc/nginx/sites-enabled/
   
   # Test configuration
   sudo nginx -t
   
   # Reload Nginx
   sudo systemctl reload nginx
   ```

6. **Test from Outside:**
   ```bash
   # From your local machine (not on server)
   curl -I http://api.bridgebond.ai
   
   # Should return HTTP headers (200, 301, or 302)
   # If timeout → Security Group still not configured correctly
   # If response → Port 80 is open, proceed to next step
   ```

7. **Retry Certbot:**
   ```bash
   sudo certbot --nginx -d api.bridgebond.ai
   ```

**Expected Success Output:**
```
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/api.bridgebond.ai/fullchain.pem
Key is saved at: /etc/letsencrypt/live/api.bridgebond.ai/privkey.pem
```

**Quick Diagnostic Commands:**
```bash
# 1. Check DNS (should return server IP)
dig api.bridgebond.ai +short

# 2. Check UFW (should be inactive or allow port 80)
sudo ufw status

# 3. Check Nginx status
sudo systemctl status nginx

# 4. Check if Nginx is listening
sudo netstat -tulpn | grep :80

# 5. Test from outside (from local machine)
curl -I http://api.bridgebond.ai
```

**Reference:** Based on deployment experience with `api.bridgebond.ai` - DNS was correct (51.20.185.71), UFW was inactive, but Security Group was blocking port 80.

---

### Common Deployment Checklist

Before running Certbot or accessing your domain, verify:

- [ ] DNS A record points to server IP: `dig api.bridgebond.ai +short`
- [ ] EC2 Security Group allows port 80: HTTP (TCP 80) from 0.0.0.0/0
- [ ] EC2 Security Group allows port 443: HTTPS (TCP 443) from 0.0.0.0/0
- [ ] UFW allows ports 80 and 443: `sudo ufw status`
- [ ] Nginx is running: `sudo systemctl status nginx`
- [ ] Nginx is listening on port 80: `sudo netstat -tulpn | grep :80`
- [ ] Nginx config is valid: `sudo nginx -t`
- [ ] Can reach server from outside: `curl -I http://api.bridgebond.ai`
- [ ] PM2 app is running: `pm2 status`
- [ ] Application responds locally: `curl http://localhost:2323`
- [ ] MongoDB is running: `sudo systemctl status mongod`
- [ ] Domain matches in all configs: `.ai` not `.io` (or your actual domain)

---

### Quick Fix Commands

**If Certbot times out:**
```bash
# 1. Check Security Group in AWS Console (add ports 80, 443)
# 2. Allow UFW
sudo ufw allow 'Nginx Full'
# 3. Verify Nginx
sudo systemctl status nginx
sudo netstat -tulpn | grep :80
# 4. Test from outside
curl -I http://api.bridgebond.ai
# 5. Retry Certbot
sudo certbot --nginx -d api.bridgebond.ai
```

**If 502 Bad Gateway:**
```bash
# 1. Check PM2
pm2 status
pm2 restart backend-staging
# 2. Test local
curl http://localhost:2323
# 3. Check Nginx logs
sudo tail -50 /var/log/nginx/api.bridgebond.ai-error.log
# 4. Reload Nginx
sudo systemctl reload nginx
```

**If domain not resolving:**
```bash
# 1. Check DNS
dig api.bridgebond.ai +short
# 2. Verify in DNS provider
# 3. Wait 5-10 minutes for propagation
# 4. Check globally: https://dnschecker.org
```

---

**Note:** These issues were encountered during the actual deployment of `api.bridgebond.ai`. Always verify your specific domain and server configuration before applying these solutions.

**References:**
- Deployment experience: `api.bridgebond.ai`
- Domain: `api.bridgebond.ai` (not `.io`)
- Server IP: Verify with `curl ifconfig.me` or AWS Console

---

## 📋 Quick Reference Commands

### PM2 Commands
```bash
pm2 list                    # List all processes
pm2 start ecosystem.config.json --only backend-staging
pm2 logs backend-staging    # View logs
pm2 restart backend-staging # Restart
pm2 reload backend-staging  # Zero-downtime reload
pm2 stop backend-staging    # Stop
pm2 delete backend-staging # Delete
pm2 save                   # Save configuration
pm2 monit                  # Monitor dashboard
```

### Nginx Commands
```bash
sudo nginx -t               # Test configuration
sudo systemctl status nginx # Check status
sudo systemctl reload nginx # Reload configuration
sudo systemctl restart nginx # Restart
sudo tail -f /var/log/nginx/api.bridgebond.ai-error.log
```

### MongoDB Commands
```bash
sudo systemctl status mongod
sudo systemctl restart mongod
mongosh -u bridgebond -p PASSWORD --authenticationDatabase bridgebond
```

### SSL Commands
```bash
sudo certbot certificates   # List certificates
sudo certbot renew         # Renew manually
sudo certbot renew --dry-run # Test renewal
```

### System Commands
```bash
htop                       # Monitor resources
df -h                      # Disk usage
free -h                    # Memory usage
sudo ufw status           # Firewall status
```

---

## ✅ Deployment Checklist

- [ ] AWS EC2 instance created and running
- [ ] SSH access configured
- [ ] System updated and essential tools installed
- [ ] Git installed and repository cloned
- [ ] Node.js and npm installed
- [ ] MongoDB installed and configured with authentication
- [ ] PM2 installed and configured
- [ ] Application dependencies installed
- [ ] Environment variables configured (.env file)
- [ ] Application started with PM2
- [ ] Nginx installed and configured
- [ ] DNS A record configured (api.bridgebond.ai → server IP)
- [ ] SSL certificate obtained and configured
- [ ] Firewall (UFW) configured
- [ ] Application accessible via https://api.bridgebond.ai
- [ ] API documentation accessible at https://api.bridgebond.ai/v1/docs
- [ ] Backups configured
- [ ] Monitoring set up
- [ ] All services running correctly

---

## 🎯 Next Steps

1. **Set up monitoring** - Consider using services like PM2 Plus, New Relic, or Datadog
2. **Configure backups** - Set up automated MongoDB backups
3. **Set up CI/CD** - Automate deployments with GitHub Actions or similar
4. **Add monitoring alerts** - Set up alerts for downtime, high CPU, etc.
5. **Scale horizontally** - Add more instances behind a load balancer if needed
6. **Set up staging environment** - Deploy a separate staging instance

---

## 📚 Additional Resources

- [PM2 Documentation](https://pm2.keymetrics.io/docs/)
- [Nginx Documentation](https://nginx.org/en/docs/)
- [Let's Encrypt Documentation](https://letsencrypt.org/docs/)
- [MongoDB Documentation](https://docs.mongodb.com/)
- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)

---

**Last Updated**: 2024  
**Domain**: api.bridgebond.ai  
**Maintained by**: Bridge Bond Team

