# Set Up Hosts

This directory contains the playbook and associated files required to set up machines in the cluster. The `set-up-machine` playbook automates the initial configuration of hosts, ensuring they are ready for deployment and maintenance tasks. This includes configuring SSH access, updating the system, installing essential packages, and rebooting the machine if necessary.

## Rationale

Setting up machines manually can be time-consuming and error-prone, especially in a cluster environment where consistency is critical. This playbook ensures that all machines in the cluster are configured uniformly, reducing the risk of misconfiguration and improving operational efficiency. By automating tasks such as SSH key setup, system updates, and package installation, this playbook simplifies the onboarding of new machines and ensures they meet the cluster's requirements.

## Features

The `set-up-machine` playbook performs the following tasks:
1. **Configure SSH Access**:
   - Adds a public SSH key to the `authorized_keys` file of the target user, enabling secure, passwordless SSH access.
   - Ensures that administrators can manage machines without relying on passwords.

2. **Update and Upgrade System Packages**:
   - Updates the package cache and performs a full distribution upgrade to ensure the system is up-to-date.
   - Checks if a reboot is required after updates and reboots the machine if necessary.

3. **Install Core Packages**:
   - Installs a predefined list of essential packages required for system monitoring, networking, and general functionality.

## Files

### 1. `setup.yml`
This is the main playbook that performs the setup tasks. It:
- Configures SSH access for the current user.
- Updates and upgrades system packages.
- Installs essential packages.
- Reboots the machine if required.

For more details, refer to the [setup.yml file](./setup.yml).

### 2. `group_vars/all.yml`
This file defines the group variables used by the playbook. It includes:
- `public_key`: The path to the public SSH key used for configuring passwordless SSH access.

For more details, refer to the [group_vars/all.yml file](./group_vars/all.yml).

## Prerequisites

Before running the `set-up-machine` playbook, ensure the following:
1. **SSH Key Pair**:
   - Generate an SSH key pair if one does not already exist. Use the following command:
     ```bash
     ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f ~/.ssh/homelab
     ```
   - Update the `public_key` variable in `group_vars/all.yml` to point to the public key file (e.g., `~/.ssh/homelab.pub`).

2. **Ansible Inventory**:
   - Ensure the `all` group in your inventory file includes all the machines you want to configure.

3. **Sudo Access**:
   - Verify that the user running the playbook has sudo privileges on the target machines.

## Usage

To execute the `set-up-machine` playbook, run the following command:

```bash
ansible-playbook setup.yml --ask-pass --ask-become-pass
```