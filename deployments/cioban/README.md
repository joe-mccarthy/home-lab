# Cioban Deployment

[![Ansible](https://img.shields.io/badge/Ansible-Automation-EE0000?logo=ansible&logoColor=white&style=flat-square)](https://docs.ansible.com/) [![Docker Swarm](https://img.shields.io/badge/Docker%20Swarm-Orchestration-2496ED?logo=docker&logoColor=white&style=flat-square)](https://docs.docker.com/engine/swarm/) ![Auto Update](https://img.shields.io/badge/Auto%20Update-Service%20Images-2EA44F?style=flat-square) ![Docker Socket](https://img.shields.io/badge/Docker%20Socket-Required-2496ED?logo=docker&logoColor=white&style=flat-square) ![Cioban](https://img.shields.io/badge/cioban-v0.17.14-5C5C5C?style=flat-square)

This deployment runs **Cioban** as a single Docker Swarm service that checks labelled services for image updates. Ansible validates the registry configuration file on every manager before it replaces the existing service and deploys the rendered stack.

## Overview

Cioban needs two privileged host resources:

- `/var/run/docker.sock`, mounted read-write so Cioban can update Swarm services.
- A Docker CLI configuration file, mounted read-only at `/root/.docker/config.json` so Cioban can inspect private registries.

The service can run on any manager node. The host-side Docker configuration must therefore exist with the same path, ownership, and permissions on every manager.

## Configuration

The default host path is defined in [`group_vars/all.yml`](group_vars/all.yml):

```yaml
cioban_docker_config_path: "/root/.docker/config.json"
```

Override it with an inventory variable when required, but keep it absolute. Tilde expansion and other shell shortcuts are intentionally rejected because Docker resolves bind sources in the deployment execution context.

## Prerequisites

Before deploying Cioban:

1. Ensure the `manager` inventory group contains every Swarm manager.
2. Authenticate every cluster node to the configured private registries:

   ```bash
   ansible-playbook -i inventory.yml maintenance/docker-login/login.yml --ask-vault-pass
   ```

3. Confirm `cioban_docker_config_path` is a regular file owned by `root:root` with mode `0600` on every manager.

## Deployment Process

Run the deployment from the repository root:

```bash
ansible-playbook -i inventory.yml deployments/cioban/deploy.yml
```

The playbook checks the configured path on every manager before stopping the existing Cioban service. A missing, non-regular, incorrectly owned, or incorrectly permissioned file stops the run without replacing the service. After the checks pass, one selected manager deploys the stack.

## Security Considerations

- Never commit Docker registry credentials to this repository.
- Keep the host configuration owned by `root:root` with mode `0600`; the container mount remains read-only.
- Treat Cioban as a privileged service. Its read-write Docker socket gives it control over the Swarm manager, independently of the registry configuration mount.
- Rotate registry credentials with the Docker login maintenance playbook so every manager stays consistent.

## Troubleshooting

- **Absolute-path assertion fails**: remove `~` or relative components and set `cioban_docker_config_path` to an absolute manager-host path.
- **Protected-file assertion fails**: run the Docker login maintenance playbook on every cluster node, then verify ownership and mode on the named manager.
- **Cioban cannot inspect a private registry**: confirm the registry entry exists in the configured Docker file on every manager and rerun the login playbook after credential rotation.
