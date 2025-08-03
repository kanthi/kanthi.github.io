+++
title = "FreeBSD Jails with Bastille"
date = "2025-01-30T15:47:36+05:30"
author = ""
draft = false
authorTwitter = "" #do not include @
cover = ""
tags = ["freebsd", "jails", "bastillebsd", "freebsd14"]
keywords = ["freebsd", "jails", "bastillebsd", "freebsd14"]
description = "Complete guide to FreeBSD 14 Jails with Bastille for production environments"
showFullContent = false
readingTime = false
hideComments = false
+++

# FreeBSD 14 Jails with Bastille: Complete Guide for Production Environments

FreeBSD Jails have long been considered one of the most robust containerization technologies available, predating Docker by over a decade. With FreeBSD 14's latest improvements including enhanced security features, better performance, and improved hardware support, combined with Bastille's modern jail management framework, they provide an incredibly powerful platform for running production workloads. In this comprehensive guide, we'll explore how to leverage FreeBSD 14 Jails with Bastille for enterprise-grade deployments.

## What are FreeBSD Jails?

FreeBSD Jails are an operating system-level virtualization technology that allows you to partition a FreeBSD system into several independent mini-systems called "jails." Each jail appears to be a complete FreeBSD installation from the perspective of processes running within it, but shares the same kernel with the host system.

### Key Benefits of FreeBSD Jails:

- **Security**: Complete process and file system isolation
- **Performance**: Near-native performance with minimal overhead
- **Resource Efficiency**: Share kernel resources while maintaining isolation
- **Stability**: Mature technology with over 20 years of development
- **Flexibility**: Support for both traditional and Linux binary compatibility

## What's New in FreeBSD 14?

FreeBSD 14 brings significant improvements that enhance the jail experience:

### Performance Enhancements
- **Improved VNET Performance**: Better network virtualization with reduced overhead
- **Enhanced ZFS Integration**: Better memory management and performance optimizations
- **Optimized Scheduler**: Better CPU scheduling for containerized workloads

### Security Improvements
- **Enhanced Capsicum**: More granular capability-based security
- **Improved MAC Framework**: Better mandatory access controls
- **Updated OpenSSL**: Latest cryptographic libraries and protocols

### Hardware Support
- **Better ARM64 Support**: Enhanced support for ARM-based systems
- **Updated Network Drivers**: Better performance for modern network hardware
- **Improved Virtualization**: Better support for running on hypervisors

### Developer Experience
- **Updated Base System**: Latest versions of core utilities and libraries
- **Improved pkg**: Better package management with enhanced dependency resolution
- **Enhanced Debugging**: Better tools for troubleshooting jail issues

## Introducing Bastille

Bastille is a modern jail management framework that simplifies the creation, configuration, and management of FreeBSD jails. It provides a Docker-like experience while maintaining the power and flexibility of traditional FreeBSD jails.

### Why Choose Bastille?

- **Simplicity**: Easy-to-use command-line interface
- **Templates**: Pre-built jail templates for common services
- **Networking**: Advanced networking capabilities with VNET support
- **Storage**: ZFS integration for snapshots and clones
- **Automation**: Infrastructure-as-code approach with Bastillefile support

## Installation and Initial Setup

### Prerequisites

Ensure your FreeBSD system meets the following requirements:

```bash
# FreeBSD 14.0 or later (14.2-RELEASE recommended)
freebsd-version

# ZFS recommended for advanced features
zpool status

# Sufficient disk space (minimum 20GB recommended)
df -h

# Enable required kernel modules
kldload if_bridge
kldload pf
```

### Installing Bastille

```bash
# Update package repository
pkg update

# Install from packages (recommended)
pkg install bastille

# Or install from ports for latest features
cd /usr/ports/sysutils/bastille && make install clean
```

### Initial Configuration

