# Docker Login

This directory contains the playbook and associated files required to log into a private Docker registry on all nodes in the cluster. Logging into the registry ensures that the nodes can pull private images for deployments.

## Overview

The `docker-login` playbook is responsible for:
- Authenticating with a private Docker registry.
- Using credentials securely stored in an Ansible Vault.
- Ensuring all nodes in the cluster are authenticated to pull images from the registry.

## Files

### 1. `login.yml`
This is the main playbook that performs the Docker login operation. It:
- Targets all nodes in the `cluster` group.
- Uses the `docker login` command to authenticate with the registry.
- Retrieves credentials (`username`, `password`, and `registry URL`) from the group variables file.

For more details, refer to the [login.yml file](./login.yml).

### 2. `group_vars/all.yml`
This file defines the group variables used by the `docker-login` playbook. It includes:
- `registry_user`: The username for the Docker registry.
- `registry_password`: The password for the Docker registry.
- `registry_url`: The URL of the Docker registry.

These variables are securely stored in an Ansible Vault to protect sensitive information. For more details, refer to the [group_vars/all.yml file](./group_vars/all.yml).

## Prerequisites

Before running the `docker-login` playbook, ensure the following:
1. **Ansible Vault Setup**: The credentials (`vault_registry_user`, `vault_registry_password`, and `vault_registry_url`) must be stored in an Ansible Vault.
   - For more information on using Ansible Vault, refer to the [Ansible Vault documentation](https://docs.ansible.com/ansible/latest/vault_guide/index.html).
2. **Cluster Inventory**: The `cluster` group must be defined in your Ansible inventory file, listing all nodes that require Docker login.

## Usage

To execute the `docker-login` playbook, run the following command:

```bash
ansible-playbook login.yml --ask-vault-pass
```