# Immich Photo Management Deployment

A comprehensive Docker Swarm deployment for Immich, a high-performance self-hosted photo and video management solution. This deployment provides a complete alternative to Google Photos with AI-powered features including face recognition, object detection, and intelligent search capabilities.

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Configuration](#configuration)
- [Deployment](#deployment)
- [Post-Deployment](#post-deployment)
- [Security Considerations](#security-considerations)
- [File Structure](#file-structure)
- [Support and Resources](#support-and-resources)

## Overview

Immich is a modern photo and video management application that offers:

- **Web Interface**: Intuitive browser-based photo organization and viewing
- **Mobile Apps**: Automatic photo backup from iOS and Android devices
- **AI Features**: Face recognition, object detection, and smart search
- **Timeline View**: Chronological organization of your media library
- **Album Management**: Create and share photo albums
- **External Sharing**: Share photos and albums with external users
- **Metadata Extraction**: Automatic EXIF data processing and location mapping

## Architecture

The deployment consists of four interconnected services:

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Traefik       │────│  Immich Server   │────│  PostgreSQL     │
│  (Reverse Proxy)│    │  (Web App/API)   │    │  (Database)     │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                                │                        │
                       ┌─────────────────┐    ┌─────────────────┐
                       │     Redis       │    │  Machine        │
                       │   (Cache)       │    │  Learning       │
                       └─────────────────┘    └─────────────────┘
```

### Services

- **immich-server**: Main application providing web interface and REST API
- **immich-database**: PostgreSQL with vector extensions for AI-powered search
- **immich-redis**: High-performance cache for sessions and temporary data
- **immich-machine-learning**: TensorFlow-based AI processing for smart features

## Prerequisites

### Infrastructure Requirements

- **Docker Swarm**: Active swarm cluster with at least one manager node
- **NFS Storage**: Network file system mounted at `/mnt/nfs/docker`
- **Traefik**: Reverse proxy with Let's Encrypt certificate resolver
- **DNS**: Properly configured domain pointing to your cluster

### System Requirements

- **CPU**: Minimum 2 cores (4+ recommended for AI features)
- **RAM**: Minimum 4GB (8GB+ recommended)
- **Storage**: Depends on photo library size (plan for growth)
- **Network**: Stable connection for NFS and external access

### Software Dependencies

- Ansible 2.9+
- Docker 20.10+
- Docker Compose (for template processing)

## Configuration

### 1. Ansible Vault Setup

Create and encrypt sensitive variables:

```bash
# Create vault file
ansible-vault create group_vars/vault.yml

# Add the following content:
vault_immich_user: "immich_user"
vault_immich_password: "secure_random_password"
```

### 2. Domain Configuration

Update your main variables file to include your domain:

```yaml
# In ../../vars/general.yml
general:
  domain: "yourdomain.com"
```

### 3. Storage Planning

Ensure adequate NFS storage for:

- **Photos/Videos**: Primary storage requirement
- **Database**: Metadata and user data (~1-5% of media size)
- **ML Cache**: AI models and processing data (~2-10GB)
- **Backups**: Database exports and configuration

## Deployment

### Quick Start

```bash
# Navigate to the immich deployment directory
cd /home/joseph/projects/home-lab/deployments/immich

# Deploy the entire stack
ansible-playbook -i ../../inventory deploy.yml --ask-vault-pass
```

### Step-by-Step Deployment

1. **Prepare Environment**
   ```bash
   # Verify Docker Swarm status
   docker node ls
   
   # Check NFS mount
   ls -la /mnt/nfs/docker/
   ```

2. **Run NFS Preparation**
   ```bash
   ansible-playbook -i ../../inventory deploy.yml --tags check_nfs --ask-vault-pass
   ```

3. **Deploy Application**
   ```bash
   ansible-playbook -i ../../inventory deploy.yml --tags deploy --ask-vault-pass
   ```

### Deployment Process

The deployment automatically:

1. Verifies prerequisites (Docker, docker-compose)
2. Creates required NFS directories with proper permissions
3. Stops existing services for clean deployment
4. Processes configuration templates with vault variables
5. Deploys services to Docker Swarm
6. Cleans up temporary files

## Post-Deployment

### Service Verification

```bash
# Check service status
docker service ls | grep immich

# View service logs
docker service logs immich_immich-server
docker service logs immich_immich-database
docker service logs immich_immich-redis
docker service logs immich_immich-machine-learning
```

### Initial Setup

1. **Access Web Interface**
   - Navigate to `https://immich.yourdomain.com`
   - Create the first admin account
   - Configure basic settings

2. **Mobile App Setup**
   - Download Immich mobile app
   - Configure server URL: `https://immich.yourdomain.com`
   - Login with admin credentials
   - Enable automatic backup

3. **AI Features Configuration**
   - Navigate to Administration → Machine Learning
   - Enable face detection and object recognition
   - Configure processing settings based on hardware

### Health Checks

The deployment includes comprehensive health monitoring:

- **Database**: `pg_isready` checks with 30s intervals
- **Redis**: `redis-cli ping` verification
- **Server**: Built-in application health endpoints
- **ML Service**: TensorFlow model availability checks

## Security Considerations

### Network Security

- All external traffic routed through Traefik with HTTPS
- Internal services communicate on encrypted overlay network
- Database and Redis not exposed externally
- Automatic SSL certificate management

### Data Protection

- Database credentials encrypted in Ansible vault
- Sensitive environment variables not logged
- File permissions follow least-privilege principle
- Regular backup procedures documented


## File Structure

```
immich/
├── README.md                          # This comprehensive guide
├── deploy.yml                         # Main deployment playbook
├── group_vars/
│   ├── all.yml                        # Non-sensitive variables
│   └── vault.yml                      # Encrypted sensitive data (not in repo)
├── roles/
│   ├── check_nfs/
│   │   └── tasks/
│   │       └── main.yml               # NFS directory preparation
│   └── deploy/
│       └── tasks/
│           └── main.yml               # Service deployment logic
└── templates/
    └── docker-compose.yaml            # Service definitions template
```

### Key Files

- **`deploy.yml`**: Main entry point for deployment
- **`group_vars/all.yml`**: Configuration variables and vault references
- **`templates/docker-compose.yaml`**: Complete service definitions with Traefik integration
- **`roles/check_nfs/`**: Ensures proper NFS directory structure
- **`roles/deploy/`**: Handles service lifecycle and deployment

## Support and Resources

### Official Documentation

- [Immich Documentation](https://immich.app/docs)
- [Docker Swarm Guide](https://docs.docker.com/engine/swarm/)
- [Traefik Documentation](https://doc.traefik.io/traefik/)

### Community Resources

- [Immich GitHub Repository](https://github.com/immich-app/immich)
- [Docker Community Forums](https://forums.docker.com/)
- [Ansible Documentation](https://docs.ansible.com/)

### Getting Help

1. Check service logs for specific error messages
2. Verify all prerequisites are met
3. Review troubleshooting section above
4. Consult official Immich documentation
5. Search GitHub issues for similar problems