```bash
# Enable Bastille in rc.conf
echo 'bastille_enable="YES"' >> /etc/rc.conf

# Configure PF firewall (recommended)
echo 'pf_enable="YES"' >> /etc/rc.conf
echo 'pflog_enable="YES"' >> /etc/rc.conf

# Start services
service pf start
service bastille start
```

### Bootstrap Base System

```bash
# Bootstrap FreeBSD 14 base system for jails
bastille bootstrap 14.2-RELEASE

# Also bootstrap 14.1 for compatibility if needed
bastille bootstrap 14.1-RELEASE

# Verify bootstrap
bastille list
bastille list releases
```

## Basic Jail Operations

### Creating Your First Jail

```bash
# Create a basic jail with FreeBSD 14
bastille create myjail 14.2-RELEASE 10.17.89.10

# Create with VNET for advanced networking
bastille create -V myjail 14.2-RELEASE 10.17.89.10

# Start the jail
bastille start myjail

# Access the jail console
bastille console myjail
```

### Managing Jails

```bash
# List all jails
bastille list

# Stop a jail
bastille stop myjail

# Restart a jail
bastille restart myjail

# Destroy a jail
bastille destroy myjail
```

### Package Management

```bash
# Install packages in a jail
bastille pkg myjail install nginx

# Update packages
bastille pkg myjail update
bastille pkg myjail upgrade
```

## Advanced Networking Configuration

### VNET (Virtual Networking)

VNET provides each jail with its own virtualized network stack, with improved performance in FreeBSD 14:

```bash
# Create jail with VNET
bastille create -V webserver 14.2-RELEASE 10.17.89.11

# Configure bridge interface
bastille config webserver set vnet.interface = "bridge0"

# Enable additional VNET features in FreeBSD 14
bastille config webserver set allow.raw_sockets = 1
bastille config webserver set allow.socket_af = 1
```

### PF Firewall Rules

Create `/etc/pf.conf` for network security with FreeBSD 14 optimizations:

```pf
# Variables
ext_if = "em0"
jail_if = "bridge0"
jail_net = "10.17.89.0/24"

# Tables
table <jails> persist
table <web_ports> { 80, 443, 8080, 8443 }

# Scrub packets
scrub in all

# NAT for jail network
nat on $ext_if from $jail_net to any -> ($ext_if)

# Rate limiting for SSH
table <ssh_abuse> persist
pass in quick on $ext_if proto tcp to port 22 \
    keep state (max-src-conn 5, max-src-conn-rate 3/60, \
    overload <ssh_abuse> flush global)

# Allow jail traffic
pass quick on lo0
pass in quick on $jail_if
pass out quick on $ext_if from $jail_net
block in quick from <ssh_abuse>
```

## Storage Management with ZFS

### ZFS Dataset Configuration

```bash
# Create ZFS dataset for jails with FreeBSD 14 optimizations
zfs create -o mountpoint=/usr/local/bastille \
           -o compression=lz4 \
           -o atime=off \
           -o recordsize=128k \
           zroot/bastille

# Create datasets for jail storage
zfs create -o quota=50G zroot/bastille/jails
zfs create -o quota=10G zroot/bastille/cache
zfs create -o quota=5G zroot/bastille/logs

# Enable deduplication for base system (optional)
zfs create -o dedup=on zroot/bastille/releases
```

### Snapshots and Clones

```bash
# Create snapshot
bastille snapshot myjail

# List snapshots
bastille list -s

# Clone from snapshot
bastille clone myjail_snapshot newjail
```

## Bastille Templates

### Creating Custom Templates

Create a template directory structure:

```bash
mkdir -p /usr/local/bastille/templates/nginx-php/
```

Create template files:

**Bastillefile**:
```dockerfile
FROM 14.2-RELEASE

PKG nginx php83 php83-extensions

SYSRC nginx_enable=YES
SYSRC php_fpm_enable=YES

SERVICE nginx start
SERVICE php-fpm start

CMD nginx -g 'daemon off;'
```

**template.conf**:
```ini
[template]
name = nginx-php
description = Nginx with PHP-FPM
version = 1.0
```

