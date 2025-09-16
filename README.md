# influxdb Installation Guide

influxdb is a free and open-source time series database. InfluxDB is designed for high-performance time series data storage and retrieval, serving as an open-source alternative to proprietary solutions like AWS Timestream or Azure Time Series Insights

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
  - CPU: 2+ cores recommended
  - RAM: 2GB minimum (8GB+ for production)
  - Storage: 10GB+ for data
  - Network: HTTP/HTTPS API access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8086 (default influxdb port)
  - Port 8088 for RPC
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

# Install influxdb
sudo dnf install -y influxdb

# Enable and start service
sudo systemctl enable --now influxdb

# Configure firewall
sudo firewall-cmd --permanent --add-port=8086/tcp
sudo firewall-cmd --reload

# Verify installation
influx --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install influxdb
sudo apt install -y influxdb

# Enable and start service
sudo systemctl enable --now influxdb

# Configure firewall
sudo ufw allow 8086

# Verify installation
influx --version
```

### Arch Linux

```bash
# Install influxdb
sudo pacman -S influxdb

# Enable and start service
sudo systemctl enable --now influxdb

# Verify installation
influx --version
```

### Alpine Linux

```bash
# Install influxdb
apk add --no-cache influxdb

# Enable and start service
rc-update add influxdb default
rc-service influxdb start

# Verify installation
influx --version
```

### openSUSE/SLES

```bash
# Install influxdb
sudo zypper install -y influxdb

# Enable and start service
sudo systemctl enable --now influxdb

# Configure firewall
sudo firewall-cmd --permanent --add-port=8086/tcp
sudo firewall-cmd --reload

# Verify installation
influx --version
```

### macOS

```bash
# Using Homebrew
brew install influxdb

# Start service
brew services start influxdb

# Verify installation
influx --version
```

### FreeBSD

```bash
# Using pkg
pkg install influxdb

# Enable in rc.conf
echo 'influxdb_enable="YES"' >> /etc/rc.conf

# Start service
service influxdb start

# Verify installation
influx --version
```

### Windows

```bash
# Using Chocolatey
choco install influxdb

# Or using Scoop
scoop install influxdb

# Verify installation
influx --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/influxdb

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
influx --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable influxdb

# Start service
sudo systemctl start influxdb

# Stop service
sudo systemctl stop influxdb

# Restart service
sudo systemctl restart influxdb

# Check status
sudo systemctl status influxdb

# View logs
sudo journalctl -u influxdb -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add influxdb default

# Start service
rc-service influxdb start

# Stop service
rc-service influxdb stop

# Restart service
rc-service influxdb restart

# Check status
rc-service influxdb status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'influxdb_enable="YES"' >> /etc/rc.conf

# Start service
service influxdb start

# Stop service
service influxdb stop

# Restart service
service influxdb restart

# Check status
service influxdb status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start influxdb
brew services stop influxdb
brew services restart influxdb

# Check status
brew services list | grep influxdb
```

### Windows Service Manager

```powershell
# Start service
net start influxdb

# Stop service
net stop influxdb

# Using PowerShell
Start-Service influxdb
Stop-Service influxdb
Restart-Service influxdb

# Check status
Get-Service influxdb
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream influxdb_backend {
    server 127.0.0.1:8086;
}

server {
    listen 80;
    server_name influxdb.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name influxdb.example.com;

    ssl_certificate /etc/ssl/certs/influxdb.example.com.crt;
    ssl_certificate_key /etc/ssl/private/influxdb.example.com.key;

    location / {
        proxy_pass http://influxdb_backend;
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
    ServerName influxdb.example.com
    Redirect permanent / https://influxdb.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName influxdb.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/influxdb.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/influxdb.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8086/
    ProxyPassReverse / http://127.0.0.1:8086/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend influxdb_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/influxdb.pem
    redirect scheme https if !{ ssl_fc }
    default_backend influxdb_backend

backend influxdb_backend
    balance roundrobin
    server influxdb1 127.0.0.1:8086 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R influxdb:influxdb /etc/influxdb
sudo chmod 750 /etc/influxdb

# Configure firewall
sudo firewall-cmd --permanent --add-port=8086/tcp
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
sudo systemctl status influxdb

# View logs
sudo journalctl -u influxdb -f

# Monitor resource usage
top -p $(pgrep influxdb)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/influxdb"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/influxdb-backup-$DATE.tar.gz" /etc/influxdb /var/lib/influxdb

echo "Backup completed: $BACKUP_DIR/influxdb-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop influxdb

# Restore from backup
tar -xzf /backup/influxdb/influxdb-backup-*.tar.gz -C /

# Start service
sudo systemctl start influxdb
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u influxdb -n 100
sudo tail -f /var/log/influxdb/influxdb.log

# Check configuration
influx --version

# Check permissions
ls -la /etc/influxdb
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8086

# Test connectivity
telnet localhost 8086

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep influxdb)

# Check disk I/O
iotop -p $(pgrep influxdb)

# Check connections
ss -an | grep 8086
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  influxdb:
    image: influxdb:latest
    ports:
      - "8086:8086"
    volumes:
      - ./config:/etc/influxdb
      - ./data:/var/lib/influxdb
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update influxdb

# Debian/Ubuntu
sudo apt update && sudo apt upgrade influxdb

# Arch Linux
sudo pacman -Syu influxdb

# Alpine Linux
apk update && apk upgrade influxdb

# openSUSE
sudo zypper update influxdb

# FreeBSD
pkg update && pkg upgrade influxdb

# Always backup before updates
tar -czf /backup/influxdb-pre-update-$(date +%Y%m%d).tar.gz /etc/influxdb

# Restart after updates
sudo systemctl restart influxdb
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/influxdb

# Clean old logs
find /var/log/influxdb -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/influxdb
```

## Additional Resources

- Official Documentation: https://docs.influxdb.org/
- GitHub Repository: https://github.com/influxdb/influxdb
- Community Forum: https://forum.influxdb.org/
- Best Practices Guide: https://docs.influxdb.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
