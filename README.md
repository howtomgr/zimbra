# Zimbra Installation Guide

Zimbra is a free and open-source Collaboration Suite. An open-source collaborative software suite

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
  - CPU: 2 cores minimum (4+ cores recommended)
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 1GB for installation
  - Network: 443/7071 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 443/7071 (default zimbra port)
- **Dependencies**:
  - perl, sysstat, sqlite
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

# Install zimbra
sudo dnf install -y zimbra perl, sysstat, sqlite

# Enable and start service
sudo systemctl enable --now zimbra

# Configure firewall
sudo firewall-cmd --permanent --add-service=zimbra
sudo firewall-cmd --reload

# Verify installation
zimbra --version || systemctl status zimbra
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install zimbra
sudo apt install -y zimbra perl, sysstat, sqlite

# Enable and start service
sudo systemctl enable --now zimbra

# Configure firewall
sudo ufw allow 443/7071

# Verify installation
zimbra --version || systemctl status zimbra
```

### Arch Linux

```bash
# Install zimbra
sudo pacman -S zimbra

# Enable and start service
sudo systemctl enable --now zimbra

# Verify installation
zimbra --version || systemctl status zimbra
```

### Alpine Linux

```bash
# Install zimbra
apk add --no-cache zimbra

# Enable and start service
rc-update add zimbra default
rc-service zimbra start

# Verify installation
zimbra --version || rc-service zimbra status
```

### openSUSE/SLES

```bash
# Install zimbra
sudo zypper install -y zimbra perl, sysstat, sqlite

# Enable and start service
sudo systemctl enable --now zimbra

# Configure firewall
sudo firewall-cmd --permanent --add-service=zimbra
sudo firewall-cmd --reload

# Verify installation
zimbra --version || systemctl status zimbra
```

### macOS

```bash
# Using Homebrew
brew install zimbra

# Start service
brew services start zimbra

# Verify installation
zimbra --version
```

### FreeBSD

```bash
# Using pkg
pkg install zimbra

# Enable in rc.conf
echo 'zimbra_enable="YES"' >> /etc/rc.conf

# Start service
service zimbra start

# Verify installation
zimbra --version || service zimbra status
```

### Windows

```powershell
# Using Chocolatey
choco install zimbra

# Or using Scoop
scoop install zimbra

# Verify installation
zimbra --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /opt/zimbra/conf

# Set up basic configuration
sudo tee /opt/zimbra/conf/zimbra.conf << 'EOF'
# Zimbra Configuration
zimbraMailThreadPoolSize=250
EOF

# Test configuration
sudo zimbra -t || sudo zimbra configtest

# Reload service
sudo systemctl reload zimbra
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R zimbra:zimbra /opt/zimbra/conf
sudo chmod 750 /opt/zimbra/conf

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable zimbra

# Start service
sudo systemctl start zimbra

# Stop service
sudo systemctl stop zimbra

# Restart service
sudo systemctl restart zimbra

# Reload configuration
sudo systemctl reload zimbra

# Check status
sudo systemctl status zimbra

# View logs
sudo journalctl -u zimbra -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add zimbra default

# Start service
rc-service zimbra start

# Stop service
rc-service zimbra stop

# Restart service
rc-service zimbra restart

# Check status
rc-service zimbra status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'zimbra_enable="YES"' >> /etc/rc.conf

# Start service
service zimbra start

# Stop service
service zimbra stop

# Restart service
service zimbra restart

# Check status
service zimbra status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start zimbra
brew services stop zimbra
brew services restart zimbra

# Check status
brew services list | grep zimbra
```

### Windows Service Manager

```powershell
# Start service
net start zimbra

# Stop service
net stop zimbra

# Using PowerShell
Start-Service zimbra
Stop-Service zimbra
Restart-Service zimbra

# Check status
Get-Service zimbra
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /opt/zimbra/conf/zimbra.conf << 'EOF'
zimbraMailThreadPoolSize=250
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart zimbra
```

### Clustering and High Availability

```bash
# Configure clustering (if supported)
# See official documentation for cluster setup

# Basic load balancing setup example
# Configure multiple instances on different ports
```

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream zimbra_backend {
    server 127.0.0.1:443/7071;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name zimbra.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name zimbra.example.com;

    ssl_certificate /etc/ssl/certs/zimbra.example.com.crt;
    ssl_certificate_key /etc/ssl/private/zimbra.example.com.key;

    location / {
        proxy_pass http://zimbra_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support (if needed)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName zimbra.example.com
    Redirect permanent / https://zimbra.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName zimbra.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/zimbra.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/zimbra.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:443/7071/
    ProxyPassReverse / http://127.0.0.1:443/7071/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:443/7071/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend zimbra_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/zimbra.pem
    redirect scheme https if !{ ssl_fc }
    default_backend zimbra_backend

backend zimbra_backend
    balance roundrobin
    option httpchk GET /health
    server zimbra1 127.0.0.1:443/7071 check
    server zimbra2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R zimbra:zimbra /opt/zimbra/conf
sudo chmod 750 /opt/zimbra/conf

# Configure firewall
sudo firewall-cmd --permanent --add-service=zimbra
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/zimbra.conf << 'EOF'
[zimbra]
enabled = true
port = 443/7071
filter = zimbra
logpath = /opt/zimbra/log/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/zimbra.key \
    -out /etc/ssl/certs/zimbra.crt

# Configure SSL in zimbra
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE zimbra_db;
CREATE USER zimbra_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE zimbra_db TO zimbra_user;
EOF

# Configure zimbra to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE zimbra_db;
CREATE USER 'zimbra_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON zimbra_db.* TO 'zimbra_user'@'localhost';
FLUSH PRIVILEGES;
EOF
```