### Using Templates

```bash
# Apply template to jail
bastille template myjail nginx-php

# Create jail from template with FreeBSD 14
bastille create -T nginx-php webserver1 14.2-RELEASE 10.17.89.12

# List available templates
bastille template list
```

## Monitoring and Logging

### System Resource Monitoring

```bash
# View jail resource usage
bastille top

# Detailed jail statistics
bastille stats myjail

# Process monitoring within jail
bastille cmd myjail top
```

### Centralized Logging

Configure syslog forwarding in jails:

```bash
# In jail: /etc/syslog.conf
*.* @10.17.89.1:514
```

Host syslog configuration:

```bash
# /etc/syslog.conf
+10.17.89.0/24
*.*    /var/log/jails.log
```

## Production Lab: Multi-Jail SaaS Environment

Now let's create a comprehensive production lab setup for a SaaS application using multiple jails on a single server.

### Architecture Overview

Our production environment will consist of:

- **Load Balancer Jail**: HAProxy for traffic distribution
- **Web Server Jails**: Multiple Nginx instances
- **Application Jails**: Node.js/Python application servers
- **Database Jail**: PostgreSQL database server
- **Cache Jail**: Redis for session storage and caching
- **Monitoring Jail**: Prometheus and Grafana
- **Log Management Jail**: ELK stack (Elasticsearch, Logstash, Kibana)

### Network Planning

```bash
# Network layout
# Host: 192.168.1.100
# Jail Network: 10.17.89.0/24
# - Load Balancer: 10.17.89.10
# - Web Servers: 10.17.89.11-13
# - App Servers: 10.17.89.21-23
# - Database: 10.17.89.30
# - Cache: 10.17.89.31
# - Monitoring: 10.17.89.40
# - Logging: 10.17.89.41
```

### Step 1: Infrastructure Setup

```bash
#!/bin/sh
# production-setup.sh

# Bootstrap base system with FreeBSD 14
bastille bootstrap 14.2-RELEASE

# Create bridge interface for jail networking
ifconfig bridge0 create
ifconfig bridge0 addm em0
ifconfig bridge0 inet 10.17.89.1/24

# Enable bridge in rc.conf for persistence
sysrc cloned_interfaces+="bridge0"
sysrc ifconfig_bridge0="addm em0 inet 10.17.89.1/24"

# Configure PF rules with FreeBSD 14 enhancements
cat > /etc/pf.conf << 'EOF'
ext_if = "em0"
jail_if = "bridge0"
jail_net = "10.17.89.0/24"

# Tables
table <web_servers> { 10.17.89.11, 10.17.89.12, 10.17.89.13 }
table <app_servers> { 10.17.89.21, 10.17.89.22, 10.17.89.23 }
table <ssh_abuse> persist

# Scrub packets for security
scrub in all

# NAT
nat on $ext_if from $jail_net to any -> ($ext_if)

# Load balancer access with rate limiting
rdr on $ext_if proto tcp from any to any port 80 -> 10.17.89.10
rdr on $ext_if proto tcp from any to any port 443 -> 10.17.89.10

# SSH access with protection
rdr on $ext_if proto tcp from any to any port 2222 -> 10.17.89.10 port 22

# Rules with enhanced security
pass quick on lo0
pass in quick on $ext_if proto tcp to port 22 \
    keep state (max-src-conn 5, max-src-conn-rate 3/60, \
    overload <ssh_abuse> flush global)
pass out quick
pass in quick on $jail_if
block in quick from <ssh_abuse>
EOF

# Reload firewall
pfctl -f /etc/pf.conf
```

### Step 2: Create Load Balancer Jail

