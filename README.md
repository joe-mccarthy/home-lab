# Home Lab
[![Issues and PRs](https://img.shields.io/github/issues/joe-mccarthy/homelab?style=flat-square)](https://github.com/joe-mccarthy/homelab/issues)
[![Release](https://img.shields.io/github/v/release/joe-mccarthy/homelab?style=flat-square)](https://github.com/joe-mccarthy/homelab/releases)
[![Last commit](https://img.shields.io/github/last-commit/joe-mccarthy/homelab?style=flat-square)](https://github.com/joe-mccarthy/homelab/commits/main)
[![License](https://img.shields.io/github/license/joe-mccarthy/homelab?style=flat-square)](LICENSE)
[![ansible-lint](https://img.shields.io/github/actions/workflow/status/joe-mccarthy/homelab/ansible-linter.yml?style=flat-square&label=ansible%20lint)](https://github.com/joe-mccarthy/homelab/actions/workflows/ansible-linter.yml)


The purpose of this repository is to create and manage a simple [home lab](https://linuxhandbook.com/homelab/) built around clusters of [Raspberry Pi's](https://www.raspberrypi.com/) using [Docker Swarm](https://docs.docker.com/engine/swarm/) for container deployment and management. The primary tool for managing the home lab is [Ansible](https://docs.ansible.com/ansible/latest/index.html). 

This repository is designed to help you learn, test, and experiment with deploying and managing services in a clustered environment. It provides a structured approach to setting up a home lab, automating deployments, and maintaining the cluster.

For a service-by-service deployment catalog, see [deployments/README.md](deployments/README.md).

> **⚠️ Warning**  
> All examples within this repository are for testing, learning, and development purposes and should not be used for production environments.

---

## Hardware

This home lab is built entirely using Raspberry Pi devices. The current setup includes:

- **8 Raspberry Pi 4s** with 8GB of memory, each with a 64GB SD card.
- **1 Raspberry Pi 5** with 8GB of memory and a 2TB NVMe drive connected via PCIe using a [Pimoroni NVMe base](https://shop.pimoroni.com/products/nvme-base?variant=41219587178579).
- **PoE Hats** for each Raspberry Pi 4 to simplify power and networking.
- **8-Port PoE Switch** to power and connect all devices.

The cluster is designed to be flexible, with services distributed across nodes. However, one node must host the [NFS](https://en.wikipedia.org/wiki/Network_File_System) server for persistent storage. In this setup, the Raspberry Pi 5 with the NVMe drive serves as the NFS server.

### Why Persistent Storage?

Persistent storage is critical for:
- Allowing services to move between nodes without losing data.
- Supporting deployments like [Home Assistant](https://www.home-assistant.io/) that perform frequent writes, which can quickly wear out SD cards.

---

## Authentication, Hosts, and Vaults

Before deploying services to Docker Swarm, the cluster must be defined, machines set up, and sensitive variables secured.

### Defining Hosts

Ansible uses an inventory file to define the hosts it manages. In this project, the inventory is referenced externally using [Ansible configuration](https://docs.ansible.com/ansible/latest/reference_appendices/config.html). A ready-to-use example is provided at [`inventory.example.yml`](inventory.example.yml). Copy it and populate it with your own IPs and hostnames:

```bash
cp inventory.example.yml inventory.yml
```

The inventory defines three Ansible groups: `nfs_servers` (the NFS host, which can double as a Swarm manager), `manager` (all Swarm manager nodes), and `docker` (Swarm worker nodes). Both `manager` and `docker` are children of `cluster`, which lets maintenance playbooks target all nodes at once. The example uses Norse mythology names — `odin` as the primary manager and NFS server, `thor` and `loki` as additional managers, and `freyr`, `tyr`, `heimdall`, `baldur`, `frigg`, and `skadi` as workers.

It's important to consider the number of managers you have for the size and reliability of the cluster[^1].

### Setting Up Machines

All nodes in the cluster have had a similar installation of Ubuntu 24.10 along with a hostname to match what their role will become within the cluster. 

Once the machine has been booted up for the first time, then a static IP address is assigned to it from my router. Your configuration, IP address ranges, and networking hardware might vary and limit what you're able to do with networking. However, hostnames should in most cases be enough to get the cluster running.

Other than this configuration, all other management will be done through Ansible.

### Keeping Secrets Secure with Vaults

Within this repository, there are deployments and configurations that require sensitive information in order to work. For example, the Cloudflare API token for setting the Dynamic DNS records that you wish. For this reason, the use of [Ansible Vault](https://docs.ansible.com/ansible/latest/vault_guide/index.html) is leaned on in order to keep these sensitive credentials safe.

A complete reference of every vault variable expected across all deployments is documented in [`vault.template.yml`](vault.template.yml) at the root of this repository. Copy it, fill in your real values, and encrypt it:

```bash
cp vault.template.yml vault.yml
# edit vault.yml with real values
ansible-vault encrypt vault.yml
```

The variables are referenced within each deployment's `group_vars/all.yml`, which maps vault lookups to the variables used in templates. If you prefer not to use vaults, you can replace the vault lookup with your actual data directly — for example:

```yml
# With vault (recommended)
cf_token: "{{ vault_cf_token }}"

# Without vault (keep the file private)
cf_token: your-actual-token
```

#### Domain Name Information

There is one other vault variable used in the inventory itself — the base domain for all proxy routing. This is consumed by [Traefik](https://doc.traefik.io/traefik/) to define what it listens for, where it forwards traffic, and which domain to issue certificates for. It is documented in [`vault.template.yml`](vault.template.yml) under `general.domain`.

```yml
general:
   domain: "example.com"
```

This value can sit in `all.vars` of your `inventory.yml` as plain text, or be encrypted using [Ansible Vault Strings](https://docs.ansible.com/ansible/latest/vault_guide/vault_encrypting_content.html#encrypting-individual-variables-with-ansible-vault) — sharing less is always better.

---

## Getting Started

Assuming that you've created an `inventory.yml` from [`inventory.example.yml`](inventory.example.yml) as described above, along with actually installing an Operating System and setting up the initial Hardware, I recommend the following steps.

1. **Create an SSH Key Pair**:
   - Generate a private/public key pair for the cluster:
     ```bash
     ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f ~/.ssh/homelab
     ```
   - Place the private key in your `.ssh` folder.

2. **Set Up Machines**:
   - Use the [set-up-machine](maintenance/set-up-machine/README.md) playbook to prepare all nodes in the cluster. This playbook installs updates, required packages, and copies the public SSH key to each node.

3. **Create the Cluster**:
   - Once the machines are ready, initialize Docker Swarm using the [create-swarm](docker-swarm/README.md) playbook.

4. **Deploy Core Services**:
   - Deploy essential services like Traefik, Portainer, and Dynamic DNS using the [core-deployments](deployments/core-deployments/README.md) playbook.
  - Use the [deployments catalog](deployments/README.md) for the full list of available services and per-service documentation.

---

## Contributions

I'm one person, who has learned this stuff and developed this for their own use. I don't pretend to be an expert; I'm just sharing what I've learned. That being said, if you see any issues, or ways to do things better, then please raise an issue and pull request to make everything better for everyone. Any contributions you make are greatly appreciated.

If you have a suggestion that would make this better, please fork the repo and create a pull request. You can also simply open an issue with the tag "enhancement".

Don't forget to give the project a star! Thanks again!

1. Fork the Project
1. Create your Feature Branch (git checkout -b feature/AmazingFeature)
1. Commit your Changes (git commit -m 'Add some AmazingFeature')
1. Push to the Branch (git push origin feature/AmazingFeature)
1. Open a Pull Request

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

[^1]: An odd number N of manager nodes in the cluster tolerates the loss of at most (N-1)/2 managers. Docker recommends a maximum of seven manager nodes for a swarm.
