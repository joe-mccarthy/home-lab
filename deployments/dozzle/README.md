# Dozzle Deployment

[![Ansible](https://img.shields.io/badge/Ansible-Automation-EE0000?logo=ansible&logoColor=white&style=flat-square)](https://docs.ansible.com/) [![Docker Swarm](https://img.shields.io/badge/Docker%20Swarm-Orchestration-2496ED?logo=docker&logoColor=white&style=flat-square)](https://docs.docker.com/engine/swarm/) [![Traefik](https://img.shields.io/badge/Traefik-HTTPS%20Access-24A1C1?logo=traefikproxy&logoColor=white&style=flat-square)](https://doc.traefik.io/traefik/) ![Dozzle](https://img.shields.io/badge/Dozzle-latest-5C5C5C?style=flat-square)

This directory contains an Ansible-based deployment for **Dozzle** using Docker Swarm and Traefik for HTTPS ingress. Dozzle provides a lightweight real-time log viewer for Docker containers, making it easy to inspect service logs from a browser.

## What this deployment does

- Stops any running `dozzle_dozzel` service before redeployment.
- Renders a Docker Compose stack definition from a Jinja2 template.
- Deploys the stack to Docker Swarm using `docker stack deploy`.
- Exposes Dozzle through Traefik with automatic HTTPS redirection and Let's Encrypt TLS certificates.
- Cleans up temporary compose files after deployment.

## Architecture overview

The deployment runs a single Dozzle service in **global** mode so each swarm node can run an instance.

- **Docker Swarm stack**: managed as a stack named `dozzle`.
- **Traefik**: routes external traffic via the shared `proxy` overlay network.
- **Services**:
  - `dozzle` - Dozzle web UI on internal port `8080`, configured with `DOZZLE_MODE=swarm`.
- **Networks**:
  - `proxy` (external) - used by Traefik for ingress.
  - `dozzle` (overlay) - service-local swarm network.

## Prerequisites

1. **Docker Swarm cluster already initialized** (at least one manager node).
2. **Traefik stack deployed**, providing an external overlay network named `proxy`.
3. **Ansible control node** with access to `inventory.yml` (see [`inventory.example.yml`](../../inventory.example.yml)) and Vault secrets (see [`vault.template.yml`](../../vault.template.yml)).
4. **Docker socket access** on target nodes (`/var/run/docker.sock`), since Dozzle reads logs directly from the Docker API.

## Deployment

```bash
ansible-playbook -i ../../inventory.yml deployments/dozzle/deploy.yml
```

## Configuration

### Key variables

- `vault.shared.general.domain` - the base domain used to construct the service URL (`dozzle.<domain>`).

This value is sourced from Ansible Vault. See [`vault.template.yml`](../../vault.template.yml) at the repo root for the full variable reference.

## Important paths

- `deployments/dozzle/deploy.yml` - main playbook
- `deployments/dozzle/roles/deploy/tasks/main.yml` - pre-cleanup, stack deployment, and cleanup steps
- `deployments/dozzle/templates/docker-compose.yaml` - Docker stack template

## Security considerations

- All traffic is redirected from HTTP to HTTPS by Traefik middleware.
- TLS certificates are issued automatically using the Let's Encrypt `letsencrypt` resolver.
- The Dozzle container mounts `/var/run/docker.sock`, which grants broad visibility into container runtime metadata and logs. Restrict access to the Dozzle URL and trusted users.
- The `ai.ix.auto-update=true` label enables automatic image updates via the [Cioban](../cioban/README.md) service.

## Troubleshooting

Common issues and their resolutions:

- **Service not accessible**: confirm Traefik is running and DNS resolves `dozzle.<domain>` to your cluster ingress IP.
- **TLS certificate issues**: verify Traefik ACME configuration and that `web`/`websecure` entrypoints are active.
- **Missing logs**: ensure `/var/run/docker.sock` is mounted and readable by the Dozzle container.
- **Deployment conflicts**: if an old service remains, manually remove it with `docker service rm dozzle_dozzel` and redeploy.