```bash
# Create load balancer jail with FreeBSD 14
bastille create -V loadbalancer 14.2-RELEASE 10.17.89.10

# Install HAProxy (latest version)
bastille pkg loadbalancer install haproxy29

# Configure HAProxy
bastille cmd loadbalancer "cat > /usr/local/etc/haproxy.conf" << 'EOF'
global
    daemon
    log 127.0.0.1:514 local0
    chroot /var/haproxy
    user haproxy
    group haproxy

defaults
    mode http
    log global
    option httplog
    option dontlognull
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

frontend web_frontend
    bind *:80
    bind *:443 ssl crt /usr/local/etc/ssl/server.pem
    redirect scheme https if !{ ssl_fc }
    default_backend web_servers

backend web_servers
    balance roundrobin
    option httpchk GET /health
    server web1 10.17.89.11:80 check
    server web2 10.17.89.12:80 check
    server web3 10.17.89.13:80 check

frontend app_frontend
    bind *:8080
    default_backend app_servers

backend app_servers
    balance roundrobin
    option httpchk GET /api/health
    server app1 10.17.89.21:3000 check
    server app2 10.17.89.22:3000 check
    server app3 10.17.89.23:3000 check

listen stats
    bind *:8404
    stats enable
    stats uri /stats
    stats refresh 30s
EOF

# Enable and start HAProxy
bastille sysrc loadbalancer haproxy_enable=YES
bastille service loadbalancer haproxy start
```

### Step 3: Create Web Server Jails

```bash
# Create web server template
mkdir -p /usr/local/bastille/templates/nginx-web/

cat > /usr/local/bastille/templates/nginx-web/Bastillefile << 'EOF'
FROM 14.2-RELEASE

PKG nginx nginx-prometheus-exporter

# Copy configuration
CP usr/local/etc/nginx/nginx.conf

SYSRC nginx_enable=YES
SYSRC nginx_prometheus_exporter_enable=YES
SERVICE nginx start
SERVICE nginx_prometheus_exporter start
EOF

# Nginx configuration
mkdir -p /usr/local/bastille/templates/nginx-web/usr/local/etc/nginx/
cat > /usr/local/bastille/templates/nginx-web/usr/local/etc/nginx/nginx.conf << 'EOF'
worker_processes auto;
error_log /var/log/nginx/error.log;

events {
    worker_connections 1024;
}

http {
    include mime.types;
    default_type application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                   '$status $body_bytes_sent "$http_referer" '
                   '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;

    upstream app_backend {
        server 10.17.89.21:3000;
        server 10.17.89.22:3000;
        server 10.17.89.23:3000;
    }

    server {
        listen 80;
        server_name _;

        location /health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }

        location / {
            proxy_pass http://app_backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
EOF

# Create web server jails with FreeBSD 14
for i in {1..3}; do
    ip="10.17.89.1$i"
    bastille create -V web$i 14.2-RELEASE $ip
    bastille template web$i nginx-web
done
```

### Step 4: Create Application Server Jails

```bash
# Create Node.js application template
mkdir -p /usr/local/bastille/templates/nodejs-app/

cat > /usr/local/bastille/templates/nodejs-app/Bastillefile << 'EOF'
FROM 14.2-RELEASE

PKG node20 npm-node20

# Copy application
CP opt/app

SYSRC app_enable=YES
SERVICE app start
EOF

# Sample Node.js application
mkdir -p /usr/local/bastille/templates/nodejs-app/opt/app/
cat > /usr/local/bastille/templates/nodejs-app/opt/app/package.json << 'EOF'
{
  "name": "saas-app",
  "version": "1.0.0",
  "main": "server.js",
  "dependencies": {
    "express": "^4.18.0",
    "redis": "^4.0.0",
    "pg": "^8.7.0"
  }
}
EOF

cat > /usr/local/bastille/templates/nodejs-app/opt/app/server.js << 'EOF'
const express = require('express');
const redis = require('redis');
const { Pool } = require('pg');

const app = express();
const port = 3000;

// Database connection
const pool = new Pool({
  host: '10.17.89.30',
  port: 5432,
  database: 'saasapp',
  user: 'appuser',
  password: 'secure_password'
});

// Redis connection
const redisClient = redis.createClient({
  host: '10.17.89.31',
  port: 6379
});

app.use(express.json());

app.get('/api/health', (req, res) => {
  res.json({ status: 'healthy', timestamp: new Date().toISOString() });
});

app.get('/api/users', async (req, res) => {
  try {
    const result = await pool.query('SELECT * FROM users LIMIT 10');
    res.json(result.rows);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

app.listen(port, () => {
  console.log(`App server listening on port ${port}`);
});
EOF

# Create application server jails with FreeBSD 14
for i in {1..3}; do
    ip="10.17.89.2$i"
    bastille create -V app$i 14.2-RELEASE $ip
    bastille template app$i nodejs-app
    bastille cmd app$i "cd /opt/app && npm install"
done
```

