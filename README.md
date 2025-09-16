# sonic Installation Guide

sonic is a free and open-source search backend. Sonic provides fast, lightweight, and schemaless search backend

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
  - RAM: 256MB minimum
  - Storage: 5GB for indices
  - Network: Sonic protocol
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 1491 (default sonic port)
  - None
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

# Install sonic
sudo dnf install -y sonic

# Enable and start service
sudo systemctl enable --now sonic

# Configure firewall
sudo firewall-cmd --permanent --add-port=1491/tcp
sudo firewall-cmd --reload

# Verify installation
sonic --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install sonic
sudo apt install -y sonic

# Enable and start service
sudo systemctl enable --now sonic

# Configure firewall
sudo ufw allow 1491

# Verify installation
sonic --version
```

### Arch Linux

```bash
# Install sonic
sudo pacman -S sonic

# Enable and start service
sudo systemctl enable --now sonic

# Verify installation
sonic --version
```

### Alpine Linux

```bash
# Install sonic
apk add --no-cache sonic

# Enable and start service
rc-update add sonic default
rc-service sonic start

# Verify installation
sonic --version
```

### openSUSE/SLES

```bash
# Install sonic
sudo zypper install -y sonic

# Enable and start service
sudo systemctl enable --now sonic

# Configure firewall
sudo firewall-cmd --permanent --add-port=1491/tcp
sudo firewall-cmd --reload

# Verify installation
sonic --version
```

### macOS

```bash
# Using Homebrew
brew install sonic

# Start service
brew services start sonic

# Verify installation
sonic --version
```

### FreeBSD

```bash
# Using pkg
pkg install sonic

# Enable in rc.conf
echo 'sonic_enable="YES"' >> /etc/rc.conf

# Start service
service sonic start

# Verify installation
sonic --version
```

### Windows

```bash
# Using Chocolatey
choco install sonic

# Or using Scoop
scoop install sonic

# Verify installation
sonic --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/sonic

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
sonic --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable sonic

# Start service
sudo systemctl start sonic

# Stop service
sudo systemctl stop sonic

# Restart service
sudo systemctl restart sonic

# Check status
sudo systemctl status sonic

# View logs
sudo journalctl -u sonic -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add sonic default

# Start service
rc-service sonic start

# Stop service
rc-service sonic stop

# Restart service
rc-service sonic restart

# Check status
rc-service sonic status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'sonic_enable="YES"' >> /etc/rc.conf

# Start service
service sonic start

# Stop service
service sonic stop

# Restart service
service sonic restart

# Check status
service sonic status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start sonic
brew services stop sonic
brew services restart sonic

# Check status
brew services list | grep sonic
```

### Windows Service Manager

```powershell
# Start service
net start sonic

# Stop service
net stop sonic

# Using PowerShell
Start-Service sonic
Stop-Service sonic
Restart-Service sonic

# Check status
Get-Service sonic
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream sonic_backend {
    server 127.0.0.1:1491;
}

server {
    listen 80;
    server_name sonic.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name sonic.example.com;

    ssl_certificate /etc/ssl/certs/sonic.example.com.crt;
    ssl_certificate_key /etc/ssl/private/sonic.example.com.key;

    location / {
        proxy_pass http://sonic_backend;
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
    ServerName sonic.example.com
    Redirect permanent / https://sonic.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName sonic.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/sonic.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/sonic.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:1491/
    ProxyPassReverse / http://127.0.0.1:1491/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend sonic_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/sonic.pem
    redirect scheme https if !{ ssl_fc }
    default_backend sonic_backend

backend sonic_backend
    balance roundrobin
    server sonic1 127.0.0.1:1491 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R sonic:sonic /etc/sonic
sudo chmod 750 /etc/sonic

# Configure firewall
sudo firewall-cmd --permanent --add-port=1491/tcp
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
sudo systemctl status sonic

# View logs
sudo journalctl -u sonic -f

# Monitor resource usage
top -p $(pgrep sonic)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/sonic"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/sonic-backup-$DATE.tar.gz" /etc/sonic /var/lib/sonic

echo "Backup completed: $BACKUP_DIR/sonic-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop sonic

# Restore from backup
tar -xzf /backup/sonic/sonic-backup-*.tar.gz -C /

# Start service
sudo systemctl start sonic
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u sonic -n 100
sudo tail -f /var/log/sonic/sonic.log

# Check configuration
sonic --version

# Check permissions
ls -la /etc/sonic
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 1491

# Test connectivity
telnet localhost 1491

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep sonic)

# Check disk I/O
iotop -p $(pgrep sonic)

# Check connections
ss -an | grep 1491
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  sonic:
    image: sonic:latest
    ports:
      - "1491:1491"
    volumes:
      - ./config:/etc/sonic
      - ./data:/var/lib/sonic
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update sonic

# Debian/Ubuntu
sudo apt update && sudo apt upgrade sonic

# Arch Linux
sudo pacman -Syu sonic

# Alpine Linux
apk update && apk upgrade sonic

# openSUSE
sudo zypper update sonic

# FreeBSD
pkg update && pkg upgrade sonic

# Always backup before updates
tar -czf /backup/sonic-pre-update-$(date +%Y%m%d).tar.gz /etc/sonic

# Restart after updates
sudo systemctl restart sonic
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/sonic

# Clean old logs
find /var/log/sonic -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/sonic
```

## Additional Resources

- Official Documentation: https://docs.sonic.org/
- GitHub Repository: https://github.com/sonic/sonic
- Community Forum: https://forum.sonic.org/
- Best Practices Guide: https://docs.sonic.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
