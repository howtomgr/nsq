# nsq Installation Guide

nsq is a free and open-source message platform. NSQ provides distributed messaging platform designed for scale

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 2+ cores
  - RAM: 2GB minimum
  - Storage: 10GB for messages
  - Network: HTTP/TCP
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 4151 (default nsq port)
  - TCP on 4150, HTTP on 4171
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install nsq
sudo dnf install -y nsq

# Enable and start service
sudo systemctl enable --now nsq

# Configure firewall
sudo firewall-cmd --permanent --add-port=4151/tcp
sudo firewall-cmd --reload

# Verify installation
nsq --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install nsq
sudo apt install -y nsq

# Enable and start service
sudo systemctl enable --now nsq

# Configure firewall
sudo ufw allow 4151

# Verify installation
nsq --version
```

### Arch Linux

```bash
# Install nsq
sudo pacman -S nsq

# Enable and start service
sudo systemctl enable --now nsq

# Verify installation
nsq --version
```

### Alpine Linux

```bash
# Install nsq
apk add --no-cache nsq

# Enable and start service
rc-update add nsq default
rc-service nsq start

# Verify installation
nsq --version
```

### openSUSE/SLES

```bash
# Install nsq
sudo zypper install -y nsq

# Enable and start service
sudo systemctl enable --now nsq

# Configure firewall
sudo firewall-cmd --permanent --add-port=4151/tcp
sudo firewall-cmd --reload

# Verify installation
nsq --version
```

### macOS

```bash
# Using Homebrew
brew install nsq

# Start service
brew services start nsq

# Verify installation
nsq --version
```

### FreeBSD

```bash
# Using pkg
pkg install nsq

# Enable in rc.conf
echo 'nsq_enable="YES"' >> /etc/rc.conf

# Start service
service nsq start

# Verify installation
nsq --version
```

### Windows

```bash
# Using Chocolatey
choco install nsq

# Or using Scoop
scoop install nsq

# Verify installation
nsq --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/nsq

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
nsq --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable nsq

# Start service
sudo systemctl start nsq

# Stop service
sudo systemctl stop nsq

# Restart service
sudo systemctl restart nsq

# Check status
sudo systemctl status nsq

# View logs
sudo journalctl -u nsq -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add nsq default

# Start service
rc-service nsq start

# Stop service
rc-service nsq stop

# Restart service
rc-service nsq restart

# Check status
rc-service nsq status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'nsq_enable="YES"' >> /etc/rc.conf

# Start service
service nsq start

# Stop service
service nsq stop

# Restart service
service nsq restart

# Check status
service nsq status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start nsq
brew services stop nsq
brew services restart nsq

# Check status
brew services list | grep nsq
```

### Windows Service Manager

```powershell
# Start service
net start nsq

# Stop service
net stop nsq

# Using PowerShell
Start-Service nsq
Stop-Service nsq
Restart-Service nsq

# Check status
Get-Service nsq
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream nsq_backend {
    server 127.0.0.1:4151;
}

server {
    listen 80;
    server_name nsq.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name nsq.example.com;

    ssl_certificate /etc/ssl/certs/nsq.example.com.crt;
    ssl_certificate_key /etc/ssl/private/nsq.example.com.key;

    location / {
        proxy_pass http://nsq_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName nsq.example.com
    Redirect permanent / https://nsq.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName nsq.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/nsq.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/nsq.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:4151/
    ProxyPassReverse / http://127.0.0.1:4151/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend nsq_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/nsq.pem
    redirect scheme https if !{ ssl_fc }
    default_backend nsq_backend

backend nsq_backend
    balance roundrobin
    server nsq1 127.0.0.1:4151 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R nsq:nsq /etc/nsq
sudo chmod 750 /etc/nsq

# Configure firewall
sudo firewall-cmd --permanent --add-port=4151/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status nsq

# View logs
sudo journalctl -u nsq -f

# Monitor resource usage
top -p $(pgrep nsq)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/nsq"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/nsq-backup-$DATE.tar.gz" /etc/nsq /var/lib/nsq

echo "Backup completed: $BACKUP_DIR/nsq-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop nsq

# Restore from backup
tar -xzf /backup/nsq/nsq-backup-*.tar.gz -C /

# Start service
sudo systemctl start nsq
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u nsq -n 100
sudo tail -f /var/log/nsq/nsq.log

# Check configuration
nsq --version

# Check permissions
ls -la /etc/nsq
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 4151

# Test connectivity
telnet localhost 4151

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep nsq)

# Check disk I/O
iotop -p $(pgrep nsq)

# Check connections
ss -an | grep 4151
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  nsq:
    image: nsq:latest
    ports:
      - "4151:4151"
    volumes:
      - ./config:/etc/nsq
      - ./data:/var/lib/nsq
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update nsq

# Debian/Ubuntu
sudo apt update && sudo apt upgrade nsq

# Arch Linux
sudo pacman -Syu nsq

# Alpine Linux
apk update && apk upgrade nsq

# openSUSE
sudo zypper update nsq

# FreeBSD
pkg update && pkg upgrade nsq

# Always backup before updates
tar -czf /backup/nsq-pre-update-$(date +%Y%m%d).tar.gz /etc/nsq

# Restart after updates
sudo systemctl restart nsq
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/nsq

# Clean old logs
find /var/log/nsq -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/nsq
```

## Additional Resources

- Official Documentation: https://docs.nsq.org/
- GitHub Repository: https://github.com/nsq/nsq
- Community Forum: https://forum.nsq.org/
- Best Practices Guide: https://docs.nsq.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