### Step 5: Create Database Jail

```bash
# Create PostgreSQL jail with FreeBSD 14
bastille create -V database 14.2-RELEASE 10.17.89.30

# Install PostgreSQL 16 (latest stable)
bastille pkg database install postgresql16-server postgresql16-client

# Initialize database
bastille cmd database "service postgresql oneinitdb"
bastille sysrc database postgresql_enable=YES

# Configure PostgreSQL 16 with FreeBSD 14 optimizations
bastille cmd database "cat >> /var/db/postgres/data16/postgresql.conf" << 'EOF'
listen_addresses = '*'
max_connections = 200
shared_buffers = 256MB
work_mem = 4MB
maintenance_work_mem = 64MB
effective_cache_size = 1GB
random_page_cost = 1.1
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100
EOF

bastille cmd database "cat > /var/db/postgres/data16/pg_hba.conf" << 'EOF'
local all all trust
host all all 10.17.89.0/24 md5
host all all 127.0.0.1/32 md5
EOF

# Start PostgreSQL
bastille service database postgresql start

# Create application database
bastille cmd database "su - postgres -c 'createdb saasapp'"
bastille cmd database "su - postgres -c \"psql -c \\\"CREATE USER appuser WITH PASSWORD 'secure_password';\\\"\""
bastille cmd database "su - postgres -c \"psql -c \\\"GRANT ALL PRIVILEGES ON DATABASE saasapp TO appuser;\\\"\""
```

### Step 6: Create Cache Jail

```bash
# Create Redis jail with FreeBSD 14
bastille create -V cache 14.2-RELEASE 10.17.89.31

# Install Redis 7 (latest stable)
bastille pkg cache install redis7

# Configure Redis 7 with enhanced settings
bastille cmd cache "cat > /usr/local/etc/redis.conf" << 'EOF'
bind 0.0.0.0
port 6379
maxmemory 512mb
maxmemory-policy allkeys-lru
save 900 1
save 300 10
save 60 10000
# Redis 7 features
enable-protected-configs no
enable-debug-command no
enable-module-command no
EOF

# Enable and start Redis
bastille sysrc cache redis_enable=YES
bastille service cache redis start
```

### Step 7: Create Monitoring Jail

```bash
# Create monitoring jail with FreeBSD 14
bastille create -V monitoring 14.2-RELEASE 10.17.89.40

# Install latest Prometheus and Grafana
bastille pkg monitoring install prometheus grafana10

# Configure Prometheus
bastille cmd monitoring "cat > /usr/local/etc/prometheus.yml" << 'EOF'
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'haproxy'
    static_configs:
      - targets: ['10.17.89.10:8404']

  - job_name: 'web-servers'
    static_configs:
      - targets: ['10.17.89.11:9113', '10.17.89.12:9113', '10.17.89.13:9113']

  - job_name: 'app-servers'
    static_configs:
      - targets: ['10.17.89.21:3000', '10.17.89.22:3000', '10.17.89.23:3000']
EOF

# Enable services
bastille sysrc monitoring prometheus_enable=YES
bastille sysrc monitoring grafana_enable=YES

# Start services
bastille service monitoring prometheus start
bastille service monitoring grafana start
```

