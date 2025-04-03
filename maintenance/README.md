# Maintenance

The `maintenance` directory contains playbooks and associated files designed to perform routine maintenance tasks on the home lab cluster. These tasks ensure the cluster remains secure, up-to-date, and operational. By automating these processes, the maintenance playbooks reduce manual effort, minimize downtime, and improve the overall reliability of the cluster.

## Rationale

Maintaining a home lab cluster involves repetitive and time-sensitive tasks such as updating packages, managing credentials, and ensuring system health. Performing these tasks manually can lead to inconsistencies, missed updates, and potential security vulnerabilities. The maintenance playbooks in this directory provide a standardized and automated approach to managing the cluster, ensuring that all nodes are configured and maintained uniformly.

## Playbooks

### 1. `update.yml`
This playbook is responsible for:
- Updating and upgrading system packages on all nodes in the cluster.
- Checking if a reboot is required after updates and rebooting the system if necessary.

**Use Case**: Run this playbook regularly to ensure all nodes are running the latest software and security patches.

For more details, refer to the [update.yml file](./update.yml).

### 2. `docker-login/login.yml`
This playbook handles:
- Logging into a private Docker registry on all nodes in the cluster.
- Using credentials securely stored in an Ansible Vault to authenticate with the registry.

**Use Case**: Run this playbook before deploying services that require pulling private Docker images.

For more details, refer to the [docker-login README](./docker-login/README.md).

### 3. `set-up-machine/setup.yml`
This playbook automates the initial configuration of new machines in the cluster. It:
- Configures SSH access for passwordless login.
- Updates and upgrades system packages.
- Installs essential packages required for system monitoring and management.
- Reboots the machine if necessary.

**Use Case**: Run this playbook when adding new machines to the cluster to ensure they meet the cluster's configuration standards.

For more details, refer to the [set-up-machine README](./set-up-machine/README.md).

### 4. `shutdown-cluster.yml`
This playbook gracefully shuts down all nodes in the cluster. It:
- Sends a shutdown signal to all nodes in the `cluster` group.
- Delays the shutdown to allow any critical processes to complete.

**Use Case**: Run this playbook when performing hardware maintenance or powering down the cluster.

For more details, refer to the [shutdown-cluster.yml file](./shutdown-cluster.yml).

## Prerequisites

Before running any maintenance playbooks, ensure the following:
1. **Ansible Inventory**:
   - The inventory file must define all nodes in the cluster and group them appropriately (e.g., `all`, `cluster`).

2. **SSH Access**:
   - Ensure passwordless SSH access is configured for all nodes, or be prepared to provide the SSH password using the `--ask-pass` option.

3. **Sudo Privileges**:
   - Verify that the user running the playbooks has sudo privileges on the target machines. Use the `--ask-become-pass` option if required.

4. **Ansible Vault**:
   - Sensitive information such as Docker registry credentials must be securely stored in an Ansible Vault.

## Usage

To execute a playbook, use the following command:

```bash
ansible-playbook <playbook-name>.yml --ask-pass --ask-become-pass
```