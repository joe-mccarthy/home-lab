# Omni Tools Deployment

[![Ansible](https://img.shields.io/badge/Ansible-Automation-EE0000?logo=ansible&logoColor=white&style=flat-square)](https://docs.ansible.com/) [![Docker Swarm](https://img.shields.io/badge/Docker%20Swarm-Orchestration-2496ED?logo=docker&logoColor=white&style=flat-square)](https://docs.docker.com/engine/swarm/) [![Traefik](https://img.shields.io/badge/Traefik-HTTPS%20Access-24A1C1?logo=traefikproxy&logoColor=white&style=flat-square)](https://doc.traefik.io/traefik/) ![Omni Tools](https://img.shields.io/badge/Omni%20Tools-latest-5C5C5C?style=flat-square)

This directory contains an Ansible-based deployment for **Omni Tools** using Docker Swarm and Traefik for HTTPS ingress. Omni Tools is a self-hosted, browser-based collection of everyday utility tools — a lightweight, privacy-friendly alternative to scattered online services.

## What this deployment does

- Stops any running `omni_omni` service before redeployment.
- Renders a Docker Compose stack definition from a Jinja2 template.
- Deploys the stack to Docker Swarm using `docker stack deploy`.
- Uses Traefik labels to expose the Omni Tools UI over HTTPS using Let's Encrypt.
- Cleans up temporary compose files after deployment.

## Architecture overview

The deployment is a single stateless service with no persistent storage requirements.

- **Docker Swarm stack**: managed as a Swarm stack (`docker stack deploy`).
- **Traefik**: handles ingress on the `proxy` overlay network, enforcing HTTPS and issuing Let's Encrypt certificates.
- **Services**:
  - `omni` – the Omni Tools web application, served on port 80 internally.

## Prerequisites

1. **Docker Swarm cluster already initialized** (at least one manager and one worker node).
2. **Traefik stack deployed**, providing an external overlay network named `proxy`.
3. **Ansible control node** with access to `inventory.yml` (see [`inventory.example.yml`](../../inventory.example.yml)) and Vault secrets (see [`vault.template.yml`](../../vault.template.yml)).

## Deployment

```bash
ansible-playbook -i ../../inventory.yml deployments/omni/deploy.yml
```

## Configuration

### Key variables

- `vault.shared.general.domain` – the base domain used to construct the service URL (`omni.<domain>`).

This value is sourced from Ansible Vault. See [`vault.template.yml`](../../vault.template.yml) at the repo root for the full variable reference.

## Important paths

- `deployments/omni/deploy.yml` – main playbook
- `deployments/omni/roles/deploy/tasks/main.yml` – stops existing service, deploys stack, and cleans up
- `deployments/omni/templates/docker-compose.yaml` – the Docker stack template

## Security considerations

- All traffic is redirected from HTTP to HTTPS via a Traefik middleware rule.
- TLS certificates are managed automatically by Traefik using the Let's Encrypt `letsencrypt` resolver.
- The container is deployed on worker nodes only and runs as a single replica.
- The `ai.ix.auto-update=true` label enables automatic image updates via the [Cioban](../cioban/README.md) service.

## Troubleshooting

Common issues and their resolutions:

- **Service not accessible**: confirm the `proxy` overlay network exists and Traefik is running.
- **Certificate not issued**: verify DNS resolves `omni.<domain>` to the cluster's public IP.
- **Deployment fails**: check that Docker Swarm is initialized and the target worker nodes are healthy.
- **Stale service**: the playbook stops `omni_omni` before redeployment; if the stop step fails, manually remove the service with `docker service rm omni_omni`.