### Step 8: Management Scripts

Create management scripts for easy operations:

**start-production.sh**:
```bash
#!/bin/sh
# Start all production jails in order

echo "Starting production environment..."

# Start core infrastructure
bastille start database
bastille start cache
sleep 5

# Start application servers
bastille start app1 app2 app3
sleep 10

# Start web servers
bastille start web1 web2 web3
sleep 5

# Start load balancer
bastille start loadbalancer

# Start monitoring
bastille start monitoring

echo "Production environment started successfully!"
```

**stop-production.sh**:
```bash
#!/bin/sh
# Stop all production jails

echo "Stopping production environment..."

bastille stop loadbalancer
bastille stop web1 web2 web3
bastille stop app1 app2 app3
bastille stop cache
bastille stop database
bastille stop monitoring

echo "Production environment stopped."
```

**health-check.sh**:
```bash
#!/bin/sh
# Health check script for all services

echo "=== Production Health Check ==="

# Check jail status
echo "Jail Status:"
bastille list | grep -E "(loadbalancer|web[1-3]|app[1-3]|database|cache|monitoring)"

# Check service endpoints
echo -e "\nService Health:"
curl -s http://10.17.89.10/health || echo "Load Balancer: FAILED"
curl -s http://10.17.89.21:3000/api/health || echo "App Server 1: FAILED"
curl -s http://10.17.89.22:3000/api/health || echo "App Server 2: FAILED"
curl -s http://10.17.89.23:3000/api/health || echo "App Server 3: FAILED"

# Check database connection
echo -e "\nDatabase Status:"
bastille cmd database "pg_isready -h localhost -p 5432" || echo "Database: FAILED"

# Check Redis
echo -e "\nCache Status:"
bastille cmd cache "redis-cli ping" || echo "Redis: FAILED"

echo "=== Health Check Complete ==="
```

### Step 9: Backup and Recovery

**backup-production.sh**:
```bash
#!/bin/sh
# Backup script for production environment

BACKUP_DIR="/backups/$(date +%Y-%m-%d)"
mkdir -p $BACKUP_DIR

echo "Creating production backup in $BACKUP_DIR..."

# Backup database
bastille cmd database "pg_dump saasapp" > $BACKUP_DIR/database-backup.sql

# Backup jail snapshots
bastille snapshot database database-$(date +%Y%m%d)
bastille snapshot cache cache-$(date +%Y%m%d)

# Backup configurations
cp /usr/local/bastille/jails/*/root/usr/local/etc/ $BACKUP_DIR/configs/ -r

echo "Backup completed: $BACKUP_DIR"
```

### Step 10: Scaling Operations

**scale-app-servers.sh**:
```bash
#!/bin/sh
# Scale application servers up or down

OPERATION=$1  # "up" or "down"
COUNT=${2:-1}

if [ "$OPERATION" = "up" ]; then
    echo "Scaling up $COUNT application servers..."
    NEXT_ID=$(bastille list | grep app | wc -l | awk '{print $1+1}')

    for i in $(seq $NEXT_ID $((NEXT_ID + COUNT - 1))); do
        IP="10.17.89.2$i"
        bastille create -V app$i 14.2-RELEASE $IP
        bastille template app$i nodejs-app
        bastille cmd app$i "cd /opt/app && npm install"
        bastille start app$i

        # Update load balancer configuration
        echo "    server app$i $IP:3000 check" >> /usr/local/bastille/jails/loadbalancer/root/usr/local/etc/haproxy.conf
    done

    bastille service loadbalancer haproxy reload

elif [ "$OPERATION" = "down" ]; then
    echo "Scaling down $COUNT application servers..."
    # Implementation for scaling down
fi
```

## Production Best Practices

### Security Hardening

1. **Jail Isolation**: Use proper resource limits and security levels
2. **Network Segmentation**: Implement proper firewall rules
3. **User Management**: Use dedicated service users in jails
4. **SSL/TLS**: Implement proper certificate management

