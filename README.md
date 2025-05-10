# Home Lab
![GitHub Issues or Pull Requests](https://img.shields.io/github/issues/joe-mccarthy/home-lab?style=for-the-badge)
![GitHub Release](https://img.shields.io/github/v/release/joe-mccarthy/home-lab?style=for-the-badge)
![GitHub commits since latest release](https://img.shields.io/github/commits-since/joe-mccarthy/home-lab/latest?style=for-the-badge)
![GitHub License](https://img.shields.io/github/license/joe-mccarthy/home-lab?style=for-the-badge)

The purpose of this repository is to create and manage a simple [home lab](https://linuxhandbook.com/homelab/) built around clusters of [Raspberry Pi's](https://www.raspberrypi.com/) using [Docker Swarm](https://docs.docker.com/engine/swarm/) for container deployment and management. The primary tool for managing the home lab is [Ansible](https://docs.ansible.com/ansible/latest/index.html). 

This repository is designed to help you learn, test, and experiment with deploying and managing services in a clustered environment. It provides a structured approach to setting up a home lab, automating deployments, and maintaining the cluster.

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

Ansible uses an inventory file to define the hosts it manages. In this project, the inventory is referenced externally using [Ansible configuration](https://docs.ansible.com/ansible/latest/reference_appendices/config.html). Below is an example inventory structure:

```yml
cluster:
  children:
    docker:
    manager:

nfs_servers:
  hosts:
    nfs:

docker:
    hosts:
      docker-[01:06]:

manager:
    hosts:      
      manager-[01:02]:
      nfs:
```

The first block is the definition of the cluster itself so it can be referenced directly for common actions. This definition is a group that doesn't reference actual hosts, but it references other groups. The two groups that it references are __docker__ and __manager__, which correspond to their roles within the cluster. It's important to consider the number of managers you have for the size and reliability of the cluster[^1].

The machines have also been configured to have hostnames that define their role within the cluster. For instance, docker-01 through docker-06 will be considered worker nodes, while all hosts under the manager group would be managers within the cluster. This includes the host nfs. 

### Setting Up Machines

All nodes in the cluster have had a similar installation of Ubuntu 24.10 along with a hostname to match what their role will become within the cluster. 

Once the machine has been booted up for the first time, then a static IP address is assigned to it from my router. Your configuration, IP address ranges, and networking hardware might vary and limit what you're able to do with networking. However, hostnames should in most cases be enough to get the cluster running.

Other than this configuration, all other management will be done through Ansible.

### Keeping Secrets Secure with Vaults

Within this repository, there are deployments and configurations that require sensitive information in order to work. For example, the Cloudflare API token for setting the Dynamic DNS records that you wish. For this reason, the use of [Ansible Vault](https://docs.ansible.com/ansible/latest/vault_guide/index.html) is leaned on in order to keep these sensitive credentials safe. 

That being said, the variables are referenced within each of the deployments as group vars, and these group vars look up the same variable within the Ansible Vault, so feel free to replace the vault lookup with your actual data. Here is an example below for setting the Cloudflare email and token.

```yml
cf_token: "{{ vault_cf_token }}" # Cloudflare API token
cf_email: "{{ vault_cf_email }}" # Cloudflare email
``` 
The above defines a variable to be used within the context of the Ansible run, however, it calls on another variable which is actually held within a vault. To amend this and not use vaults, change it to the following, however, be careful and remember not to share it.

```yml
cf_token: compromisedToken # Cloudflare API token
cf_email: compromisedEmail # Cloudflare email
``` 

#### Domain Name Information

There is one other location where there is a vault variable being used, which is within the hosts inventory file itself. This is the group vars to define the domain for all the proxy information. This was included within the inventory because it applies to all machines, well it does in my instance. This information is used with [Traefik](https://doc.traefik.io/traefik/) to define what it's listening for and where it will forward to and required to issue certificates for secure connections.

```yml
all:
  vars:
    general:
      domain: ".something.com"
```

The above can be added to the top of the inventory file that you're using, it can be left as plain text if you wish, however sharing less is always better so I've encrypted mine using [Ansible Vault Strings](https://docs.ansible.com/ansible/latest/vault_guide/vault_encrypting_content.html#encrypting-individual-variables-with-ansible-vault). 

---

## Getting Started

Assuming that you've created an inventory file like stated above, along with actually installing an Operating System and setting up the initial Hardware, I recommend the following steps. 

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