# Home Assistant Smart Home Stack Deployment

This project provides a comprehensive, production-ready solution for deploying **Home Assistant** with integrated smart home services using Ansible and Docker Swarm. The deployment creates a complete smart home automation platform with Zigbee device support, MQTT communication, and secure external access.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Deployment](#deployment)
- [Post-Deployment](#post-deployment)
- [Management](#management)
- [Troubleshooting](#troubleshooting)
- [Security](#security)
- [Backup & Recovery](#backup--recovery)
- [Contributing](#contributing)

## 🏠 Overview

This deployment solution creates a complete smart home automation stack consisting of:

- **Home Assistant Core**: Primary automation platform with web interface
- **Zigbee2MQTT**: Bridge for Zigbee smart devices (lights, sensors, switches)
- **Mosquitto MQTT Broker**: Communication backbone for IoT devices
- **Traefik Integration**: Reverse proxy with automatic SSL certificates
- **NFS Storage**: Persistent, shared storage for configurations and data

The deployment is designed for high availability, security, and ease of management in a Docker Swarm environment.

## 🏗️ Architecture

### System Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Internet      │    │   Docker Swarm   │    │   NFS Storage   │
│                 │    │                  │    │                 │
│  ┌───────────┐  │    │ ┌──────────────┐ │    │ ┌─────────────┐ │
│  │  Users    │  │    │ │   Traefik    │ │    │ │   Config    │ │
│  │  Devices  │  │    │ │  (Proxy)     │ │    │ │   Database  │ │
│  └───────────┘  │    │ └──────┬───────┘ │    │ │   Logs      │ │
│                 │    │        │         │    │ └─────────────┘ │
└─────────────────┘    │ ┌──────▼───────┐ │    └─────────────────┘
                       │ │ Home         │ │            │
┌─────────────────┐    │ │ Assistant    │◄├────────────┘
│  Zigbee Devices │    │ └──────┬───────┘ │
│                 │    │        │         │
│ ┌─────┐ ┌─────┐ │    │ ┌──────▼───────┐ │
│ │Light│ │Sensor│ │    │ │ Zigbee2MQTT  │ │
│ └─────┘ └─────┘ │    │ └──────┬───────┘ │
│                 │    │        │         │
└─────────────────┘    │ ┌──────▼───────┐ │
                       │ │ MQTT Broker  │ │
                       │ │ (Mosquitto)  │ │
                       │ └──────────────┘ │
                       └──────────────────┘
```

### Network Design

- **Proxy Network**: External network for Traefik reverse proxy
- **Default Network**: Internal service communication
- **Host Network**: Direct hardware access for USB devices

### Data Flow

1. **External Access**: Users → Traefik → Home Assistant (HTTPS)
2. **Device Communication**: Zigbee Devices → Zigbee2MQTT → MQTT → Home Assistant
3. **Data Persistence**: All services → NFS Storage

## ✨ Features

### 🏠 Home Automation
- **Complete Home Assistant Installation**: Latest stable version with all integrations
- **Zigbee Device Support**: Native support for 1000+ Zigbee devices
- **MQTT Integration**: Reliable IoT device communication
- **Web Interface**: Modern, responsive web UI for device management

### 🔧 Infrastructure
- **Docker Swarm Deployment**: High availability and container orchestration
- **NFS Persistent Storage**: Shared storage across cluster nodes
- **Automatic SSL Certificates**: Let's Encrypt integration via Traefik
- **Zero-Downtime Updates**: Rolling updates with Docker Swarm

### 🔐 Security
- **Ansible Vault Integration**: Encrypted secrets management
- **HTTPS Enforcement**: Automatic HTTP to HTTPS redirection
- **Network Isolation**: Segregated internal and external networks
- **Container Security**: Minimal privileges and security contexts

### 🛠️ Operations
- **Infrastructure as Code**: Fully automated deployment
- **Configuration Management**: Template-based configuration
- **Health Monitoring**: Built-in service health checks
- **Backup Integration**: NFS-based backup strategy

## 📋 Prerequisites

### Infrastructure Requirements

- **Docker Swarm Cluster**: Initialized and operational
  ```bash
  docker swarm init
  ```

- **NFS Server**: Accessible storage for persistent data
  ```bash
  # Mount point must exist: /mnt/nfs/docker
  ```

- **Ansible Control Machine**: With required collections
  ```bash
  ansible-galaxy collection install community.docker
  ansible-galaxy collection install ansible.posix
  ```

### Hardware Requirements

- **Zigbee Coordinator**: USB dongle for Zigbee communication
  - Recommended: ConBee II, Sonoff Zigbee 3.0, CC2652R
  - Connected to Docker Swarm node running Zigbee2MQTT

- **System Resources** (per node):
  - RAM: 2GB minimum, 4GB recommended
  - Storage: 10GB minimum for system, NFS for data
  - CPU: 2 cores recommended

### Network Requirements

- **Domain Name**: For external access (e.g., `yourdomain.com`)
- **DNS Configuration**: Subdomains pointing to Traefik
  - `homeassistant.yourdomain.com`
  - `zigbee2mqtt.yourdomain.com`
- **Firewall Rules**: Ports 80, 443, 1883 (MQTT) accessible

## 🚀 Quick Start

### 1. Clone and Navigate
```bash
git clone <repository-url>
cd deployments/home-assistant
```

### 2. Configure Secrets
```bash
# Create and encrypt secrets
ansible-vault create group_vars/all.yml
```

Add the following encrypted variables:
```yaml
---
vault_proxy: "https://homeassistant.yourdomain.com"
vault_mqtt_server: "192.168.1.100"
vault_zigbee_serial_port: "/dev/ttyUSB0"
```

### 3. Update Inventory
```bash
# Edit your inventory file
vim inventory.yml
```

```yaml
all:
  hosts:
    swarm-node-1:
      ansible_host: 192.168.1.10
    swarm-node-2:
      ansible_host: 192.168.1.11
```

### 4. Deploy
```bash
# Run the deployment playbook
ansible-playbook -i inventory.yml deploy.yml --ask-vault-pass
```

### 5. Verify Deployment
```bash
# Check service status
docker service ls
docker service logs homeassistant_homeassistant
```

## ⚙️ Configuration

### Ansible Variables

The deployment uses several configuration files:

#### `group_vars/all.yml` (Encrypted)
```yaml
# Reverse proxy configuration
vault_proxy: "https://homeassistant.yourdomain.com"

# MQTT broker address
vault_mqtt_server: "192.168.1.100"

# Zigbee coordinator device path
vault_zigbee_serial_port: "/dev/ttyUSB0"
```

#### `deploy.yml` (Main Playbook)
The main deployment playbook that orchestrates the entire process:
- Environment preparation
- NFS directory structure creation
- Docker Swarm service deployment
- Configuration template deployment

### Service Configuration

#### Home Assistant (`templates/configuration.yaml`)
- Integration configurations
- Device discovery settings
- Database configuration
- Logging preferences

#### Zigbee2MQTT (`templates/zigbee/configuration.yaml`)
- MQTT broker connection
- Zigbee coordinator settings
- Device pairing configuration
- Network security settings

#### Docker Compose (`templates/docker-compose.yaml`)
- Service definitions
- Volume mappings
- Network configuration
- Traefik labels

## 🚀 Deployment

### Standard Deployment Process

1. **Environment Preparation**
   ```bash
   # Verify NFS mount
   ls -la /mnt/nfs/docker/
   
   # Check Docker Swarm status
   docker node ls
   ```

2. **Run Deployment**
   ```bash
   ansible-playbook -i inventory.yml deploy.yml --ask-vault-pass
   ```

3. **Monitor Progress**
   ```bash
   # Watch service deployment
   watch docker service ls
   
   # Check specific service logs
   docker service logs -f homeassistant_homeassistant
   ```

### Advanced Deployment Options

#### Selective Service Deployment
```bash
# Deploy only specific roles
ansible-playbook deploy.yml --tags "nfs,config"
```

#### Dry Run
```bash
# Check what would be changed
ansible-playbook deploy.yml --check --diff
```

#### Verbose Output
```bash
# Detailed deployment logs
ansible-playbook deploy.yml -vvv
```

## 📊 Post-Deployment

### Service Verification

1. **Check Service Status**
   ```bash
   docker service ls
   docker service ps homeassistant_homeassistant
   docker service ps homeassistant_zigbee2mqtt
   docker service ps homeassistant_mqtt
   ```

2. **Access Web Interfaces**
   - Home Assistant: `https://homeassistant.yourdomain.com`
   - Zigbee2MQTT: `https://zigbee2mqtt.yourdomain.com`

3. **Test MQTT Connectivity**
   ```bash
   # Install MQTT client
   sudo apt install mosquitto-clients
   
   # Test MQTT broker
   mosquitto_pub -h <mqtt-server> -t test/topic -m "Hello World"
   mosquitto_sub -h <mqtt-server> -t test/topic
   ```

### Initial Configuration

1. **Home Assistant Setup**
   - Access web interface
   - Complete initial setup wizard
   - Configure integrations
   - Add MQTT broker integration

2. **Zigbee2MQTT Setup**
   - Access web interface
   - Verify coordinator connection
   - Enable device pairing
   - Add your first Zigbee device

3. **MQTT Integration**
   - Configure Home Assistant MQTT integration
   - Verify device discovery
   - Test device control

## 🔧 Management

### Service Management

```bash
# Scale services (if supported)
docker service scale homeassistant_homeassistant=1

# Update service image
docker service update --image ghcr.io/home-assistant/home-assistant:2024.1.0 homeassistant_homeassistant

# Remove service
docker service rm homeassistant_homeassistant

# Redeploy entire stack
docker stack deploy -c docker-compose.yaml homeassistant
```

### Configuration Updates

```bash
# Update configuration
ansible-playbook deploy.yml --tags "config"

# Restart services after config changes
docker service update --force homeassistant_homeassistant
```

### Log Management

```bash
# View service logs
docker service logs homeassistant_homeassistant
docker service logs homeassistant_zigbee2mqtt
docker service logs homeassistant_mqtt

# Follow logs in real-time
docker service logs -f homeassistant_homeassistant

# Export logs
docker service logs homeassistant_homeassistant > homeassistant.log
```

## 🔍 Troubleshooting

### Common Issues

#### Service Won't Start
```bash
# Check service status
docker service ps homeassistant_homeassistant --no-trunc

# Check node resources
docker node ls
docker system df

# Verify network
docker network ls
docker network inspect proxy
```

#### Database Issues
```bash
# Check NFS mount
df -h /mnt/nfs/docker
ls -la /mnt/nfs/docker/home_assistant/

# Check permissions
sudo chown -R nobody:nogroup /mnt/nfs/docker/home_assistant/
```

#### Zigbee Coordinator Not Found
```bash
# List USB devices
lsusb
ls -la /dev/ttyUSB*

# Check udev rules
sudo udevadm info -a -p $(udevadm info -q path -n /dev/ttyUSB0)

# Verify device permissions
ls -la /dev/ttyUSB0
```

#### Network Connectivity Issues
```bash
# Test internal network
docker exec -it $(docker ps -q -f name=homeassistant) ping mqtt

# Test external access
curl -I https://homeassistant.yourdomain.com

# Check Traefik configuration
docker service logs traefik_traefik
```

### Debug Commands

```bash
# Enter running container
docker exec -it $(docker ps -q -f name=homeassistant) /bin/bash

# Check container logs in detail
docker service logs homeassistant_homeassistant 2>&1 | grep ERROR

# Verify service discovery
docker service inspect homeassistant_homeassistant
```

## 🔒 Security

### Security Best Practices

1. **Secrets Management**
   - Use Ansible Vault for all sensitive data
   - Rotate vault passwords regularly
   - Never commit unencrypted secrets

2. **Network Security**
   - Use HTTPS for all external access
   - Restrict MQTT broker access
   - Implement firewall rules
   - Network segmentation for IoT devices

3. **Container Security**
   - Regular image updates
   - Minimal container privileges
   - Security scanning of images
   - Resource limits and constraints

4. **Access Control**
   - Strong Home Assistant passwords
   - Enable two-factor authentication
   - Regular access reviews
   - Audit logs monitoring

### Security Hardening

```bash
# Enable MQTT authentication
# Edit mosquitto configuration to require passwords

# Implement network policies
# Use Docker network policies for traffic control

# Regular security updates
ansible-playbook deploy.yml --tags "security"
```

## 💾 Backup & Recovery

### Backup Strategy

1. **NFS Storage Backup**
   ```bash
   # Backup all Home Assistant data
   sudo tar -czf homeassistant-backup-$(date +%Y%m%d).tar.gz \
     -C /mnt/nfs/docker/home_assistant .
   ```

2. **Configuration Backup**
   ```bash
   # Backup Ansible configuration
   tar -czf ansible-config-backup-$(date +%Y%m%d).tar.gz \
     group_vars/ templates/ *.yml
   ```

3. **Zigbee Network Backup**
   ```bash
   # Backup Zigbee coordinator state
   cp /mnt/nfs/docker/home_assistant/zigbee2mqtt/data/coordinator_backup.json \
     zigbee-backup-$(date +%Y%m%d).json
   ```

### Recovery Procedures

1. **Full System Recovery**
   ```bash
   # Restore NFS data
   sudo tar -xzf homeassistant-backup-YYYYMMDD.tar.gz \
     -C /mnt/nfs/docker/home_assistant/
   
   # Redeploy services
   ansible-playbook deploy.yml
   ```

2. **Zigbee Network Recovery**
   ```bash
   # Restore Zigbee coordinator backup
   cp zigbee-backup-YYYYMMDD.json \
     /mnt/nfs/docker/home_assistant/zigbee2mqtt/data/coordinator_backup.json
   
   # Restart Zigbee2MQTT service
   docker service update --force homeassistant_zigbee2mqtt
   ```

### Automated Backup

```bash
# Create backup script
cat > /usr/local/bin/homeassistant-backup.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/backup/homeassistant"
DATE=$(date +%Y%m%d-%H%M%S)

# Create backup directory
mkdir -p "$BACKUP_DIR"

# Backup Home Assistant data
tar -czf "$BACKUP_DIR/homeassistant-$DATE.tar.gz" \
  -C /mnt/nfs/docker/home_assistant .

# Keep only last 30 days of backups
find "$BACKUP_DIR" -name "homeassistant-*.tar.gz" -mtime +30 -delete
EOF

chmod +x /usr/local/bin/homeassistant-backup.sh

# Add to crontab for daily backups
echo "0 2 * * * /usr/local/bin/homeassistant-backup.sh" | sudo crontab -
```

## 🤝 Contributing

We welcome contributions to improve this deployment solution!

### How to Contribute

1. **Fork the Repository**
2. **Create a Feature Branch**
   ```bash
   git checkout -b feature/improvement-name
   ```
3. **Make Changes**
   - Update documentation
   - Improve automation
   - Add new features
   - Fix bugs

4. **Test Changes**
   ```bash
   # Test in staging environment
   ansible-playbook deploy.yml --check
   ```

5. **Submit Pull Request**
   - Clear description of changes
   - Testing evidence
   - Documentation updates

### Development Guidelines

- Follow Ansible best practices
- Document all changes
- Test in isolated environment
- Maintain backward compatibility
- Update version numbers appropriately

---

## 📞 Support

For support and questions:

1. **Check Documentation**: Review this README and inline comments
2. **Search Issues**: Look for similar problems in project issues
3. **Create Issue**: Report bugs or request features
4. **Community Support**: Join Home Assistant community forums