### Performance Optimization

1. **Resource Allocation**: Set appropriate CPU and memory limits
2. **ZFS Tuning**: Optimize ZFS parameters for workload
3. **Network Tuning**: Configure network buffer sizes
4. **Application Tuning**: Optimize application-specific parameters

### Monitoring and Alerting

1. **Metrics Collection**: Use Prometheus for comprehensive monitoring
2. **Log Aggregation**: Centralize logs from all jails
3. **Alerting Rules**: Set up alerts for critical conditions
4. **Performance baselines**: Establish baseline metrics

### Disaster Recovery

1. **Regular Backups**: Automated backup procedures
2. **Recovery Testing**: Regular disaster recovery drills
3. **Documentation**: Maintain up-to-date runbooks
4. **Redundancy**: Plan for hardware failure scenarios

## Modern Container Orchestration with Bastille

### Service Discovery and Load Balancing

FreeBSD 14 with Bastille supports modern service discovery patterns:

```bash
# Create service discovery jail
bastille create -V consul 14.2-RELEASE 10.17.89.50
bastille pkg consul install consul

# Configure Consul for service discovery
bastille cmd consul "cat > /usr/local/etc/consul.d/consul.hcl" << 'EOF'
datacenter = "dc1"
data_dir = "/var/db/consul"
log_level = "INFO"
server = true
bootstrap_expect = 1
bind_addr = "10.17.89.50"
client_addr = "0.0.0.0"
ui_config {
  enabled = true
}
EOF
```

### Container Health Monitoring

Implement comprehensive health monitoring:

```bash
# Health monitoring with modern tools
bastille pkg monitoring install node_exporter prometheus-alertmanager

# Configure Prometheus with service discovery
bastille cmd monitoring "cat >> /usr/local/etc/prometheus.yml" << 'EOF'
  - job_name: 'consul'
    consul_sd_configs:
      - server: '10.17.89.50:8500'
        services: ['web', 'app', 'database']
    relabel_configs:
      - source_labels: [__meta_consul_service]
        target_label: job
EOF
```

### GitOps Integration

Modern deployment patterns with FreeBSD 14:

```bash
# Create GitOps deployment jail
bastille create -V gitops 14.2-RELEASE 10.17.89.60
bastille pkg gitops install git drone-cli

# Automated deployment script
cat > /usr/local/bin/deploy-app.sh << 'EOF'
#!/bin/sh
# GitOps deployment for jail applications

REPO_URL=$1
APP_NAME=$2
VERSION=$3

# Clone or update repository
if [ -d "/opt/repos/$APP_NAME" ]; then
    cd "/opt/repos/$APP_NAME" && git pull
else
    git clone "$REPO_URL" "/opt/repos/$APP_NAME"
fi

# Deploy to jail
bastille template "$APP_NAME" "/opt/repos/$APP_NAME/bastille-template"
bastille restart "$APP_NAME"

# Health check
sleep 10
curl -f "http://10.17.89.21:3000/health" || exit 1
EOF
```

## Conclusion

FreeBSD 14 Jails with Bastille provide a robust, secure, and efficient platform for running production SaaS applications. The combination offers enterprise-grade containerization with the stability and security that FreeBSD is known for, enhanced by the latest improvements in FreeBSD 14.

This production lab demonstrates how to architect a scalable, multi-tier application using jails for:
- Load balancing and SSL termination
- Horizontal scaling of web and application tiers
- Secure database and caching layers
- Comprehensive monitoring and logging
- Automated backup and recovery procedures

The jail-based architecture provides excellent isolation, security, and resource efficiency while maintaining the flexibility to scale individual components as needed. With proper monitoring, backup procedures, and automation scripts, this setup can serve as the foundation for a robust production SaaS environment.

Remember to regularly update your jails, monitor performance metrics, and test your disaster recovery procedures to ensure your production environment remains secure and reliable.
