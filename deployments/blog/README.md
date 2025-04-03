# Personal Blog

This deployment sets up two instances of my personal blog. The blog is written using Hugo and a custom theme. Previously, it was deployed to GitHub Pages. However, this example demonstrates pulling an image from a private Docker registry and deploying it in a Docker Swarm environment.

## Overview

The `blog` deployment is designed to:
- Deploy a highly available personal blog using Docker Swarm.
- Demonstrate best practices for handling private Docker registries.
- Showcase how to manage service updates with minimal downtime.

This deployment uses a Docker image hosted as a private package within GitHub. To authenticate with the private registry, you must run either the [docker-login](../../maintenance/docker-login/README.md) playbook or the [create-swarm](../../docker-swarm/README.md) playbook beforehand.

## Features

1. **Highly Available Deployment**:
   - Runs two replicas of the blog service to ensure high availability.
   - Uses Docker Swarm's `replicated` mode to manage multiple instances.

2. **Seamless Updates**:
   - Configured to update one instance at a time to minimize downtime.
   - Uses the `start-first` update order to ensure new instances are running before old ones are terminated.

3. **Traefik Integration**:
   - Includes labels for Traefik to handle proxying, HTTPS certificates, and routing.

4. **Auto-Update Support**:
   - Includes a custom label for [cioban](../cioban/README.md) to monitor and update the service when new image versions are available.

## Deployment Tasks

The playbook performs the following tasks:
1. Ensures Docker is installed and running on the target node.
2. Removes any previously running instances of the blog service to avoid conflicts.
3. Copies the Docker Compose file to the `manager-01` node.
4. Deploys the blog service using Docker Swarm.

## Configuration

### Docker Swarm Update Configuration

The following configuration is used to manage updates for the blog service:

```yml
update_config:
  parallelism: 1  # Update one replica at a time to ensure stability.
  delay: 10s      # Wait 10 seconds between updating replicas to avoid downtime.
  order: start-first  # Start new containers before stopping old ones for seamless updates.
placement:
  constraints: [node.role == worker]  # Ensure the service only runs on worker nodes.
mode: replicated  # Use replicated mode to run multiple instances of the service.
replicas: 2       # Run 2 replicas of the service for high availability.
```

## Deployment Labels

This deployment includes the following labels for Traefik and auto-update functionality:

```yml
labels:
  - "traefik.enable=true"  # Enable Traefik for this service.
  - "traefik.http.routers.blog.entrypoints=web"  # Define the HTTP entry point for the router.
  - "traefik.http.routers.blog.rule=Host(`blog.{{ general.domain }}`)"  # Define the routing rule based on the host.
  - "traefik.http.middlewares.blog-https-redirect.redirectscheme.scheme=https"  # Redirect HTTP to HTTPS.
  - "traefik.http.routers.blog.middlewares=blog-https-redirect"  # Apply the HTTPS redirect middleware.
  - "traefik.http.routers.blog-secure.entrypoints=websecure"  # Define the HTTPS entry point for the secure router.
  - "traefik.http.routers.blog-secure.rule=Host(`blog.{{ general.domain }}`)"  # Define the secure routing rule based on the host.
  - "traefik.http.routers.blog-secure.tls=true"  # Enable TLS for the secure router.
  - "traefik.http.routers.blog-secure.service=blog"  # Link the secure router to the blog service.
  - "traefik.http.routers.blog-secure.tls.certresolver=letsencrypt"  # Use Let's Encrypt for TLS certificates.
  - "traefik.http.services.blog.loadbalancer.server.port=80"  # Define the internal port for the service.
  - "traefik.docker.network=proxy"  # Specify the Traefik network for the service.
  - "ai.ix.auto-update=true"  # Enable auto-update for the service (used by cioban or similar tools).
```

### Key Points:
- **Traefik Labels**: Handle routing, HTTPS redirection, and certificate management.
- **Auto-Update Label**: Used by [cioban](../cioban/README.md) to monitor and update the service when new image versions are available.

## Prerequisites

Before deploying the blog service, ensure the following:
1. **Docker Swarm**:
   - The cluster must be initialized as a Docker Swarm.
   - Nodes should be joined to the Swarm and properly configured.

2. **Traefik Deployment**:
   - Deploy Traefik first to handle proxying and certificate management for the blog service.

3. **Private Docker Registry Login**:
   - Authenticate with the private Docker registry by running the [docker-login](../../maintenance/docker-login/README.md) playbook.

4. **DNS Configuration**:
   - Ensure the domain name for the blog is correctly configured in your DNS provider.

## Usage

To deploy the blog service, run the following command:

```bash
ansible-playbook deploy.yml
```

- This command will deploy the blog service to the `manager-01` node in the Docker Swarm cluster.

## Troubleshooting

1. **Service Not Accessible**:
   - Verify that Traefik is running and properly configured to proxy traffic to the blog service.
   - Check DNS records to ensure they point to the correct public IP address.

2. **Certificate Issues**:
   - Check Traefik's logs for errors related to certificate issuance.
   - Ensure the DNS provider supports API-based challenges for Let's Encrypt.

3. **Deployment Failures**:
   - Verify that the target nodes are reachable via SSH.
   - Check the Ansible inventory file for correct group and host definitions.

## Related Areas

- **Traefik Deployment**: Refer to the [Traefik README](../traefik/README.md) for details on setting up Traefik.
- **Docker Login**: Refer to the [Docker Login README](../../maintenance/docker-login/README.md) for instructions on authenticating with a private Docker registry.
- **Auto-Update**: Refer to the [Cioban README](../cioban/README.md) for details on enabling automatic updates for services.

## Additional Resources

- [Docker Swarm Overview](https://docs.docker.com/engine/swarm/)
- [Traefik Documentation](https://doc.traefik.io/traefik/)
- [Ansible Documentation](https://docs.ansible.com/)




