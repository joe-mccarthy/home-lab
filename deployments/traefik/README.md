# Traefik Deployment

[![Ansible](https://img.shields.io/badge/Ansible-Automation-EE0000?logo=ansible&logoColor=white&style=flat-square)](https://docs.ansible.com/) [![Docker Swarm](https://img.shields.io/badge/Docker%20Swarm-Ingress-2496ED?logo=docker&logoColor=white&style=flat-square)](https://docs.docker.com/engine/swarm/) [![Let's Encrypt](https://img.shields.io/badge/Let%27s%20Encrypt-TLS-003A70?logo=letsencrypt&logoColor=white&style=flat-square)](https://letsencrypt.org/docs/) [![Cloudflare](https://img.shields.io/badge/Cloudflare-DNS%20Challenge-F38020?logo=cloudflare&logoColor=white&style=flat-square)](https://developers.cloudflare.com/dns/) ![Traefik](https://img.shields.io/badge/Traefik-v3.6.11-5C5C5C?style=flat-square)

This project provides an automated solution for deploying and managing **Traefik** in a Docker Swarm environment using Ansible. Traefik is a modern reverse proxy and load balancer that simplifies the management of microservices by dynamically routing traffic to services based on their configuration.

## Overview

The deployment uses Ansible playbooks and roles to automate the setup and configuration of Traefik in a Docker Swarm cluster. It includes tasks for preparing the environment, managing persistent storage, and securely configuring Traefik for dynamic routing and HTTPS support.

## Features

- **Dynamic Reverse Proxy**: Automatically discovers and routes traffic to services based on Docker labels.
- **Docker Swarm Integration**: Deploys Traefik as a service in a Swarm cluster for high availability and scalability.
- **Secure Secrets Management**: Uses Ansible Vault to securely store sensitive data like API tokens and credentials.
- **Let's Encrypt Integration**: Automates the issuance and renewal of SSL/TLS certificates using DNS-based challenges.
- **Persistent Storage**: Configures NFS directories to ensure data persistence for certificates and logs.
- **Comprehensive Logging**: Supports access and error logging for monitoring and debugging.

## Prerequisites

To deploy Traefik, you need a Docker Swarm cluster, Ansible installed on the control machine, and access to an NFS server for persistent storage. Configuration values and secrets must be defined and encrypted using Ansible Vault. A DNS provider (e.g., Cloudflare) is required for managing DNS-based challenges for Let's Encrypt.

## Deployment Process

The deployment process involves preparing the environment, ensuring the required directory structure exists, and deploying the Traefik stack using Docker Swarm. The Ansible playbooks automate these tasks, ensuring consistency and reliability. Traefik is configured to dynamically route traffic to services based on Docker labels and to handle HTTPS traffic using Let's Encrypt.

## Security Considerations

The deployment emphasizes security by encrypting sensitive data, restricting permissions on configuration files, and automating the issuance of SSL/TLS certificates. It is recommended to test the deployment in a staging environment before applying it to production.

The Traefik dashboard and API are not published directly on the host. The insecure API listener is disabled, API debug endpoints are disabled, and dashboard access is routed through `https://traefik.<domain>` with a Traefik IP allowlist middleware. Configure trusted dashboard source ranges in `vault.services.traefik.dashboard.allowed_ips` as a comma-separated list. Set `vault.services.traefik.dashboard.enabled` to `false` to disable the dashboard route entirely.

## Troubleshooting

Common issues such as missing directories, uninitialized Docker Swarm, or misconfigured secrets can be resolved by verifying the environment setup and reviewing Ansible playbook logs.
