# manticore Installation Guide

manticore is a free and open-source database for search. Manticore provides fast database for search with SQL support

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
  - CPU: 1 core minimum
  - RAM: 1GB minimum
  - Storage: 10GB for indices
  - Network: MySQL protocol
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 9306 (default manticore port)
  - HTTP on 9308
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

# Install manticore
sudo dnf install -y manticore

# Enable and start service
sudo systemctl enable --now manticore

# Configure firewall
sudo firewall-cmd --permanent --add-port=9306/tcp
sudo firewall-cmd --reload

# Verify installation
manticore --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install manticore
sudo apt install -y manticore

# Enable and start service
sudo systemctl enable --now manticore

# Configure firewall
sudo ufw allow 9306

# Verify installation
manticore --version
```

### Arch Linux

```bash
# Install manticore
sudo pacman -S manticore

# Enable and start service
sudo systemctl enable --now manticore

# Verify installation
manticore --version
```

### Alpine Linux

```bash
# Install manticore
apk add --no-cache manticore

# Enable and start service
rc-update add manticore default
rc-service manticore start

# Verify installation
manticore --version
```

### openSUSE/SLES

```bash
# Install manticore
sudo zypper install -y manticore

# Enable and start service
sudo systemctl enable --now manticore

# Configure firewall
sudo firewall-cmd --permanent --add-port=9306/tcp
sudo firewall-cmd --reload

# Verify installation
manticore --version
```

### macOS

```bash
# Using Homebrew
brew install manticore

# Start service
brew services start manticore

# Verify installation
manticore --version
```

### FreeBSD

```bash
# Using pkg
pkg install manticore

# Enable in rc.conf
echo 'manticore_enable="YES"' >> /etc/rc.conf

# Start service
service manticore start

# Verify installation
manticore --version
```

### Windows

```bash
# Using Chocolatey
choco install manticore

# Or using Scoop
scoop install manticore

# Verify installation
manticore --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/manticore

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
manticore --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable manticore

# Start service
sudo systemctl start manticore

# Stop service
sudo systemctl stop manticore

# Restart service
sudo systemctl restart manticore

# Check status
sudo systemctl status manticore

# View logs
sudo journalctl -u manticore -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add manticore default

# Start service
rc-service manticore start

# Stop service
rc-service manticore stop

# Restart service
rc-service manticore restart

# Check status
rc-service manticore status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'manticore_enable="YES"' >> /etc/rc.conf

# Start service
service manticore start

# Stop service
service manticore stop

# Restart service
service manticore restart

# Check status
service manticore status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start manticore
brew services stop manticore
brew services restart manticore

# Check status
brew services list | grep manticore
```

### Windows Service Manager

```powershell
# Start service
net start manticore

# Stop service
net stop manticore

# Using PowerShell
Start-Service manticore
Stop-Service manticore
Restart-Service manticore

# Check status
Get-Service manticore
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream manticore_backend {
    server 127.0.0.1:9306;
}

server {
    listen 80;
    server_name manticore.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name manticore.example.com;

    ssl_certificate /etc/ssl/certs/manticore.example.com.crt;
    ssl_certificate_key /etc/ssl/private/manticore.example.com.key;

    location / {
        proxy_pass http://manticore_backend;
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
    ServerName manticore.example.com
    Redirect permanent / https://manticore.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName manticore.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/manticore.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/manticore.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9306/
    ProxyPassReverse / http://127.0.0.1:9306/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend manticore_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/manticore.pem
    redirect scheme https if !{ ssl_fc }
    default_backend manticore_backend

backend manticore_backend
    balance roundrobin
    server manticore1 127.0.0.1:9306 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R manticore:manticore /etc/manticore
sudo chmod 750 /etc/manticore

# Configure firewall
sudo firewall-cmd --permanent --add-port=9306/tcp
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
sudo systemctl status manticore

# View logs
sudo journalctl -u manticore -f

# Monitor resource usage
top -p $(pgrep manticore)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/manticore"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/manticore-backup-$DATE.tar.gz" /etc/manticore /var/lib/manticore

echo "Backup completed: $BACKUP_DIR/manticore-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop manticore

# Restore from backup
tar -xzf /backup/manticore/manticore-backup-*.tar.gz -C /

# Start service
sudo systemctl start manticore
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u manticore -n 100
sudo tail -f /var/log/manticore/manticore.log

# Check configuration
manticore --version

# Check permissions
ls -la /etc/manticore
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9306

# Test connectivity
telnet localhost 9306

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep manticore)

# Check disk I/O
iotop -p $(pgrep manticore)

# Check connections
ss -an | grep 9306
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  manticore:
    image: manticore:latest
    ports:
      - "9306:9306"
    volumes:
      - ./config:/etc/manticore
      - ./data:/var/lib/manticore
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update manticore

# Debian/Ubuntu
sudo apt update && sudo apt upgrade manticore

# Arch Linux
sudo pacman -Syu manticore

# Alpine Linux
apk update && apk upgrade manticore

# openSUSE
sudo zypper update manticore

# FreeBSD
pkg update && pkg upgrade manticore

# Always backup before updates
tar -czf /backup/manticore-pre-update-$(date +%Y%m%d).tar.gz /etc/manticore

# Restart after updates
sudo systemctl restart manticore
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/manticore

# Clean old logs
find /var/log/manticore -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/manticore
```

## Additional Resources

- Official Documentation: https://docs.manticore.org/
- GitHub Repository: https://github.com/manticore/manticore
- Community Forum: https://forum.manticore.org/
- Best Practices Guide: https://docs.manticore.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
