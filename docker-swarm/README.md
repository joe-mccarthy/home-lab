# Docker Swarm

This repository contains Ansible playbooks and roles to automate the setup, management, and teardown of a Docker Swarm cluster. It is designed to simplify the deployment of containerized applications in a distributed environment, leveraging Docker Swarm's orchestration capabilities.

## Features

- **Automated Setup**: Fully automated playbooks to set up a Docker Swarm cluster, including NFS configuration, Docker installation, Swarm initialization, and network setup.
- **Role-Based Structure**: Modular roles for better reusability and maintainability.
- **Cluster Management**: Add or remove manager and worker nodes dynamically.
- **Service Management**: Start, stop, and clean up Docker services across the cluster.
- **NFS Integration**: Configure and mount NFS volumes for shared storage across nodes.
- **Cluster Teardown**: Safely remove all nodes from the Swarm and clean up Docker resources.

## Prerequisites

Before using this repository, ensure the following:

1. Ensure passwordless SSH access is configured for all nodes in the cluster.

## Playbooks

### 1. **Create Swarm Cluster**

This playbook sets up a Docker Swarm cluster from scratch. It includes:
- Setting up the NFS server and mounting NFS shares on all nodes.
- Installing Docker on all nodes.
- Initializing the Swarm cluster and adding manager and worker nodes.
- Creating a proxy network for service communication.

### 2. **Destroy Swarm Cluster**

This playbook tears down the Docker Swarm cluster. It includes:
- Stopping all running services.
- Draining and removing worker nodes.
- Removing manager nodes in reverse order to avoid quorum issues.
- Cleaning up Docker resources on all nodes.

## Roles

The repository is structured into modular roles for better maintainability:

1. Configures the NFS server and exports shared directories.
2. Mounts NFS shares on all nodes.
3. Installs and configures Docker on all nodes.
4. Initializes the Docker Swarm cluster on the first manager node.
5. Adds additional manager nodes to the Swarm.
6. Adds worker nodes to the Swarm.
7. Assigns labels to nodes for service placement.
8. Creates a Docker overlay network for service communication.
9. Stops all running Docker services in the Swarm.
10. Drains and removes worker nodes from the Swarm.
11. Removes nodes (managers or workers) from the Swarm.
12. Cleans up unused Docker resources (containers, images, volumes, networks).

## Usage

To execute a playbook, use the appropriate command for the desired playbook.

## Best Practices

1. Always back up critical configuration files (e.g., `/etc/exports` for NFS) before running destructive playbooks.
2. Test the playbooks in a non-production environment before deploying to production.
3. Store sensitive information (e.g., SSH keys, Docker registry credentials) securely using Ansible Vault.

## Troubleshooting

- Ensure the Docker service is running on all nodes.
- Verify that the NFS server is reachable and the exports are correctly configured.
- Check the logs on the manager node for detailed error messages.

## Acknowledgments

- Docker Documentation
- Ansible Documentation
- Community Docker Collection