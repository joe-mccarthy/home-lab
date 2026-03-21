# DDNS Deployment

[![Ansible](https://img.shields.io/badge/Ansible-Automation-EE0000?logo=ansible&logoColor=white&style=flat-square)](https://docs.ansible.com/) [![Docker Swarm](https://img.shields.io/badge/Docker%20Swarm-Service-2496ED?logo=docker&logoColor=white&style=flat-square)](https://docs.docker.com/engine/swarm/) ![Dynamic DNS](https://img.shields.io/badge/Dynamic%20DNS-Automation-1F6FEB?style=flat-square) [![Cloudflare](https://img.shields.io/badge/Cloudflare-DNS%20API-F38020?logo=cloudflare&logoColor=white&style=flat-square)](https://developers.cloudflare.com/dns/) ![cloudflare-ddns](https://img.shields.io/badge/cloudflare--ddns-v1.15.1-5C5C5C?style=flat-square)

This project provides an automated solution for deploying and managing a **Dynamic DNS (DDNS)** service using Ansible. The deployment is designed to ensure seamless updates of DNS records for dynamic IP addresses, enabling reliable access to services hosted on networks with non-static IPs.

## Overview

The deployment uses Ansible playbooks and roles to configure and manage the DDNS service. It integrates with DNS providers to update records dynamically based on the current IP address of the host. The solution is modular, secure, and easily customizable to fit various environments and DNS providers.

## Features

- **Dynamic DNS Updates**: Automatically updates DNS records for hosts with dynamic IP addresses.
- **Provider Integration**: Supports integration with popular DNS providers via their APIs.
- **Secure Secrets Management**: Uses Ansible Vault to securely store API tokens and other sensitive data.
- **Modular Design**: Simplifies maintenance and customization with dedicated roles for tasks like configuration and deployment.
- **Scalable**: Easily extendable to manage multiple domains or subdomains.

## Prerequisites

To deploy the DDNS service, you need Ansible installed on the control machine and access to the target host. API credentials for the DNS provider must be configured and encrypted using Ansible Vault.

## Deployment Process

The deployment process involves configuring the required variables, securely storing secrets, and running the Ansible playbook to set up the DDNS service. The playbook ensures that the service is properly configured and running, with periodic updates to DNS records.

## Security Considerations

The deployment emphasizes security by encrypting sensitive data using Ansible Vault. It is recommended to use restrictive permissions for configuration files and to test the deployment in a staging environment before applying it to production.

## Troubleshooting

Common issues such as misconfigured API credentials or DNS provider errors can be resolved by reviewing the Ansible playbook logs and verifying the configuration settings.