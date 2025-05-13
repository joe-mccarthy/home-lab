# NFS Backup Solution

A comprehensive automated backup solution for Docker Swarm NFS shared volumes using restic.

## Overview

This project provides an automated backup system for NFS shared volumes used in a Docker Swarm environment. It uses restic, a modern, efficient backup tool with strong encryption, deduplication, and compression features.

The solution deploys three coordinated services:
1. **backup** - Creates regular incremental backups of the NFS shared volumes
2. **prune** - Manages backup retention according to defined policies
3. **check** - Verifies backup repository integrity

## Architecture

The system is designed to run on a Docker Swarm cluster with designated storage nodes, backing up critical NFS-shared data that's used by various services.

## Prerequisites

- Docker Swarm cluster
- Node(s) labeled with `storage=true`
- NFS shared volume mounted at `/exports/docker`
- Local backup storage at `/backups`
- Restore directory at `/restore`
- Ansible for deployment

## Configuration

### Backup Schedule

- **Backup**: Runs every 2 hours at 45 minutes past the hour (`45 1/2 * * *`)
  - Example: 01:45, 03:45, 05:45, 07:45, etc.
- **Prune**: Runs 15 minutes past every even hour (`15 0/2 * * *`) 
  - Example: 00:15, 02:15, 04:15, 06:15, etc.
- **Check**: Runs on the same schedule as backup service (`15 1/2 * * *`)
  - Example: 01:15, 03:15, 05:15, 07:15, etc.

### Retention Policy

The default retention policy keeps:
- Last 12 snapshots (recent work protection)
- Daily snapshots for 7 days (weekly history)
- Weekly snapshots for 4 weeks (monthly history)
- Monthly snapshots for 6 months (half-year history)

These settings are grouped by host for more logical organization.

### Compression

Backups use maximum compression (`RESTIC_COMPRESSION: max`) to reduce storage requirements. This provides better space efficiency at the cost of slightly increased CPU usage during backup operations.

### Encryption

Backups are encrypted using a password stored in Ansible Vault. The key is referenced as `file_encryption_key` in `group_vars/all.yml`.

**IMPORTANT:** Store this encryption key securely outside the system. If the key is lost, backups cannot be restored.

## Deployment

Deploy the backup system using Ansible:

```bash
ansible-playbook -i inventory/hosts deploy.yml
```

## Manual Operations

### Listing Backups

```bash
docker exec restic restic snapshots
```

### Manual Backup

```bash
docker exec restic restic backup /exports/docker
```

### Restoring Files

To restore a specific file or directory:

```bash
# Find the snapshot ID
docker exec restic restic snapshots

# Restore to the tmp-for-restore directory
docker exec restic restic restore [SNAPSHOT_ID] --include="/path/to/restore" --target="/tmp-for-restore"
```

The restored files will be available in `/backups/tmp-for-restore` on the host.

### Mounting a Backup

To browse a backup snapshot:

```bash
docker exec -it restic restic mount [SNAPSHOT_ID] /mnt
```

### Checking Repository Integrity

```bash
docker exec restic restic check
```

## Monitoring

Check the status of backup services:

```bash
docker service ls | grep restic
```

View logs from backup service:

```bash
docker service logs nfs_backup_restic_backup
```

## Troubleshooting

### Common Issues

1. **Service won't start**: Check if the backup directory exists and has proper permissions
2. **Backup fails**: Verify that the source directory is properly mounted
3. **Out of space**: Check available space in `/backups`

### Restic Repository Repair

If the repository becomes corrupted:

```bash
docker exec -it restic restic rebuild-index
```

## Security Considerations

- Encryption key is stored in Ansible Vault
- Services run with `no-new-privileges` security option
- Source volumes are mounted read-only for backup
- All services use non-privileged users

## License

This project is licensed under the MIT License - see the LICENSE file for details.
