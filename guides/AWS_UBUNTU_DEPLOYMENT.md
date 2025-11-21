# AWS Ubuntu Server Setup Guide - Complete A to Z

This comprehensive guide covers everything you need to set up and deploy Bridge Bond on an AWS Ubuntu server, including system configuration, Git, Nginx, SSL, and all related components.

## Table of Contents

1. [AWS EC2 Instance Setup](#1-aws-ec2-instance-setup)
2. [Initial Server Configuration](#2-initial-server-configuration)
3. [System Updates and Essential Tools](#3-system-updates-and-essential-tools)
4. [Git Installation and Configuration](#4-git-installation-and-configuration)
5. [Node.js and npm Installation](#5-nodejs-and-npm-installation)
6. [MongoDB Installation and Setup](#6-mongodb-installation-and-setup)
7. [Nginx Installation and Configuration](#7-nginx-installation-and-configuration)
8. [SSL Certificate Setup (Let's Encrypt)](#8-ssl-certificate-setup-lets-encrypt)
9. [Firewall Configuration (UFW)](#9-firewall-configuration-ufw)
10. [PM2 Process Manager Setup](#10-pm2-process-manager-setup)
11. [Application Deployment](#11-application-deployment)
12. [Domain and DNS Configuration](#12-domain-and-dns-configuration)
13. [Security Hardening](#13-security-hardening)
14. [Monitoring and Logging](#14-monitoring-and-logging)
15. [Backup and Maintenance](#15-backup-and-maintenance)
16. [Troubleshooting](#16-troubleshooting)

---

## 1. AWS EC2 Instance Setup

### 1.1 Create EC2 Instance

1. **Login to AWS Console**
   - Go to [AWS Console](https://console.aws.amazon.com)
   - Navigate to EC2 Dashboard

2. **Launch Instance**
   - Click "Launch Instance"
   - Name: `bridge-bond-production` (or your preferred name)

3. **Choose AMI**
   - Select **Ubuntu Server 22.04 LTS** (or latest LTS)
   - Architecture: 64-bit (x86)

4. **Instance Type**
   - For production: `t3.medium` or `t3.large` (2-4 vCPU, 4-8 GB RAM)
   - For development: `t3.micro` or `t3.small` (1-2 vCPU, 1-2 GB RAM)

5. **Key Pair**
   - Create new key pair or use existing
   - Download the `.pem` file securely
   - **Important**: Save this file - you'll need it to SSH

6. **Network Settings**
   - VPC: Default or your custom VPC
   - Subnet: Public subnet
   - Auto-assign Public IP: Enable
   - Security Group: Create new or use existing
     - **Inbound Rules**:
       - SSH (22) - Your IP only
       - HTTP (80) - 0.0.0.0/0
       - HTTPS (443) - 0.0.0.0/0
     - **Outbound Rules**: All traffic

7. **Storage**
   - Root volume: 20-30 GB (gp3 SSD)
   - Add additional volumes if needed

8. **Launch Instance**
   - Review and launch
   - Wait for instance to be in "Running" state

### 1.2 Get Instance Details

- Note the **Public IPv4 address** (e.g., `54.123.45.67`)
- Note the **Elastic IP** (if assigned)

---

## 2. Initial Server Configuration

### 2.1 Connect to Server via SSH

```bash
# On your local machine
chmod 400 /path/to/your-key.pem
ssh -i /path/to/your-key.pem ubuntu@YOUR_SERVER_IP

# Example:
ssh -i ~/Downloads/bridge-bond-key.pem ubuntu@54.123.45.67
```

### 2.2 Update System

```bash
# Update package list
sudo apt update

# Upgrade all packages
sudo apt upgrade -y

# Install essential build tools
sudo apt install -y build-essential curl wget git software-properties-common
```

### 2.3 Create Non-Root User (Optional but Recommended)

```bash
# Create new user
sudo adduser deploy

# Add to sudo group
sudo usermod -aG sudo deploy

# Switch to new user
su - deploy
```

### 2.4 Configure Timezone

```bash
# Set timezone
sudo timedatectl set-timezone UTC

# Verify
timedatectl
```

### 2.5 Configure Hostname

```bash
# Set hostname
sudo hostnamectl set-hostname bridge-bond-server

# Add to /etc/hosts
echo "127.0.0.1 bridge-bond-server" | sudo tee -a /etc/hosts
```

---

## 3. System Updates and Essential Tools

### 3.1 Install Essential Packages

```bash
# Install essential tools
sudo apt install -y \
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
    unattended-upgrades \
    apt-transport-https \
    ca-certificates \
    gnupg \
    lsb-release
```

### 3.2 Configure Automatic Security Updates

```bash
# Enable automatic security updates
sudo dpkg-reconfigure -plow unattended-upgrades

# Verify
sudo systemctl status unattended-upgrades
```

### 3.3 Install Fail2Ban (Brute Force Protection)

```bash
# Configure fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

# Check status
sudo fail2ban-client status
```

---

## 4. Git Installation and Configuration

### 4.1 Install Git

```bash
# Git is usually pre-installed, but verify
git --version

# If not installed
sudo apt install -y git

# Configure Git
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set default branch name
git config --global init.defaultBranch main

# Configure line endings (for cross-platform)
git config --global core.autocrlf input
```

### 4.2 Generate SSH Key for Git (Optional)

```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"

# Start SSH agent
eval "$(ssh-agent -s)"

# Add key to agent
ssh-add ~/.ssh/id_ed25519

# Display public key (add to GitHub/GitLab)
cat ~/.ssh/id_ed25519.pub
```

### 4.3 Clone Repository

```bash
# Navigate to home directory
cd ~

# Clone repository (HTTPS)
git clone https://github.com/ITSOLSTUDIOLTD/bridgebond-backend.git bridge-bond

# Or using SSH (if configured)
# git clone git@github.com:ITSOLSTUDIOLTD/bridgebond-backend.git bridge-bond

# Navigate to project
cd bridge-bond
```

---

## 5. Node.js and npm Installation

### 5.1 Install Node.js (Using NodeSource)

```bash
# Install Node.js 20.x LTS (or latest LTS)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Verify installation
node --version  # Should be v20.x.x
npm --version   # Should be 10.x.x
```

### 5.2 Install Yarn (Optional)

```bash
# Install Yarn
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt update
sudo apt install -y yarn

# Verify
yarn --version
```

### 5.3 Install PM2 Globally

```bash
# Install PM2
sudo npm install -g pm2

# Verify
pm2 --version

# Setup PM2 startup script
pm2 startup systemd
# Follow the instructions shown
```

---

## 6. MongoDB Installation and Setup

### 6.1 Install MongoDB

```bash
# Import MongoDB GPG key
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor

# Add MongoDB repository
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

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

### 6.2 Configure MongoDB

```bash
# Edit MongoDB config
sudo nano /etc/mongod.conf

# Important settings:
# - bindIp: 127.0.0.1 (for local only) or 0.0.0.0 (if needed externally)
# - port: 27017
# - Enable authentication (see security section)

# Restart MongoDB
sudo systemctl restart mongod
```

### 6.3 Create MongoDB Admin User

```bash
# Connect to MongoDB
mongosh

# In MongoDB shell:
use admin
db.createUser({
  user: "admin",
  pwd: "your-strong-password-here",
  roles: [ { role: "userAdminAnyDatabase", db: "admin" }, "readWriteAnyDatabase" ]
})

# Create application database user
use bridge-bond
db.createUser({
  user: "bridgebond",
  pwd: "your-app-password-here",
  roles: [ { role: "readWrite", db: "bridge-bond" } ]
})

# Exit
exit
```

### 6.4 Enable MongoDB Authentication

```bash
# Edit config
sudo nano /etc/mongod.conf

# Add security section:
security:
  authorization: enabled

# Restart MongoDB
sudo systemctl restart mongod

# Test connection
mongosh -u admin -p your-strong-password-here --authenticationDatabase admin
```

---

## 7. Nginx Installation and Configuration

### 7.1 Install Nginx

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

### 7.2 Configure Nginx for Bridge Bond

```bash
# Create Nginx configuration
sudo nano /etc/nginx/sites-available/bridge-bond
```

Add the following configuration:

```nginx
# Upstream configuration for Node.js app
upstream bridge_bond {
    server 127.0.0.1:2323;
    keepalive 64;
}

# HTTP to HTTPS redirect
server {
    listen 80;
    listen [::]:80;
    server_name yourdomain.com www.yourdomain.com;

    # For Let's Encrypt verification
    location /.well-known/acme-challenge/ {
        root /var/www/html;
    }

    # Redirect all HTTP to HTTPS
    location / {
        return 301 https://$server_name$request_uri;
    }
}

# HTTPS server
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name yourdomain.com www.yourdomain.com;

    # SSL Configuration (will be updated after Let's Encrypt)
    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;
    
    # SSL Security Settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    ssl_session_tickets off;

    # Security Headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;

    # Logging
    access_log /var/log/nginx/bridge-bond-access.log;
    error_log /var/log/nginx/bridge-bond-error.log;

    # Client body size (for file uploads)
    client_max_body_size 10M;

    # Proxy settings
    location / {
        proxy_pass http://bridge_bond;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # Health check endpoint
    location /health {
        access_log off;
        proxy_pass http://bridge_bond/health;
    }
}
```

### 7.3 Enable Site and Test Configuration

```bash
# Create symbolic link
sudo ln -s /etc/nginx/sites-available/bridge-bond /etc/nginx/sites-enabled/

# Remove default site (optional)
sudo rm /etc/nginx/sites-enabled/default

# Test Nginx configuration
sudo nginx -t

# If test passes, reload Nginx
sudo systemctl reload nginx
```

---

## 8. SSL Certificate Setup (Let's Encrypt)

### 8.1 Install Certbot

```bash
# Install Certbot
sudo apt install -y certbot python3-certbot-nginx
```

### 8.2 Obtain SSL Certificate

```bash
# Make sure your domain points to this server's IP first!

# Obtain certificate
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Follow the prompts:
# - Enter email address
# - Agree to terms
# - Choose whether to redirect HTTP to HTTPS (recommended: Yes)
```

### 8.3 Auto-Renewal Setup

```bash
# Test renewal
sudo certbot renew --dry-run

# Certbot automatically sets up a cron job, but verify:
sudo systemctl status certbot.timer
```

### 8.4 Update Nginx Config (if needed)

After Certbot runs, it automatically updates your Nginx config. Verify:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## 9. Firewall Configuration (UFW)

### 9.1 Configure UFW

```bash
# Check UFW status
sudo ufw status

# Set default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (IMPORTANT: Do this first!)
sudo ufw allow 22/tcp

# Allow HTTP and HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Enable UFW
sudo ufw enable

# Check status
sudo ufw status verbose
```

### 9.2 Additional Security Rules

```bash
# Allow specific IP for SSH (more secure)
sudo ufw delete allow 22/tcp
sudo ufw allow from YOUR_IP_ADDRESS to any port 22

# Check logs
sudo ufw status numbered
```

---

## 10. PM2 Process Manager Setup

### 10.1 Configure PM2

```bash
# Navigate to project directory
cd ~/bridge-bond

# Install project dependencies
npm install
# or
yarn install

# Create PM2 ecosystem config (if not exists)
# The project already has ecosystem.config.json
```

### 10.2 Start Application with PM2

```bash
# Start application
pm2 start ecosystem.config.json

# Or start directly
pm2 start src/index.js --name bridge-bond

# Save PM2 configuration
pm2 save

# Check status
pm2 status

# View logs
pm2 logs bridge-bond

# Monitor
pm2 monit
```

### 10.3 PM2 Useful Commands

```bash
# Restart application
pm2 restart bridge-bond

# Stop application
pm2 stop bridge-bond

# Delete from PM2
pm2 delete bridge-bond

# View logs
pm2 logs bridge-bond --lines 100

# Clear logs
pm2 flush

# Reload (zero-downtime)
pm2 reload bridge-bond
```

---

## 11. Application Deployment

### 11.1 Environment Variables Setup

```bash
# Create .env file
cd ~/bridge-bond
nano .env
```

Add your environment variables:

```env
# Environment
NODE_ENV=production

# Server
PORT=2323
HOST=0.0.0.0
BASE_URL=https://yourdomain.com

# Database
MONGODB_URL_STAGING=mongodb://bridgebond:password@localhost:27017/bridge-bond?authSource=bridge-bond
MONGODB_URL_LIVE=mongodb://bridgebond:password@localhost:27017/bridge-bond?authSource=bridge-bond
IS_LIVE=true

# JWT Configuration
JWT_SECRET=your-very-strong-secret-key-minimum-32-characters-long
JWT_ACCESS_EXPIRATION_MINUTES=2880
JWT_REFRESH_EXPIRATION_DAYS=5
JWT_RESET_PASSWORD_EXPIRATION_MINUTES=10
JWT_VERIFY_EMAIL_EXPIRATION_MINUTES=10

# Email Configuration
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_SERVICE=Gmail
SMTP_USERNAME=your-email@gmail.com
SMTP_PASSWORD=your-app-password
EMAIL_FROM=noreply@yourdomain.com

# Cloudinary (if using)
CLOUDINARY_CLOUD_NAME=your-cloud-name
CLOUDINARY_API_KEY=your-api-key
CLOUDINARY_API_SECRET=your-api-secret

# OneSignal (if using)
ONESIGNAL_APP_ID=your-app-id
ONESIGNAL_REST_API_KEY=your-rest-api-key

# Other services...
```

### 11.2 Secure Environment File

```bash
# Set proper permissions
chmod 600 .env

# Verify
ls -la .env
```

### 11.3 Initialize Database

```bash
# Create super admin
npm run create-superadmin

# Or seed database (development only)
# npm run seed
```

### 11.4 Start Application

```bash
# Start with PM2
pm2 start ecosystem.config.json

# Check status
pm2 status
pm2 logs bridge-bond
```

### 11.5 Verify Application

```bash
# Check if app is running
curl http://localhost:2323/health

# Check from outside (if firewall allows)
curl https://yourdomain.com/health
```

---

## 12. Domain and DNS Configuration

### 12.1 DNS Records Setup

In your domain registrar's DNS settings, add:

```
Type    Name    Value              TTL
A       @       YOUR_SERVER_IP     3600
A       www     YOUR_SERVER_IP      3600
```

### 12.2 Verify DNS Propagation

```bash
# Check DNS records
dig yourdomain.com
nslookup yourdomain.com

# Or use online tools:
# - https://dnschecker.org
# - https://www.whatsmydns.net
```

### 12.3 Test Domain Access

```bash
# Test HTTP
curl -I http://yourdomain.com

# Test HTTPS
curl -I https://yourdomain.com

# Test API
curl https://yourdomain.com/v1/docs
```

---

## 13. Security Hardening

### 13.1 SSH Hardening

```bash
# Edit SSH config
sudo nano /etc/ssh/sshd_config
```

Important settings:

```
Port 2222                    # Change default port (optional)
PermitRootLogin no           # Disable root login
PasswordAuthentication no    # Disable password auth (use keys only)
PubkeyAuthentication yes     # Enable key auth
X11Forwarding no            # Disable X11
MaxAuthTries 3              # Limit auth attempts
```

```bash
# Restart SSH
sudo systemctl restart sshd

# Test connection before closing current session!
```

### 13.2 MongoDB Security

```bash
# Edit MongoDB config
sudo nano /etc/mongod.conf
```

Ensure:

```yaml
net:
  bindIp: 127.0.0.1  # Only localhost
  port: 27017

security:
  authorization: enabled
```

### 13.3 Node.js Security

```bash
# Install security updates
npm audit
npm audit fix

# Use strong JWT secrets
# Use environment variables (never commit .env)
# Enable rate limiting
# Use HTTPS only
```

### 13.4 Regular Security Updates

```bash
# Update system regularly
sudo apt update
sudo apt upgrade -y

# Check for security updates
sudo unattended-upgrade --dry-run
```

---

## 14. Monitoring and Logging

### 14.1 PM2 Monitoring

```bash
# Install PM2 monitoring module
pm2 install pm2-logrotate

# Configure log rotation
pm2 set pm2-logrotate:max_size 10M
pm2 set pm2-logrotate:retain 7
pm2 set pm2-logrotate:compress true
```

### 14.2 Application Logs

```bash
# View PM2 logs
pm2 logs bridge-bond

# View Nginx logs
sudo tail -f /var/log/nginx/bridge-bond-access.log
sudo tail -f /var/log/nginx/bridge-bond-error.log

# View MongoDB logs
sudo tail -f /var/log/mongodb/mongod.log
```

### 14.3 System Monitoring

```bash
# Install monitoring tools
sudo apt install -y htop iotop nethogs

# Monitor system resources
htop

# Check disk usage
df -h

# Check memory
free -h

# Check running processes
ps aux | grep node
```

### 14.4 Set Up Log Rotation

```bash
# Configure logrotate for application logs
sudo nano /etc/logrotate.d/bridge-bond
```

Add:

```
/home/ubuntu/bridge-bond/logs/*.log {
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

---

## 15. Backup and Maintenance

### 15.1 MongoDB Backup

```bash
# Create backup script
nano ~/backup-mongodb.sh
```

Add:

```bash
#!/bin/bash
BACKUP_DIR="/home/ubuntu/backups/mongodb"
DATE=$(date +%Y%m%d_%H%M%S)
mkdir -p $BACKUP_DIR

# Backup database
mongodump --uri="mongodb://bridgebond:password@localhost:27017/bridge-bond?authSource=bridge-bond" \
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
```

### 15.2 Automated Backups (Cron)

```bash
# Edit crontab
crontab -e

# Add daily backup at 2 AM
0 2 * * * /home/ubuntu/backup-mongodb.sh >> /home/ubuntu/backup.log 2>&1
```

### 15.3 MongoDB Restore

```bash
# Restore from backup
mongorestore --uri="mongodb://bridgebond:password@localhost:27017/bridge-bond?authSource=bridge-bond" \
  --archive=/path/to/backup.tar.gz --gzip
```

### 15.4 Application Files Backup

```bash
# Backup .env and important files
tar -czf ~/backups/app-config-$(date +%Y%m%d).tar.gz \
  ~/bridge-bond/.env \
  ~/bridge-bond/ecosystem.config.json
```

---

## 16. Troubleshooting

### 16.1 Application Not Starting

```bash
# Check PM2 logs
pm2 logs bridge-bond --lines 50

# Check if port is in use
sudo netstat -tulpn | grep 2323

# Check environment variables
pm2 env 0

# Restart application
pm2 restart bridge-bond
```

### 16.2 Nginx Issues

```bash
# Test Nginx configuration
sudo nginx -t

# Check Nginx status
sudo systemctl status nginx

# View error logs
sudo tail -f /var/log/nginx/error.log
sudo tail -f /var/log/nginx/bridge-bond-error.log

# Reload Nginx
sudo systemctl reload nginx
```

### 16.3 MongoDB Connection Issues

```bash
# Check MongoDB status
sudo systemctl status mongod

# Check MongoDB logs
sudo tail -f /var/log/mongodb/mongod.log

# Test connection
mongosh -u bridgebond -p password --authenticationDatabase bridge-bond

# Restart MongoDB
sudo systemctl restart mongod
```

### 16.4 SSL Certificate Issues

```bash
# Check certificate status
sudo certbot certificates

# Renew certificate manually
sudo certbot renew

# Test renewal
sudo certbot renew --dry-run
```

### 16.5 Firewall Issues

```bash
# Check UFW status
sudo ufw status verbose

# Check if port is blocked
sudo ufw status | grep 2323

# Temporarily disable firewall (for testing only)
sudo ufw disable
# Remember to enable it back!
```

### 16.6 Disk Space Issues

```bash
# Check disk usage
df -h

# Find large files
sudo du -h --max-depth=1 / | sort -hr

# Clean old logs
pm2 flush
sudo journalctl --vacuum-time=7d
```

### 16.7 Memory Issues

```bash
# Check memory usage
free -h
htop

# Check Node.js memory
pm2 monit

# Restart if needed
pm2 restart bridge-bond --update-env
```

---

## Quick Reference Commands

### System
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Reboot server
sudo reboot

# Check system resources
htop
df -h
free -h
```

### Application
```bash
# PM2 commands
pm2 status
pm2 logs bridge-bond
pm2 restart bridge-bond
pm2 reload bridge-bond
pm2 stop bridge-bond
pm2 delete bridge-bond

# Application logs
pm2 logs bridge-bond --lines 100
```

### Nginx
```bash
# Nginx commands
sudo systemctl status nginx
sudo nginx -t
sudo systemctl reload nginx
sudo systemctl restart nginx
sudo tail -f /var/log/nginx/bridge-bond-error.log
```

### MongoDB
```bash
# MongoDB commands
sudo systemctl status mongod
sudo systemctl restart mongod
mongosh -u bridgebond -p password --authenticationDatabase bridge-bond
```

### SSL
```bash
# SSL commands
sudo certbot certificates
sudo certbot renew
sudo certbot renew --dry-run
```

### Firewall
```bash
# Firewall commands
sudo ufw status
sudo ufw allow 80/tcp
sudo ufw reload
```

---

## Deployment Checklist

- [ ] AWS EC2 instance created and running
- [ ] SSH access configured
- [ ] System updated and essential tools installed
- [ ] Git installed and repository cloned
- [ ] Node.js and npm installed
- [ ] MongoDB installed and configured with authentication
- [ ] Nginx installed and configured
- [ ] SSL certificate obtained and configured
- [ ] Firewall (UFW) configured
- [ ] PM2 installed and configured
- [ ] Environment variables set up
- [ ] Application started and running
- [ ] Domain DNS configured
- [ ] SSL certificate auto-renewal configured
- [ ] Backups configured
- [ ] Monitoring set up
- [ ] Security hardening completed
- [ ] Application tested and verified

---

## Additional Resources

- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [Ubuntu Server Guide](https://ubuntu.com/server/docs)
- [Nginx Documentation](https://nginx.org/en/docs/)
- [Let's Encrypt Documentation](https://letsencrypt.org/docs/)
- [PM2 Documentation](https://pm2.keymetrics.io/docs/)
- [MongoDB Documentation](https://docs.mongodb.com/)

---

## Support

For issues or questions:
1. Check the troubleshooting section
2. Review application logs
3. Check system resources
4. Verify all services are running
5. Review security group and firewall rules

---

**Last Updated**: 2024
**Maintained by**: Bridge Bond Team

