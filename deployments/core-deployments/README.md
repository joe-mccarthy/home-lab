# Core Deployments

Before deploying other services within this repository or creating your own, it is recommended to first deploy this stack, which I consider the core deployments of the cluster. These deployments provide essential functionality and serve as dependencies for other services, such as enabling service updates, access, and security. This Ansible playbook is a collection of imports for the individual playbooks of each core deployment.

## Traefik

[Traefik](https://traefik.io/) is responsible for acting as a reverse proxy for other services, enabling name resolution instead of relying on IP addresses and ports. Additionally, it integrates with DNS providers to perform challenges and issue valid certificates for HTTPS connections. For more details, refer to the [Traefik deployment README](../traefik/README.md).

## Portainer

[Portainer](https://www.portainer.io/) provides a user-friendly interface for managing Docker instances and Docker Swarm clusters. While the services in this repository are managed by Ansible, Portainer offers a convenient UI to monitor the cluster's status and activity. For more details, refer to the [Portainer deployment README](../portainer/README.md).

## Dynamic DNS

Dynamic DNS ensures that services hosted on the home lab cluster are accessible by updating DNS records to reflect the public IP address of your internet gateway. This service uses the Cloudflare API (assuming Cloudflare is your DNS provider) to update domain name entries dynamically. For more details, refer to the [Dynamic DNS deployment README](../ddns/README.md).

## Cioban

[Cioban](https://github.com/cioban) is responsible for automatically updating Docker services, including itself. It requires access to the Docker socket and relies on labels applied to services to determine which ones to update. When a new image for a deployed service is detected, Cioban updates the service to use the new image, adhering to the deployment policies defined for that service. For more details, refer to the [Cioban deployment README](../cioban/README.md).