# faktory Installation Guide

faktory is a free and open-source job server. Faktory provides language-agnostic persistent background job server

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
  - RAM: 512MB minimum
  - Storage: 5GB for jobs
  - Network: Faktory protocol
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 7419 (default faktory port)
  - Web UI on 7420
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

# Install faktory
sudo dnf install -y faktory

# Enable and start service
sudo systemctl enable --now faktory

# Configure firewall
sudo firewall-cmd --permanent --add-port=7419/tcp
sudo firewall-cmd --reload

# Verify installation
faktory --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install faktory
sudo apt install -y faktory

# Enable and start service
sudo systemctl enable --now faktory

# Configure firewall
sudo ufw allow 7419

# Verify installation
faktory --version
```

### Arch Linux

```bash
# Install faktory
sudo pacman -S faktory

# Enable and start service
sudo systemctl enable --now faktory

# Verify installation
faktory --version
```

### Alpine Linux

```bash
# Install faktory
apk add --no-cache faktory

# Enable and start service
rc-update add faktory default
rc-service faktory start

# Verify installation
faktory --version
```

### openSUSE/SLES

```bash
# Install faktory
sudo zypper install -y faktory

# Enable and start service
sudo systemctl enable --now faktory

# Configure firewall
sudo firewall-cmd --permanent --add-port=7419/tcp
sudo firewall-cmd --reload

# Verify installation
faktory --version
```

### macOS

```bash
# Using Homebrew
brew install faktory

# Start service
brew services start faktory

# Verify installation
faktory --version
```

### FreeBSD

```bash
# Using pkg
pkg install faktory

# Enable in rc.conf
echo 'faktory_enable="YES"' >> /etc/rc.conf

# Start service
service faktory start

# Verify installation
faktory --version
```

### Windows

```bash
# Using Chocolatey
choco install faktory

# Or using Scoop
scoop install faktory

# Verify installation
faktory --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/faktory

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
faktory --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable faktory

# Start service
sudo systemctl start faktory

# Stop service
sudo systemctl stop faktory

# Restart service
sudo systemctl restart faktory

# Check status
sudo systemctl status faktory

# View logs
sudo journalctl -u faktory -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add faktory default

# Start service
rc-service faktory start

# Stop service
rc-service faktory stop

# Restart service
rc-service faktory restart

# Check status
rc-service faktory status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'faktory_enable="YES"' >> /etc/rc.conf

# Start service
service faktory start

# Stop service
service faktory stop

# Restart service
service faktory restart

# Check status
service faktory status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start faktory
brew services stop faktory
brew services restart faktory

# Check status
brew services list | grep faktory
```

### Windows Service Manager

```powershell
# Start service
net start faktory

# Stop service
net stop faktory

# Using PowerShell
Start-Service faktory
Stop-Service faktory
Restart-Service faktory

# Check status
Get-Service faktory
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream faktory_backend {
    server 127.0.0.1:7419;
}

server {
    listen 80;
    server_name faktory.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name faktory.example.com;

    ssl_certificate /etc/ssl/certs/faktory.example.com.crt;
    ssl_certificate_key /etc/ssl/private/faktory.example.com.key;

    location / {
        proxy_pass http://faktory_backend;
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
    ServerName faktory.example.com
    Redirect permanent / https://faktory.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName faktory.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/faktory.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/faktory.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:7419/
    ProxyPassReverse / http://127.0.0.1:7419/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend faktory_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/faktory.pem
    redirect scheme https if !{ ssl_fc }
    default_backend faktory_backend

backend faktory_backend
    balance roundrobin
    server faktory1 127.0.0.1:7419 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R faktory:faktory /etc/faktory
sudo chmod 750 /etc/faktory

# Configure firewall
sudo firewall-cmd --permanent --add-port=7419/tcp
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
sudo systemctl status faktory

# View logs
sudo journalctl -u faktory -f

# Monitor resource usage
top -p $(pgrep faktory)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/faktory"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/faktory-backup-$DATE.tar.gz" /etc/faktory /var/lib/faktory

echo "Backup completed: $BACKUP_DIR/faktory-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop faktory

# Restore from backup
tar -xzf /backup/faktory/faktory-backup-*.tar.gz -C /

# Start service
sudo systemctl start faktory
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u faktory -n 100
sudo tail -f /var/log/faktory/faktory.log

# Check configuration
faktory --version

# Check permissions
ls -la /etc/faktory
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 7419

# Test connectivity
telnet localhost 7419

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep faktory)

# Check disk I/O
iotop -p $(pgrep faktory)

# Check connections
ss -an | grep 7419
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  faktory:
    image: faktory:latest
    ports:
      - "7419:7419"
    volumes:
      - ./config:/etc/faktory
      - ./data:/var/lib/faktory
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update faktory

# Debian/Ubuntu
sudo apt update && sudo apt upgrade faktory

# Arch Linux
sudo pacman -Syu faktory

# Alpine Linux
apk update && apk upgrade faktory

# openSUSE
sudo zypper update faktory

# FreeBSD
pkg update && pkg upgrade faktory

# Always backup before updates
tar -czf /backup/faktory-pre-update-$(date +%Y%m%d).tar.gz /etc/faktory

# Restart after updates
sudo systemctl restart faktory
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/faktory

# Clean old logs
find /var/log/faktory -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/faktory
```

## Additional Resources

- Official Documentation: https://docs.faktory.org/
- GitHub Repository: https://github.com/faktory/faktory
- Community Forum: https://forum.faktory.org/
- Best Practices Guide: https://docs.faktory.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
