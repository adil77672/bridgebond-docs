# PM2 Deployment Guide - Quick Reference

This guide provides quick commands for deploying and managing your Bridge Bond application with PM2.

## 📋 PM2 Configuration

Your `ecosystem.config.json` includes two environments:

- **`backend-staging`** - Staging environment (single instance, port 2323)
- **`backend-prod`** - Production environment (2 cluster instances, port 2323)

## 🚀 Quick Start Commands

### Start Applications

```bash
# Start all apps from ecosystem.config.json
pm2 start ecosystem.config.json

# Start only staging
pm2 start ecosystem.config.json --only backend-staging

# Start only production
pm2 start ecosystem.config.json --only backend-prod
```

### Check Status

```bash
# List all PM2 processes
pm2 list

# Detailed status
pm2 status

# Show specific app info
pm2 show backend-staging
pm2 show backend-prod
```

### View Logs

```bash
# View all logs
pm2 logs

# View staging logs
pm2 logs backend-staging

# View production logs
pm2 logs backend-prod

# View last 50 lines
pm2 logs backend-staging --lines 50

# View only errors
pm2 logs backend-staging --err

# View only output
pm2 logs backend-staging --out

# Follow logs in real-time
pm2 logs backend-staging --lines 0
```

### Restart/Reload

```bash
# Restart specific app
pm2 restart backend-staging

# Restart all apps
pm2 restart all

# Zero-downtime reload (cluster mode only)
pm2 reload backend-prod

# Reload all
pm2 reload all
```

### Stop/Delete

```bash
# Stop app (keeps in PM2 list)
pm2 stop backend-staging

# Stop all
pm2 stop all

# Delete app from PM2
pm2 delete backend-staging

# Delete all
pm2 delete all
```

## 🔧 Management Commands

### Save Configuration

```bash
# Save current PM2 process list
pm2 save

# This ensures apps restart on server reboot
```

### Startup Script

```bash
# Generate startup script (run once)
pm2 startup

# Follow the command it outputs (usually requires sudo)
# Example: sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u ubuntu --hp /home/ubuntu

# Then save the current process list
pm2 save
```

### Monitoring

```bash
# Real-time monitoring dashboard
pm2 monit

# Show process info
pm2 info backend-staging

# Show process tree
pm2 prettylist
```

### Memory & CPU

```bash
# Show memory usage
pm2 list

# Restart if memory exceeds limit (configured in ecosystem.config.json)
# Max memory restart is set to 1G per app
```

## 📊 Log Management

### Log Files

Logs are automatically saved to:
- Staging: `./logs/staging-err.log` and `./logs/staging-out.log`
- Production: `./logs/prod-err.log` and `./logs/prod-out.log`

### Log Rotation

```bash
# Install PM2 log rotation module
pm2 install pm2-logrotate

# Configure log rotation
pm2 set pm2-logrotate:max_size 10M
pm2 set pm2-logrotate:retain 7
pm2 set pm2-logrotate:compress true
pm2 set pm2-logrotate:dateFormat YYYY-MM-DD_HH-mm-ss
```

### Clear Logs

```bash
# Clear all logs
pm2 flush

# Clear specific app logs
pm2 flush backend-staging
```

## 🔍 Troubleshooting

### App Won't Start

```bash
# Check logs for errors
pm2 logs backend-staging --err --lines 100

# Check if port is already in use
sudo netstat -tulpn | grep 2323
# or
sudo lsof -i :2323

# Check environment variables
pm2 env 0  # Replace 0 with app ID from pm2 list
```

### App Crashes

```bash
# Check error logs
pm2 logs backend-staging --err

# Check system resources
htop
free -h
df -h

# Restart with more memory limit
# Edit ecosystem.config.json max_memory_restart value
pm2 restart ecosystem.config.json
```

### Port Already in Use

```bash
# Find process using port 2323
sudo lsof -i :2323

# Kill the process (replace PID with actual process ID)
kill -9 PID

# Or change port in ecosystem.config.json and restart
```

