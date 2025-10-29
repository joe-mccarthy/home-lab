# Gitea Deployment

This project provides an automated solution for deploying and managing a **Gitea** instance and its associated **CI/CD runners** using Ansible. Gitea is a lightweight, self-hosted Git service that allows teams to manage repositories, collaborate on code, and track issues. The deployment includes both the main Gitea server and optional runners for automated CI/CD workflows.

## Overview

The deployment uses Ansible playbooks and roles to automate the setup and configuration of:
1. **Gitea Server** (`deploy.yml`) - The main Git service in a Docker Swarm environment
2. **Gitea Runners** (`deploy-workers.yml`) - CI/CD workers using act_runner for automated workflows

Both deployments include tasks for preparing the environment, managing persistent storage, and integrating with reverse proxies for secure access.

## Features

- **Self-Hosted Git Service**: Deploys a fully functional Gitea instance for managing Git repositories.
- **CI/CD Runners**: Optional deployment of Gitea act_runner services for automated workflows and builds.
- **Docker Swarm Integration**: Ensures high availability and scalability by deploying services in a Swarm cluster.
- **Secure Secrets Management**: Uses Ansible Vault to securely store sensitive data like tokens and credentials.
- **Persistent Storage**: Configures NFS directories to ensure data persistence for repositories, logs, and configuration files.
- **Reverse Proxy Integration**: Supports integration with reverse proxies like Traefik for HTTPS and domain-based routing.

## Deployment Options

### Main Gitea Server
```bash
ansible-playbook -i inventory deploy.yml
```
This deploys the main Gitea service with web interface and Git functionality.

### Gitea CI/CD Runners
```bash
ansible-playbook -i inventory deploy-workers.yml
```
This deploys Gitea runners (act_runner) that connect to your Gitea instance to provide CI/CD capabilities.

## Prerequisites

To deploy Gitea, you need:
- A Docker Swarm cluster
- Ansible installed on the control machine
- Access to an NFS server for persistent storage
- Configuration values and secrets defined and encrypted using Ansible Vault
- For runners: A running Gitea instance and valid runner registration token

## Deployment Process

### Main Gitea Server
The main deployment process involves preparing the environment, ensuring the required directory structure exists, checking NFS connectivity, and deploying the Gitea server stack using Docker Swarm.

### Gitea Runners  
The runner deployment process sets up act_runner instances that connect to your Gitea server to provide CI/CD capabilities. Each runner is deployed globally across the swarm nodes and requires proper registration tokens.

The Ansible playbooks automate these tasks, ensuring consistency and reliability across both deployments.

## Configuration

### Required Variables
- `general.domain`: Your domain name for the Gitea instance
- `gitea.runner_token`: Registration token for connecting runners to Gitea (required for worker deployment)

### Group Variables
Configuration is managed through `group_vars/all.yml` which should contain encrypted secrets and deployment-specific settings.

## Security Considerations

The deployment emphasizes security by:
- Encrypting sensitive data like registration tokens using Ansible Vault
- Restricting permissions on configuration files  
- Integrating with reverse proxies for secure HTTPS access
- Using proper Docker security practices for both server and runner deployments

It is recommended to test both deployments in a staging environment before applying to production.

## Troubleshooting

### Common Issues
- **Missing directories**: Verify NFS connectivity and directory structure
- **Uninitialized Docker Swarm**: Ensure swarm mode is enabled on target nodes
- **Misconfigured secrets**: Check Ansible Vault encryption and variable definitions
- **Runner connection issues**: Verify registration token and Gitea instance URL
- **Service deployment failures**: Review Docker Swarm logs and container status

These issues can be resolved by verifying the environment setup and reviewing Ansible playbook logs.