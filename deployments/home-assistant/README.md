# 🏡 Home Assistant Deployment

[![Ansible](https://img.shields.io/badge/Ansible-Automation-EE0000?logo=ansible&logoColor=white&style=flat-square)](https://docs.ansible.com/) [![Docker Swarm](https://img.shields.io/badge/Docker%20Swarm-Orchestration-2496ED?logo=docker&logoColor=white&style=flat-square)](https://docs.docker.com/engine/swarm/) [![Traefik](https://img.shields.io/badge/Traefik-HTTPS%20Access-24A1C1?logo=traefikproxy&logoColor=white&style=flat-square)](https://doc.traefik.io/traefik/) [![MQTT](https://img.shields.io/badge/MQTT-IoT%20Messaging-660066?logo=eclipsemosquitto&logoColor=white&style=flat-square)](https://mqtt.org/) [![Zigbee](https://img.shields.io/badge/Zigbee-Device%20Network-EB0443?style=flat-square)](https://csa-iot.org/all-solutions/zigbee/) ![Home Assistant](https://img.shields.io/badge/Home%20Assistant-2026.7.2-5C5C5C?style=flat-square) ![Zigbee2MQTT](https://img.shields.io/badge/Zigbee2MQTT-v2.12.1-5C5C5C?style=flat-square) ![Mosquitto](https://img.shields.io/badge/Mosquitto-2.1.2--alpine-5C5C5C?style=flat-square)

An Ansible deployment for a Docker Swarm based smart home stack: [Home Assistant](https://www.home-assistant.io/), [Zigbee2MQTT](https://www.zigbee2mqtt.io/), and [Mosquitto](https://mosquitto.org/), routed through Traefik and backed by shared NFS storage.

This deployment is built for a small home lab cluster where Home Assistant should survive node maintenance, keep its configuration on persistent storage, and integrate cleanly with Zigbee devices through MQTT.

> [!WARNING]
> This stack is designed for a home lab. The bundled Mosquitto service currently runs with the upstream no-auth configuration, so review MQTT exposure and network access before using it anywhere sensitive.

---

## ✨ What It Deploys

| Service | Image | Purpose |
| --- | --- | --- |
| `homeassistant` | `ghcr.io/home-assistant/home-assistant:2026.7.2` | Main automation platform and web UI. |
| `zigbee2mqtt` | `ghcr.io/koenkk/zigbee2mqtt:2.12.1` | Bridges Zigbee devices into MQTT topics Home Assistant can discover. |
| `mqtt` | `eclipse-mosquitto:2.1.2-alpine` | MQTT broker used by Zigbee2MQTT and Home Assistant. |

The stack name is `home_assistant`, so Swarm service names become:

- `home_assistant_homeassistant`
- `home_assistant_zigbee2mqtt`
- `home_assistant_mqtt`

---

## 🧱 Architecture

```text
Zigbee devices
     │
     ▼
Zigbee coordinator ── Zigbee2MQTT ── Mosquitto MQTT ── Home Assistant
                                             │
                                             ▼
                                    Traefik HTTPS routing
```

Traefik exposes:

- `https://homeassistant.<your-domain>`
- `https://zigbee2mqtt.<your-domain>`

The domain comes from `vault.shared.general.domain`, which is shared across the deployment catalog.

---

## 📁 Files

| Path | Purpose |
| --- | --- |
| [`deploy.yml`](deploy.yml) | Main playbook. Targets the `manager` group and deploys from one selected manager. |
| [`group_vars/all.yml`](group_vars/all.yml) | Home Assistant variables, vault lookups, and pinned image versions. |
| [`templates/docker-compose.yaml`](templates/docker-compose.yaml) | Docker Swarm stack template. |
| [`templates/configuration.yaml`](templates/configuration.yaml) | Initial Home Assistant configuration template. |
| [`templates/zigbee/configuration.yaml`](templates/zigbee/configuration.yaml) | Initial Zigbee2MQTT configuration template. |
| [`roles/check_nfs/tasks/main.yml`](roles/check_nfs/tasks/main.yml) | Creates persistent directories and first-run config files. |
| [`roles/deploy_home_assistant/tasks/main.yml`](roles/deploy_home_assistant/tasks/main.yml) | Stops old services, renders the compose template, and deploys the stack. |

---

## ✅ Prerequisites

- A working Docker Swarm created by [`docker-swarm/create.yml`](../../docker-swarm/create.yml).
- At least one Swarm node labelled `storage=true`.
- The external `proxy` overlay network created by the core/Traefik deployment.
- Traefik already deployed and able to route wildcard service domains.
- NFS mounted and available on the deployment host.
- A Zigbee coordinator connected to the node that will run Zigbee2MQTT.
- Vault values defined in [`../../vault.template.yml`](../../vault.template.yml).

The compose template places all three services on nodes with:

```yaml
node.labels.storage == true
```

---

## 🚀 Quick Start

Run from the repository root:

```bash
ansible-playbook -i inventory.yml deployments/home-assistant/deploy.yml --ask-vault-pass
```

If your Ansible user requires a sudo password, add:

```bash
--ask-become-pass
```

After deployment, check the Swarm services:

```bash
docker service ls
docker service ps home_assistant_homeassistant
docker service ps home_assistant_zigbee2mqtt
docker service ps home_assistant_mqtt
```

Follow logs while the stack starts:

```bash
docker service logs -f home_assistant_homeassistant
docker service logs -f home_assistant_zigbee2mqtt
docker service logs -f home_assistant_mqtt
```

---

## 🔐 Vault Variables

This deployment reads its sensitive and environment-specific values from the root vault structure:

```yaml
vault:
  shared:
    general:
      domain: "example.com"

  services:
    home_assistant:
      proxy: "172.18.0.0/16"
      mqtt_server: "mqtt://mqtt:1883"
      zigbee_serial_port: "/dev/ttyUSB0"
```

| Variable | Used For |
| --- | --- |
| `vault.shared.general.domain` | Builds Traefik hostnames for Home Assistant and Zigbee2MQTT. |
| `vault.services.home_assistant.proxy` | Populates Home Assistant `trusted_proxies`. |
| `vault.services.home_assistant.mqtt_server` | Sets the MQTT broker URL in Zigbee2MQTT. |
| `vault.services.home_assistant.zigbee_serial_port` | Sets the Zigbee coordinator serial port in Zigbee2MQTT. |

For the bundled Mosquitto service, `mqtt://mqtt:1883` is the expected internal broker URL. Use a different value only if Zigbee2MQTT should connect to an external broker.

---

## 💾 Storage

The NFS role prepares:

```text
/mnt/nfs/docker/home_assistant
/mnt/nfs/docker/home_assistant/mosquitto
/mnt/nfs/docker/home_assistant/zigbee2mqtt
/mnt/nfs/docker/home_assistant/zigbee2mqtt/data
```

The Swarm stack mounts the storage export paths into containers:

```text
/exports/docker/home_assistant                -> /config
/exports/docker/home_assistant/zigbee2mqtt/data -> /app/data
/exports/docker/home_assistant/mosquitto      -> /mosquitto
```

The role also creates these Home Assistant files if they do not already exist:

- `configuration.yaml`
- `automations.yaml`
- `scenes.yaml`
- `scripts.yaml`

The Home Assistant and Zigbee2MQTT configuration templates use `force: false`, so existing configuration files are preserved on later runs.

---

## 🔌 Zigbee Notes

The Zigbee coordinator path must be valid on the node that runs the service. Prefer a stable `/dev/serial/by-id/...` path when possible, because `/dev/ttyUSB0` can change after reboot or when USB devices are reordered.

The compose template currently maps `/dev/ttyUSB0` into the Home Assistant container and the Zigbee2MQTT template uses `vault.services.home_assistant.zigbee_serial_port`. If you use a different device path, update the vault value and make sure the stack template maps the same hardware path.

Pairing is disabled by default:

```yaml
permit_join: false
```

Enable pairing only while adding devices, then turn it off again.

---

## 🛠️ Operations

### Redeploy the Stack

```bash
ansible-playbook -i inventory.yml deployments/home-assistant/deploy.yml --ask-vault-pass
```

The deployment role removes the existing Home Assistant services before redeploying the stack, so expect a short interruption.

### Restart a Service

```bash
docker service update --force home_assistant_homeassistant
docker service update --force home_assistant_zigbee2mqtt
docker service update --force home_assistant_mqtt
```

### Inspect Rendered Services

```bash
docker service inspect home_assistant_homeassistant
docker service inspect home_assistant_zigbee2mqtt
docker service inspect home_assistant_mqtt
```

### Check Persistent Data

```bash
ls -la /mnt/nfs/docker/home_assistant
ls -la /mnt/nfs/docker/home_assistant/zigbee2mqtt/data
```

---

## 🧯 Troubleshooting

| Symptom | What to Check |
| --- | --- |
| Home Assistant returns proxy errors | Confirm `vault.services.home_assistant.proxy` matches the Traefik network CIDR or proxy IP. |
| Zigbee2MQTT cannot reach MQTT | Confirm `vault.services.home_assistant.mqtt_server` is reachable from the stack, usually `mqtt://mqtt:1883`. |
| Zigbee coordinator is missing | Check the device path on the storage-labelled node with `ls -la /dev/serial/by-id /dev/ttyUSB* /dev/ttyACM*`. |
| Services stay pending | Confirm at least one Swarm node has the label `storage=true`. |
| Traefik routes do not work | Confirm the `proxy` network exists and DNS points `homeassistant.<domain>` and `zigbee2mqtt.<domain>` at Traefik. |
| Config changes do not appear | Existing config files are preserved by design. Edit files under `/mnt/nfs/docker/home_assistant` directly or remove the file before rerunning the playbook. |

Useful commands:

```bash
docker service ps home_assistant_homeassistant --no-trunc
docker service logs home_assistant_homeassistant
docker service logs home_assistant_zigbee2mqtt
docker network inspect proxy
```

---

## 🛡️ Security Notes

- Keep `vault.services.home_assistant.proxy` narrow. Do not trust broad networks unless they are truly controlled.
- The bundled Mosquitto service uses `mosquitto -c /mosquitto-no-auth.conf`; restrict network exposure or add authentication before exposing MQTT beyond the stack.
- Leave Zigbee pairing disabled except during device onboarding.
- Enable strong Home Assistant authentication and MFA from the Home Assistant UI.
- Back up `/mnt/nfs/docker/home_assistant`, especially `.storage/` and `zigbee2mqtt/data`.

---

## 📚 Related Docs

- [Root README](../../README.md)
- [Deployments catalog](../README.md)
- [Core deployments](../core-deployments/README.md)
- [Traefik deployment](../traefik/README.md)
- [Vault template](../../vault.template.yml)