## 🌐 Accessing Your Application

### Local Access (on server)

```bash
# Test if app is running
curl http://localhost:2323

# Test health endpoint (if available)
curl http://localhost:2323/health

# Test API docs
curl http://localhost:2323/v1/docs
```

### External Access

1. **Get your server's public IP:**
   ```bash
   curl ifconfig.me
   ```

2. **Access via browser:**
   ```
   http://YOUR_SERVER_IP:2323
   ```

3. **With Nginx reverse proxy (recommended):**
   ```
   https://api.bridgebond.ai
   ```

### AWS Security Group

Make sure port 2323 is open in your AWS EC2 Security Group:

1. Go to AWS Console → EC2 → Security Groups
2. Select your instance's security group
3. Edit Inbound Rules
4. Add rule:
   - Type: Custom TCP
   - Port: 2323
   - Source: 0.0.0.0/0 (or your specific IP for security)

## 🔄 Deployment Workflow

### Initial Deployment

```bash
# 1. Navigate to project
cd ~/bridge-bond

# 2. Pull latest code
git pull origin main

# 3. Install dependencies
npm install

# 4. Ensure .env is configured
nano .env

# 5. Start with PM2
pm2 start ecosystem.config.json --only backend-staging

# 6. Check status
pm2 status

# 7. View logs
pm2 logs backend-staging

# 8. Save configuration
pm2 save
```

### Update Deployment

```bash
# 1. Pull latest code
git pull origin main

# 2. Install new dependencies
npm install

# 3. Restart app (zero-downtime for cluster mode)
pm2 reload backend-staging

# Or for fork mode:
pm2 restart backend-staging

# 4. Check logs
pm2 logs backend-staging --lines 50
```

## 📝 Environment-Specific Commands

### Staging Environment

```bash
# Start staging
pm2 start ecosystem.config.json --only backend-staging

# View staging logs
pm2 logs backend-staging

# Restart staging
pm2 restart backend-staging
```

### Production Environment

```bash
# Start production (2 cluster instances)
pm2 start ecosystem.config.json --only backend-prod

# View production logs
pm2 logs backend-prod

# Zero-downtime reload (cluster mode)
pm2 reload backend-prod

# Scale up/down instances
pm2 scale backend-prod 4  # Scale to 4 instances
pm2 scale backend-prod 2  # Scale back to 2
```

## 🎯 Quick Reference Table

| Command | Description |
|---------|-------------|
| `pm2 list` | Show all processes |
| `pm2 start ecosystem.config.json` | Start all apps |
| `pm2 start ecosystem.config.json --only backend-staging` | Start specific app |
| `pm2 logs backend-staging` | View logs |
| `pm2 restart backend-staging` | Restart app |
| `pm2 reload backend-prod` | Zero-downtime reload |
| `pm2 stop backend-staging` | Stop app |
| `pm2 delete backend-staging` | Remove from PM2 |
| `pm2 save` | Save process list |
| `pm2 monit` | Monitor dashboard |
| `pm2 flush` | Clear logs |

## 🔐 Security Notes

1. **Never commit `.env` file** - Contains sensitive credentials
2. **Use strong JWT secrets** - Minimum 32 characters
3. **Restrict port access** - Only open necessary ports in security groups
4. **Use Nginx reverse proxy** - Don't expose Node.js directly to internet
5. **Enable SSL/HTTPS** - Use Let's Encrypt certificates
6. **Regular updates** - Keep system and dependencies updated

## 📚 Additional Resources

- [PM2 Documentation](https://pm2.keymetrics.io/docs/)
- [AWS Ubuntu Deployment Guide](./AWS_UBUNTU_DEPLOYMENT.md)
- [Nginx Configuration Guide](./AWS_UBUNTU_DEPLOYMENT.md#7-nginx-installation-and-configuration)

---

**Last Updated**: 2024

