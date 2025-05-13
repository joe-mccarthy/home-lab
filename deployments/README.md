# Deployments

The `deployments` directory contains playbooks and associated files for deploying various services within the home lab cluster. Each deployment is designed to be standalone, meaning they can be deployed independently of one another. However, it is assumed that [Traefik](traefik/README.md) has been or will be deployed, as all deployments rely on Traefik for proxying and certificate management.

## Overview

This directory includes deployments for a variety of services, ranging from personal applications to essential cluster management tools. These deployments are designed to:
- Simplify the process of deploying and managing services in a Docker Swarm environment.
- Provide examples of best practices for deploying containerized applications.
- Ensure services are configured with proper proxying, DNS resolution, and HTTPS certificates.

## Deployments

### 1. [Personal Blog](blog/README.md)
- **Description**: Deploys multiple instances of a private Docker image for a personal blog. This deployment demonstrates how to handle private registries and update services when new image versions become available.
- **Use Case**: Ideal for hosting a personal website or blog with high availability.
- **Dependencies**: Requires Traefik for proxying and certificate management.

### 2. [Cioban](cioban/README.md)
- **Description**: Automates the process of updating Docker services when new image versions are available. It monitors services with specific labels and updates them according to their deployment policies.
- **Use Case**: Ensures services remain up-to-date with minimal manual intervention.
- **Dependencies**: Requires access to the Docker socket and Traefik for proxying.

### 3. [Core Deployments](core-deployments/README.md)
- **Description**: A collection of essential services required for the rest of the deployments to function properly. This includes Traefik, Portainer, and Dynamic DNS.
- **Use Case**: Deploy this playbook first to set up the foundational services for the cluster.
- **Dependencies**: None, but it is recommended to run this playbook before deploying other services.

### 4. [DDNS](ddns/README.md)
- **Description**: Dynamically updates DNS records to reflect the public IP address of the home lab's internet gateway. This ensures services are accessible via domain names.
- **Use Case**: Useful for home labs with dynamic IP addresses.
- **Dependencies**: Requires a DNS provider that supports API-based updates (e.g., Cloudflare).

### 5. [Gitea](gitea/README.md)
- **Description**: Deploys a self-hosted Git service for managing repositories, similar to GitHub or GitLab.
- **Use Case**: Ideal for developers who want to host their own version control system.
- **Dependencies**: Requires Traefik for proxying and certificate management.

### 6. [Home Assistant](home-assistant/README.md)
- **Description**: Deploys Home Assistant, an open-source platform for home automation. It integrates with various smart devices to provide a centralized control interface.
- **Use Case**: Perfect for managing and automating smart home devices.
- **Dependencies**: Requires Traefik for proxying and certificate management.

### 7. [NFS Backup](nfs_backup/README.md)
- **Description**: Provides enterprise-grade automated backups for Docker Swarm NFS shared volumes using Restic with S3 storage backend. Implements a comprehensive backup strategy with three coordinated services:
  - **backup**: Creates hourly encrypted incremental backups
  - **prune**: Manages retention policies (keeping 24 latest, 7 daily, 4 weekly, and 4 monthly backups)
  - **check**: Performs regular integrity verification of the backup repository
- **Use Case**: Critical data protection for containerized applications with secure off-site storage
- **Security**: All backups are strongly encrypted and services operate with least-privilege principles
- **Dependencies**: Requires NFS volumes mounted at `/exports/docker` and S3-compatible storage credentials

### 8. [Portainer](portainer/README.md)
- **Description**: Provides a web-based interface for managing Docker and Docker Swarm. While deployments are managed by Ansible, Portainer offers a convenient UI for monitoring and manual management.
- **Use Case**: Useful for visualizing and managing the cluster's status and activity.
- **Dependencies**: Requires Traefik for proxying and certificate management.

### 9. [Traefik](traefik/README.md)
- **Description**: Acts as a reverse proxy for other services, enabling name resolution instead of relying on IP addresses and ports. It also integrates with DNS providers to issue valid HTTPS certificates.
- **Use Case**: A critical component for managing traffic and securing connections in the cluster.
- **Dependencies**: None, but it is recommended to deploy Traefik first.

## Prerequisites

Before deploying any services, ensure the following:
1. **Docker Swarm**:
   - The cluster must be initialized as a Docker Swarm.
   - Nodes should be joined to the Swarm and properly configured.

2. **Traefik Deployment**:
   - Deploy Traefik first to handle proxying and certificate management for other services.

3. **Ansible Inventory**:
   - Ensure the inventory file defines all nodes in the cluster and groups them appropriately.

4. **DNS Configuration**:
   - Set up DNS records for the services you plan to deploy. Use Dynamic DNS if your public IP address changes frequently.

## Usage

To deploy a service, navigate to its directory and run the associated playbook. For example, to deploy the personal blog:

```bash
ansible-playbook blog/deploy.yml 
```