## Performance Optimization

### System Tuning

```bash
# Kernel parameters
sudo tee -a /etc/sysctl.conf << EOF
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535
net.core.netdev_max_backlog = 5000
vm.swappiness = 10
EOF

sudo sysctl -p

# Zimbra specific tuning
zimbraMailThreadPoolSize=250
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
zimbra soft nofile 65535
zimbra hard nofile 65535
zimbra soft nproc 32768
zimbra hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'zimbra'
    static_configs:
      - targets: ['localhost:443/7071']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet zimbra; then
    echo "Zimbra is running"
    exit 0
else
    echo "Zimbra is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/zimbra << 'EOF'
/opt/zimbra/log/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 zimbra zimbra
    postrotate
        systemctl reload zimbra > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Zimbra backup script
BACKUP_DIR="/backup/zimbra"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop zimbra

# Backup configuration
tar -czf "$BACKUP_DIR/zimbra-config-$DATE.tar.gz" /opt/zimbra/conf

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/zimbra-data-$DATE.tar.gz" /var/lib/zimbra

# Start service
systemctl start zimbra

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop zimbra

# Restore configuration
sudo tar -xzf /backup/zimbra/zimbra-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/zimbra/zimbra-data-*.tar.gz -C /

# Set permissions
sudo chown -R zimbra:zimbra /opt/zimbra/conf
sudo chown -R zimbra:zimbra /var/lib/zimbra

# Start service
sudo systemctl start zimbra
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u zimbra -n 100
sudo tail -f /opt/zimbra/log/*.log

# Check configuration
sudo zimbra -t || sudo zimbra configtest

# Check permissions
ls -la /opt/zimbra/conf
ls -la /var/lib/zimbra
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 443/7071
sudo netstat -tlnp | grep 443/7071

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 443/7071
nc -zv localhost 443/7071
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep zimbra)
htop -p $(pgrep zimbra)

# Check connections
ss -ant | grep :443/7071 | wc -l

# Monitor I/O
iotop -p $(pgrep zimbra)
```

### Debug Mode

```bash
# Run in debug mode
sudo zimbra -d
# or
sudo zimbra debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  zimbra:
    image: zimbra:latest
    container_name: zimbra
    ports:
      - "443/7071:443/7071"
    volumes:
      - ./config:/opt/zimbra/conf
      - ./data:/var/lib/zimbra
    environment:
      - zimbra_CONFIG=/opt/zimbra/conf/zimbra.conf
    restart: unless-stopped
    networks:
      - zimbra_net

networks:
  zimbra_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: zimbra
spec:
  replicas: 3
  selector:
    matchLabels:
      app: zimbra
  template:
    metadata:
      labels:
        app: zimbra
    spec:
      containers:
      - name: zimbra
        image: zimbra:latest
        ports:
        - containerPort: 443/7071
        volumeMounts:
        - name: config
          mountPath: /opt/zimbra/conf
      volumes:
      - name: config
        configMap:
          name: zimbra-config
---
apiVersion: v1
kind: Service
metadata:
  name: zimbra
spec:
  selector:
    app: zimbra
  ports:
  - port: 443/7071
    targetPort: 443/7071
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure Zimbra
  hosts: all
  become: yes
  tasks:
    - name: Install zimbra
      package:
        name: zimbra
        state: present
    
    - name: Configure zimbra
      template:
        src: zimbra.conf.j2
        dest: /opt/zimbra/conf/zimbra.conf
        owner: zimbra
        group: zimbra
        mode: '0640'
      notify: restart zimbra
    
    - name: Start and enable zimbra
      systemd:
        name: zimbra
        state: started
        enabled: yes
  
  handlers:
    - name: restart zimbra
      systemd:
        name: zimbra
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update zimbra

# Debian/Ubuntu
sudo apt update && sudo apt upgrade zimbra

# Arch Linux
sudo pacman -Syu zimbra

# Alpine Linux
apk update && apk upgrade zimbra

# openSUSE
sudo zypper update zimbra

# FreeBSD
pkg update && pkg upgrade zimbra

# Always backup before updates
tar -czf /backup/zimbra-pre-update-$(date +%Y%m%d).tar.gz /opt/zimbra/conf

# Restart after updates
sudo systemctl restart zimbra
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /opt/zimbra/log -name "*.log" -mtime +30 -delete

# Verify integrity
sudo zimbra --verify || sudo zimbra check

# Update databases (if applicable)
sudo zimbra-update-db

# Optimize performance
sudo zimbra-optimize

# Check for security updates
sudo zimbra --security-check
```

## Additional Resources

- Official Documentation: https://docs.zimbra.org/
- GitHub Repository: https://github.com/zimbra/zimbra
- Community Forum: https://forum.zimbra.org/
- Wiki: https://wiki.zimbra.org/
- Comparison vs Exchange, Kopano, Kolab, Nextcloud: https://docs.zimbra.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
