# Paperless (paperless-ngx) Ansible Deployment

[![Ansible](https://img.shields.io/badge/Ansible-Automation-EE0000?logo=ansible&logoColor=white&style=flat-square)](https://docs.ansible.com/) [![Docker Swarm](https://img.shields.io/badge/Docker%20Swarm-Orchestration-2496ED?logo=docker&logoColor=white&style=flat-square)](https://docs.docker.com/engine/swarm/) [![Traefik](https://img.shields.io/badge/Traefik-HTTPS%20Routing-24A1C1?logo=traefikproxy&logoColor=white&style=flat-square)](https://doc.traefik.io/traefik/) [![NFS](https://img.shields.io/badge/NFS-Persistent%20Storage-2C3E50?style=flat-square)](https://documentation.ubuntu.com/server/how-to/networking/install-nfs/)

This directory contains an Ansible-based deployment for **Paperless-ngx** using Docker Swarm, Traefik for ingress routing, and NFS-backed persistent storage.

## What this deployment does

- Prepares required NFS directories and permissions for Paperless (via `check_nfs` role).
- Renders a Docker Compose stack definition from a Jinja2 template.
- Deploys the stack to Docker Swarm using `docker stack deploy`.
- Uses Traefik labels to expose the Paperless UI/API over HTTPS using Let’s Encrypt.

## Architecture overview

- **Docker Swarm stack**: the deployment is managed as a Swarm stack (`docker stack deploy`).
- **Traefik**: handles ingress routing on the `proxy` overlay network.
- **NFS-backed storage**: all data directories are bind-mounted from the host (typically an NFS share) to ensure persistence across container restarts.
- **Services**:
  - `paperless` – main Paperless web UI/API
  - `paperless-broker` – Redis message broker
  - `paperless-gotenberg` – PDF rendering/conversion
  - `paperless-tika` – document parsing (Tika)

## Prerequisites

1. **Docker Swarm cluster already initialized** (at least one manager).
2. **Traefik stack deployed**, providing an external overlay network named `proxy`.
3. **Ansible control node** with access to inventory and Vault secrets.
4. **NFS volume mounted** on target host(s) (default is `/mnt/nfs`).

## Deployment steps

1. Ensure you have the required Vault values defined (see next section).
2. Run the playbook:

```sh
ansible-playbook -i inventory deployments/paperless/deploy.yml
```

## Vault / configuration variables

This deployment pulls sensitive settings from an Ansible Vault file referenced as `vault_paperless`.

The primary runtime configuration lives under `deployments/paperless/group_vars/all.yml`, which maps values from the vault to the variables used in templates.

### Key variables

- `vault_paperless.volumes.*` – host paths for:
  - `paperless` app data
  - `media`, `consume`, `export` directories
  - Redis broker data
- `vault_paperless.ports.*` – port numbers used for:
  - `paperless` (UI/API)
  - `tika` (document parsing)
  - `broker` (Redis)
  - `gotenberg` (PDF rendering)

## Important paths

- `deployments/paperless/deploy.yml` – main playbook
- `deployments/paperless/roles/check_nfs/tasks/main.yml` – creates required NFS directories
- `deployments/paperless/roles/deploy_paperless/tasks/main.yml` – renders compose file and deploys the stack
- `deployments/paperless/templates/docker-compose.yaml` – the Docker stack template

## How the stack is deployed

- The `deploy_paperless` role renders `templates/docker-compose.yaml` into a temporary directory (`/home/docker-compose/paperless`).
- It then runs:

```sh
docker stack deploy -c /home/docker-compose/paperless/docker-compose.yaml paperless
```

- The temporary directory is removed after deployment so the host is not left with artifacts.

## Customization

### Changing the domain
Update `group_vars/all.yml` (or your vault values) so `general.domain` resolves to the desired DNS name.

### Changing ports or storage paths
Adjust the vault values referenced by `vault_paperless.ports.*` and `vault_paperless.volumes.*`.

## Troubleshooting

- If Paperless cannot connect to Redis, verify the `paperless-broker` service is running and reachable from the `paperless` service.
- If the stack fails to start due to missing directories, ensure the NFS share is mounted and the expected paths exist (see `check_nfs` role).
- If Traefik routing does not work, ensure the `proxy` network exists and Traefik is configured to watch the Swarm.