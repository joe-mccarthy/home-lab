# Gitea Deployment

This project provides an automated solution for deploying and managing a **Gitea** instance using Ansible. Gitea is a lightweight, self-hosted Git service that allows teams to manage repositories, collaborate on code, and track issues. The deployment is designed to ensure reliability, scalability, and secure integration with external services.

## Overview

The deployment uses Ansible playbooks and roles to automate the setup and configuration of Gitea in a Docker Swarm environment. It includes tasks for preparing the environment, managing persistent storage, and integrating with reverse proxies for secure access.

## Features

- **Self-Hosted Git Service**: Deploys a fully functional Gitea instance for managing Git repositories.
- **Docker Swarm Integration**: Ensures high availability and scalability by deploying Gitea as a service in a Swarm cluster.
- **Secure Secrets Management**: Uses Ansible Vault to securely store sensitive data like user IDs and group IDs.
- **Persistent Storage**: Configures NFS directories to ensure data persistence for repositories, logs, and configuration files.
- **Reverse Proxy Integration**: Supports integration with reverse proxies like Traefik for HTTPS and domain-based routing.

## Prerequisites

To deploy Gitea, you need a Docker Swarm cluster, Ansible installed on the control machine, and access to an NFS server for persistent storage. Configuration values and secrets must be defined and encrypted using Ansible Vault.

## Deployment Process

The deployment process involves preparing the environment, ensuring the required directory structure exists, and deploying the Gitea stack using Docker Swarm. The Ansible playbooks automate these tasks, ensuring consistency and reliability.

## Security Considerations

The deployment emphasizes security by encrypting sensitive data, restricting permissions on configuration files, and integrating with reverse proxies for secure HTTPS access. It is recommended to test the deployment in a staging environment before applying it to production.

## Troubleshooting

Common issues such as missing directories, uninitialized Docker Swarm, or misconfigured secrets can be resolved by verifying the environment setup and reviewing Ansible playbook logs.