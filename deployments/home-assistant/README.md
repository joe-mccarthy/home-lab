# 🏡 Home Assistant Deployment

[![Ansible](https://img.shields.io/badge/Ansible-Automation-EE0000?logo=ansible&logoColor=white&style=flat-square)](https://docs.ansible.com/) [![Docker Swarm](https://img.shields.io/badge/Docker%20Swarm-Orchestration-2496ED?logo=docker&logoColor=white&style=flat-square)](https://docs.docker.com/engine/swarm/) [![Traefik](https://img.shields.io/badge/Traefik-HTTPS%20Access-24A1C1?logo=traefikproxy&logoColor=white&style=flat-square)](https://doc.traefik.io/traefik/) [![MQTT](https://img.shields.io/badge/MQTT-IoT%20Messaging-660066?logo=eclipsemosquitto&logoColor=white&style=flat-square)](https://mqtt.org/) [![Zigbee](https://img.shields.io/badge/Zigbee-Device%20Network-EB0443?style=flat-square)](https://csa-iot.org/all-solutions/zigbee/) [![Matter](https://img.shields.io/badge/Matter-Wi--Fi%20Devices-00BFB3?style=flat-square)](https://csa-iot.org/all-solutions/matter/) ![Home Assistant](https://img.shields.io/badge/Home%20Assistant-2026.7.2-5C5C5C?style=flat-square) ![Zigbee2MQTT](https://img.shields.io/badge/Zigbee2MQTT-v2.12.1-5C5C5C?style=flat-square) ![Matter Server](https://img.shields.io/badge/Matter%20Server-1.4.0-5C5C5C?style=flat-square) ![Mosquitto](https://img.shields.io/badge/Mosquitto-2.1.2--alpine-5C5C5C?style=flat-square)

An Ansible deployment for a Docker Swarm based smart home stack: [Home Assistant](https://www.home-assistant.io/), [Matter.js Server](https://github.com/matter-js/matterjs-server), [Zigbee2MQTT](https://www.zigbee2mqtt.io/), and [Mosquitto](https://mosquitto.org/), routed through Traefik and backed by shared NFS storage.

This deployment is built for a small home lab cluster where Home Assistant should survive node maintenance, keep its configuration on persistent storage, and integrate cleanly with Zigbee devices through MQTT and Wi-Fi Matter devices through a local Matter controller.

> [!WARNING]
> This stack is designed for a home lab. The bundled Mosquitto service currently runs with the upstream no-auth configuration, and Matter Server listens directly on the Matter node's LAN port `5580`, so review MQTT and LAN exposure before using it anywhere sensitive.

---

## ✨ What It Deploys

| Service | Image | Purpose |
| --- | --- | --- |
| `homeassistant` | `ghcr.io/home-assistant/home-assistant:2026.7.2` | Main automation platform and web UI. |
| `matter-server` | `ghcr.io/matter-js/matterjs-server:1.4.0` | Matter controller WebSocket server for Wi-Fi Matter devices. |
| `zigbee2mqtt` | `ghcr.io/koenkk/zigbee2mqtt:2.12.1` | Bridges Zigbee devices into MQTT topics Home Assistant can discover. |
| `mqtt` | `eclipse-mosquitto:2.1.2-alpine` | MQTT broker used by Zigbee2MQTT and Home Assistant. |

The stack name is `home_assistant`, so Swarm service names become:

- `home_assistant_homeassistant`
- `home_assistant_matter-server`
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

Matter Wi-Fi devices ── Matter Server host network
                                  ▲
                                  │
Home Assistant ── trusted LAN ────┘
```

Traefik exposes:

- `https://homeassistant.<your-domain>`
- `https://zigbee2mqtt.<your-domain>`

Matter Server is not exposed through Traefik. Home Assistant should connect directly to the dedicated Matter node's trusted LAN address:

```text
ws://<matter-node-lan-ip>:5580/ws
```

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
- Exactly one trusted-LAN Swarm node labelled both `storage=true` and `matter=true`.
- The external `proxy` overlay network created by the core/Traefik deployment.
- Traefik already deployed and able to route wildcard service domains.
- NFS mounted and available on the deployment host.
- A Zigbee coordinator connected to the node that will run Zigbee2MQTT.
- IPv6 and mDNS/multicast working between the Matter node, the phone used for commissioning, and Wi-Fi Matter devices.
- A stable LAN IP or LAN DNS record for the Matter node.
- Vault values defined in [`../../vault.template.yml`](../../vault.template.yml).

The compose template places services that use persistent data on nodes with:

```yaml
node.labels.storage == true
```

Matter Server also requires the dedicated placement label:

```yaml
node.labels.matter == true
```

Persist `matter: "true"` under that node's `swarm_labels` in `inventory.yml`. For an existing Swarm, apply it once from a manager before deploying:

```bash
docker node update --label-add matter=true <matter-node>
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
docker service ps home_assistant_matter-server
docker service ps home_assistant_zigbee2mqtt
docker service ps home_assistant_mqtt
```

Follow logs while the stack starts:

```bash
docker service logs -f home_assistant_homeassistant
docker service logs -f home_assistant_matter-server
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
/mnt/nfs/docker/home_assistant/matter-server/data
/mnt/nfs/docker/home_assistant/mosquitto
/mnt/nfs/docker/home_assistant/zigbee2mqtt
/mnt/nfs/docker/home_assistant/zigbee2mqtt/data
```

The Swarm stack mounts the storage export paths into containers:

```text
/exports/docker/home_assistant                -> /config
/exports/docker/home_assistant/matter-server/data -> /data
/exports/docker/home_assistant/zigbee2mqtt/data -> /app/data
/exports/docker/home_assistant/mosquitto      -> /mosquitto
```

Matter.js Server runs as UID/GID `1000`. Deployment applies that ownership recursively to `matter-server/data` after stopping the old service, including existing Python Matter Server data that the replacement server migrates on first start.

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

## 🧩 Matter Notes

Matter.js Server runs on Docker's predefined `host` network because Matter over Wi-Fi depends on local IPv6 and mDNS multicast. It is pinned to the one node carrying both the `storage=true` and `matter=true` labels, giving its host-networked endpoint a predictable LAN address.

The archived Python Matter Server ended at version `8.1.2`. Matter.js Server is its maintained, WebSocket-compatible successor and migrates an existing data directory on first start.

> [!NOTE]
> Home Assistant recommends Home Assistant OS with the official Matter Server app as the supported Matter installation type. This stack uses the documented self-managed Docker container path for a Swarm-based home lab.

After deployment, add the Matter integration in Home Assistant under **Settings > Devices & services**. When asked for the Matter Server connection, use:

```text
ws://<matter-node-lan-ip>:5580/ws
```

Use the node's stable LAN IP or a LAN DNS name that resolves to that IP. Do not use a Swarm service name: a host-networked service is intentionally not attached to the overlay network.

For Wi-Fi Matter device commissioning, use the Home Assistant Companion app on a phone with Bluetooth enabled and connected to the same Wi-Fi/LAN where the Matter device will run. Many Wi-Fi Matter devices require 2.4 GHz Wi-Fi during onboarding.

This deployment supports Wi-Fi Matter devices. Thread Matter devices still need a Thread border router and additional Thread network planning.

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
docker service update --force home_assistant_matter-server
docker service update --force home_assistant_zigbee2mqtt
docker service update --force home_assistant_mqtt
```

### Inspect Rendered Services

```bash
docker service inspect home_assistant_homeassistant
docker service inspect home_assistant_matter-server
docker service inspect home_assistant_zigbee2mqtt
docker service inspect home_assistant_mqtt
```

### Check Persistent Data

```bash
ls -la /mnt/nfs/docker/home_assistant
ls -la /mnt/nfs/docker/home_assistant/matter-server/data
ls -la /mnt/nfs/docker/home_assistant/zigbee2mqtt/data
```

---

## 🧯 Troubleshooting

| Symptom | What to Check |
| --- | --- |
| Home Assistant returns proxy errors | Confirm `vault.services.home_assistant.proxy` matches the Traefik network CIDR or proxy IP. |
| Home Assistant cannot connect to Matter | Confirm the integration uses `ws://<matter-node-lan-ip>:5580/ws`, port `5580` is reachable from the Home Assistant container, and `home_assistant_matter-server` is running. |
| Matter devices fail commissioning | Confirm IPv6 is enabled, mDNS/multicast is not filtered, and the phone, Matter node, and Matter device are on the same LAN or VLAN. |
| Zigbee2MQTT cannot reach MQTT | Confirm `vault.services.home_assistant.mqtt_server` is reachable from the stack, usually `mqtt://mqtt:1883`. |
| Zigbee coordinator is missing | Check the device path on the storage-labelled node with `ls -la /dev/serial/by-id /dev/ttyUSB* /dev/ttyACM*`. |
| Services stay pending | Confirm a storage node exists and exactly one eligible node has both `storage=true` and `matter=true`. |
| Traefik routes do not work | Confirm the `proxy` network exists and DNS points `homeassistant.<domain>` and `zigbee2mqtt.<domain>` at Traefik. |
| Config changes do not appear | Existing config files are preserved by design. Edit files under `/mnt/nfs/docker/home_assistant` directly or remove the file before rerunning the playbook. |

Useful commands:

```bash
docker service ps home_assistant_homeassistant --no-trunc
docker service logs home_assistant_homeassistant
docker service logs home_assistant_matter-server
docker service logs home_assistant_zigbee2mqtt
docker network inspect proxy
docker network inspect host
```

---

## 🛡️ Security Notes

- Keep `vault.services.home_assistant.proxy` narrow. Do not trust broad networks unless they are truly controlled.
- The bundled Mosquitto service uses `mosquitto -c /mosquitto-no-auth.conf`; restrict network exposure or add authentication before exposing MQTT beyond the stack.
- Do not expose Matter Server port `5580` to the internet. It should only be reachable on the trusted LAN.
- Leave Zigbee pairing disabled except during device onboarding.
- Enable strong Home Assistant authentication and MFA from the Home Assistant UI.
- Back up `/mnt/nfs/docker/home_assistant`, especially `.storage/`, `matter-server/data`, and `zigbee2mqtt/data`.

---

## 📚 Related Docs

- [Root README](../../README.md)
- [Deployments catalog](../README.md)
- [Core deployments](../core-deployments/README.md)
- [Traefik deployment](../traefik/README.md)
- [Vault template](../../vault.template.yml)
- [Home Assistant Matter integration](https://www.home-assistant.io/integrations/matter/)
- [Matter.js Server Docker notes](https://github.com/matter-js/matterjs-server/blob/main/docs/docker.md)
