# Certbot Installation Guide

Certbot is a free and open-source ACME client for Let's Encrypt SSL/TLS certificate automation. Developed by the Electronic Frontier Foundation (EFF), Certbot automates the process of obtaining and renewing SSL/TLS certificates from Let's Encrypt, providing a free alternative to commercial certificate authorities like DigiCert, Comodo, or GoDaddy

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
  - Storage: 100MB for installation
  - Network: HTTPS access to Let's Encrypt servers
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.12+ (Sierra or newer)
  - Windows: Windows 10+ with WSL2
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80/443 (default Certbot port)
  - Port 80 for HTTP-01 challenge, Port 443 for TLS-ALPN-01 challenge
- **Dependencies**:
  - Python 3.6+, nginx/Apache (for web server plugins)
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

# Install Certbot
sudo dnf install -y certbot

# Enable and start service
sudo systemctl enable --now certbot

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/443/tcp
sudo firewall-cmd --reload

# Verify installation
certbot --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install Certbot
sudo apt install -y certbot

# Enable and start service
sudo systemctl enable --now certbot

# Configure firewall
sudo ufw allow 80/443

# Verify installation
certbot --version
```

### Arch Linux

```bash
# Install Certbot
sudo pacman -S certbot

# Enable and start service
sudo systemctl enable --now certbot

# Verify installation
certbot --version
```

### Alpine Linux

```bash
# Install Certbot
apk add --no-cache certbot

# Enable and start service
rc-update add certbot default
rc-service certbot start

# Verify installation
certbot --version
```

### openSUSE/SLES

```bash
# Install Certbot
sudo zypper install -y certbot

# Enable and start service
sudo systemctl enable --now certbot

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/443/tcp
sudo firewall-cmd --reload

# Verify installation
certbot --version
```

### macOS

```bash
# Using Homebrew
brew install certbot

# Start service
brew services start certbot

# Verify installation
certbot --version
```

### FreeBSD

```bash
# Using pkg
pkg install certbot

# Enable in rc.conf
echo 'certbot_enable="YES"' >> /etc/rc.conf

# Start service
service certbot start

# Verify installation
certbot --version
```

### Windows

```bash
# Using Chocolatey
choco install certbot

# Or using Scoop
scoop install certbot

# Verify installation
certbot --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/letsencrypt

# Set up basic configuration
# Configuration details will vary based on your specific needs
# See official documentation for detailed configuration options

# Test configuration
certbot certificates
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable certbot

# Start service
sudo systemctl start certbot

# Stop service
sudo systemctl stop certbot

# Restart service
sudo systemctl restart certbot

# Check status
sudo systemctl status certbot

# View logs
sudo journalctl -u certbot -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add certbot default

# Start service
rc-service certbot start

# Stop service
rc-service certbot stop

# Restart service
rc-service certbot restart

# Check status
rc-service certbot status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'certbot_enable="YES"' >> /etc/rc.conf

# Start service
service certbot start

# Stop service
service certbot stop

# Restart service
service certbot restart

# Check status
service certbot status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start certbot
brew services stop certbot
brew services restart certbot

# Check status
brew services list | grep certbot
```

### Windows Service Manager

```powershell
# Start service
net start certbot

# Stop service
net stop certbot

# Using PowerShell
Start-Service certbot
Stop-Service certbot
Restart-Service certbot

# Check status
Get-Service certbot
```

## Advanced Configuration

### Advanced Certbot Configuration

See the official documentation for advanced configuration options including:
- High availability setup
- Performance tuning
- Security hardening
- Integration with other services


## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream certbot_backend {
    server 127.0.0.1:80/443;
}

server {
    listen 80;
    server_name certbot.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name certbot.example.com;

    ssl_certificate /etc/ssl/certs/certbot.example.com.crt;
    ssl_certificate_key /etc/ssl/private/certbot.example.com.key;

    location / {
        proxy_pass http://certbot_backend;
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
    ServerName certbot.example.com
    Redirect permanent / https://certbot.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName certbot.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/certbot.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/certbot.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/443/
    ProxyPassReverse / http://127.0.0.1:80/443/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend certbot_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/certbot.pem
    redirect scheme https if !{ ssl_fc }
    default_backend certbot_backend

backend certbot_backend
    balance roundrobin
    server certbot1 127.0.0.1:80/443 check
```

## Security Configuration

### Security Best Practices

```bash
# Set appropriate permissions
sudo chown -R certbot:certbot /etc/letsencrypt
sudo chmod 750 /etc/letsencrypt

# Configure firewall rules
sudo firewall-cmd --permanent --add-port=80/443/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

Not applicable

## Performance Optimization

### 8. Performance Tuning

```bash
# System tuning for Certbot
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Monitor performance
certbot renew --dry-run
```

## Monitoring

### Monitoring Setup

```bash
# Basic monitoring
sudo systemctl status certbot
sudo journalctl -u certbot -f

# Set up health checks
curl -f http://localhost:80/health || exit 1
```

## 9. Backup and Restore

### Backup Procedures

```bash
#!/bin/bash
# Backup script
BACKUP_DIR="/backup/certbot"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf /backup/certbot-$(date +%Y%m%d).tar.gz /etc/letsencrypt

# Restore procedure
# Stop service, restore files, restart service
sudo systemctl stop certbot
# Restore backed up files
sudo systemctl start certbot
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u certbot -f
sudo tail -f /var/log/certbot/certbot.log

# Check configuration
certbot certificates

# Check permissions
ls -la /etc/letsencrypt
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80/443

# Test connectivity
telnet localhost 80/443

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep certbot)

# Check disk I/O
iotop -p $(pgrep certbot)

# Check network connections
ss -an | grep 80/443
```



## Integration Examples

### Example Integration

```yaml
# Docker Compose example
version: '3.8'
services:
  certbot:
    image: certbot:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/letsencrypt
      - ./data:/etc/letsencrypt
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update certbot

# Debian/Ubuntu
sudo apt update && sudo apt upgrade certbot

# Arch Linux
sudo pacman -Syu certbot

# Alpine Linux
apk update && apk upgrade certbot

# openSUSE
sudo zypper update certbot

# FreeBSD
pkg update && pkg upgrade certbot

# Always backup before updates
tar -czf /backup/certbot-$(date +%Y%m%d).tar.gz /etc/letsencrypt

# Restart after updates
sudo systemctl restart certbot
```

### Regular Maintenance Tasks

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/certbot

# Clean old logs
find /var/log/certbot -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /etc/letsencrypt

# Verify configuration
certbot certificates

# Test functionality
certbot renew --dry-run
```



## Additional Resources

- [Official Documentation](https://certbot.eff.org/docs/)
- [GitHub Repository](https://github.com/certbot/certbot)
- [Community Forum](https://community.letsencrypt.org/)
- [Best Practices Guide](https://certbot.eff.org/docs/best-practices.html)


---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
