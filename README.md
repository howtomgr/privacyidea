# privacyidea Installation Guide

privacyidea is a free and open-source multi-factor authentication system. privacyIDEA is a modular authentication system supporting various authentication devices, serving as an alternative to RSA SecurID or Duo

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
  - Storage: 500MB for installation
  - Network: HTTPS for authentication
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 5000 (default privacyidea port)
  - RADIUS on 1812 if enabled
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

# Install privacyidea
sudo dnf install -y privacyidea

# Enable and start service
sudo systemctl enable --now privacyidea

# Configure firewall
sudo firewall-cmd --permanent --add-port=5000/tcp
sudo firewall-cmd --reload

# Verify installation
pi-manage --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install privacyidea
sudo apt install -y privacyidea

# Enable and start service
sudo systemctl enable --now privacyidea

# Configure firewall
sudo ufw allow 5000

# Verify installation
pi-manage --version
```

### Arch Linux

```bash
# Install privacyidea
sudo pacman -S privacyidea

# Enable and start service
sudo systemctl enable --now privacyidea

# Verify installation
pi-manage --version
```

### Alpine Linux

```bash
# Install privacyidea
apk add --no-cache privacyidea

# Enable and start service
rc-update add privacyidea default
rc-service privacyidea start

# Verify installation
pi-manage --version
```

### openSUSE/SLES

```bash
# Install privacyidea
sudo zypper install -y privacyidea

# Enable and start service
sudo systemctl enable --now privacyidea

# Configure firewall
sudo firewall-cmd --permanent --add-port=5000/tcp
sudo firewall-cmd --reload

# Verify installation
pi-manage --version
```

### macOS

```bash
# Using Homebrew
brew install privacyidea

# Start service
brew services start privacyidea

# Verify installation
pi-manage --version
```

### FreeBSD

```bash
# Using pkg
pkg install privacyidea

# Enable in rc.conf
echo 'privacyidea_enable="YES"' >> /etc/rc.conf

# Start service
service privacyidea start

# Verify installation
pi-manage --version
```

### Windows

```bash
# Using Chocolatey
choco install privacyidea

# Or using Scoop
scoop install privacyidea

# Verify installation
pi-manage --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/privacyidea

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
pi-manage --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable privacyidea

# Start service
sudo systemctl start privacyidea

# Stop service
sudo systemctl stop privacyidea

# Restart service
sudo systemctl restart privacyidea

# Check status
sudo systemctl status privacyidea

# View logs
sudo journalctl -u privacyidea -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add privacyidea default

# Start service
rc-service privacyidea start

# Stop service
rc-service privacyidea stop

# Restart service
rc-service privacyidea restart

# Check status
rc-service privacyidea status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'privacyidea_enable="YES"' >> /etc/rc.conf

# Start service
service privacyidea start

# Stop service
service privacyidea stop

# Restart service
service privacyidea restart

# Check status
service privacyidea status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start privacyidea
brew services stop privacyidea
brew services restart privacyidea

# Check status
brew services list | grep privacyidea
```

### Windows Service Manager

```powershell
# Start service
net start privacyidea

# Stop service
net stop privacyidea

# Using PowerShell
Start-Service privacyidea
Stop-Service privacyidea
Restart-Service privacyidea

# Check status
Get-Service privacyidea
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream privacyidea_backend {
    server 127.0.0.1:5000;
}

server {
    listen 80;
    server_name privacyidea.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name privacyidea.example.com;

    ssl_certificate /etc/ssl/certs/privacyidea.example.com.crt;
    ssl_certificate_key /etc/ssl/private/privacyidea.example.com.key;

    location / {
        proxy_pass http://privacyidea_backend;
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
    ServerName privacyidea.example.com
    Redirect permanent / https://privacyidea.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName privacyidea.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/privacyidea.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/privacyidea.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:5000/
    ProxyPassReverse / http://127.0.0.1:5000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend privacyidea_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/privacyidea.pem
    redirect scheme https if !{ ssl_fc }
    default_backend privacyidea_backend

backend privacyidea_backend
    balance roundrobin
    server privacyidea1 127.0.0.1:5000 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R privacyidea:privacyidea /etc/privacyidea
sudo chmod 750 /etc/privacyidea

# Configure firewall
sudo firewall-cmd --permanent --add-port=5000/tcp
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
sudo systemctl status privacyidea

# View logs
sudo journalctl -u privacyidea -f

# Monitor resource usage
top -p $(pgrep privacyidea)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/privacyidea"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/privacyidea-backup-$DATE.tar.gz" /etc/privacyidea /var/lib/privacyidea

echo "Backup completed: $BACKUP_DIR/privacyidea-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop privacyidea

# Restore from backup
tar -xzf /backup/privacyidea/privacyidea-backup-*.tar.gz -C /

# Start service
sudo systemctl start privacyidea
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u privacyidea -n 100
sudo tail -f /var/log/privacyidea/privacyidea.log

# Check configuration
pi-manage --version

# Check permissions
ls -la /etc/privacyidea
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 5000

# Test connectivity
telnet localhost 5000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep privacyidea)

# Check disk I/O
iotop -p $(pgrep privacyidea)

# Check connections
ss -an | grep 5000
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  privacyidea:
    image: privacyidea:latest
    ports:
      - "5000:5000"
    volumes:
      - ./config:/etc/privacyidea
      - ./data:/var/lib/privacyidea
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update privacyidea

# Debian/Ubuntu
sudo apt update && sudo apt upgrade privacyidea

# Arch Linux
sudo pacman -Syu privacyidea

# Alpine Linux
apk update && apk upgrade privacyidea

# openSUSE
sudo zypper update privacyidea

# FreeBSD
pkg update && pkg upgrade privacyidea

# Always backup before updates
tar -czf /backup/privacyidea-pre-update-$(date +%Y%m%d).tar.gz /etc/privacyidea

# Restart after updates
sudo systemctl restart privacyidea
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/privacyidea

# Clean old logs
find /var/log/privacyidea -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/privacyidea
```

## Additional Resources

- Official Documentation: https://docs.privacyidea.org/
- GitHub Repository: https://github.com/privacyidea/privacyidea
- Community Forum: https://forum.privacyidea.org/
- Best Practices Guide: https://docs.privacyidea.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
