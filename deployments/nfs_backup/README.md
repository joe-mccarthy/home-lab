# NFS Backup Solution

A comprehensive S3-backed encrypted backup solution for Docker Swarm NFS shared volumes using Restic.

## Overview

This project implements an enterprise-grade backup system for NFS shared volumes in a Docker Swarm environment. It leverages Restic, a modern backup tool that provides strong encryption, efficient deduplication, and configurable compression.

The solution consists of three specialized services:
1. **backup** - Creates automated hourly incremental backups of the NFS shared data
2. **prune** - Applies retention policies to optimize storage usage and costs
3. **check** - Performs regular integrity verification of the backup repository

## Architecture

The system runs as a stack on Docker Swarm with the following components:
- **Container images**: Uses `mazzolino/restic` for all services
- **Storage**: Backs up to S3-compatible object storage for durability
- **Orchestration**: Deployed via Ansible for consistent configuration
- **Security**: All data encrypted both in transit and at rest

## Prerequisites

- Docker Swarm cluster
- Node(s) with the `storage=true` label
- NFS shared volume mounted at `/exports/docker`
- Restore directory at `/restore`
- S3-compatible storage with proper credentials
- Ansible for deployment automation

## Configuration

### Environment Variables

All configuration is managed through Ansible variables with sensitive data stored in Ansible Vault:

- **S3 Configuration**:
  - `nfs_s3_bucket`: S3 bucket URL
  - `nfs_s3_region`: AWS/S3 region
  - `nfs_s3_key_id`: S3 access key ID
  - `nfs_s3_access_key`: S3 secret access key

- **Encryption**:
  - `nfs_encryption_key`: Password for encrypting all backup data

### Backup Schedule

- **Backup**: Runs hourly at minute 0 (`0 0/1 * * *`)
  - Example: 00:00, 01:00, 02:00, 03:00, etc.
- **Prune**: Runs daily at 3:30 AM (`30 3 * * *`)
  - Consolidates and removes backups according to retention policy
- **Check**: Runs Monday, Wednesday, and Friday at 8:30 AM (`30 8 * * 1,3,5`)
  - Verifies 10% of data for quick validation while minimizing resource usage

### Retention Policy

The default retention policy preserves:
- Last 24 snapshots (recent history protection)
- Daily snapshots for 7 days (weekly history)
- Weekly snapshots for 4 weeks (monthly history)
- Monthly snapshots for 4 months (quarterly history)

All snapshots are grouped by host for easier management and organization.

### Compression

Backups use maximum compression (`RESTIC_COMPRESSION: max`) to optimize storage usage and reduce data transfer costs when backing up to S3. This setting prioritizes storage efficiency over CPU utilization.

### Security

- All backups are encrypted using a key stored in Ansible Vault
- Data is read in read-only mode to prevent backup operations from modifying source data
- Services run with `no-new-privileges` security option to prevent privilege escalation
- S3 credentials use least privilege IAM permissions (ListBucket, PutObject, GetObject, DeleteObject)

**CRITICAL SECURITY WARNING:** Store the encryption key securely outside the system. If this key is lost, backups CANNOT be restored under any circumstances.

## Deployment

Deploy using Ansible:

```bash
# Regular deployment
ansible-playbook -i inventory/hosts deploy.yml

# With verbose output
ansible-playbook -i inventory/hosts deploy.yml -v

# To view changes without applying
ansible-playbook -i inventory/hosts deploy.yml --check
```

The playbook targets only the first Swarm manager node to prevent duplicate deployment commands.

## Manual Operations

### Listing Backups

```bash
docker exec restic restic snapshots
```

### Manual Backup

```bash
docker exec restic restic backup /exports/docker --tag manual-backup
```

### Restoring Files

To restore data:

```bash
# List available snapshots
docker exec restic restic snapshots

# Restore specific path from a snapshot
docker exec restic restic restore <snapshot-id> --include="/path/to/restore" --target="/restore"

# Restore entire snapshot
docker exec restic restic restore <snapshot-id> --target="/restore"
```

### Browsing Backups

To explore a backup snapshot without restoring:

```bash
docker exec -it restic restic mount <snapshot-id> /mnt
# Use a second shell to explore mounted files at /mnt
# Press Ctrl+C in the first shell when done
```

### Repository Maintenance

```bash
# Run integrity check
docker exec restic restic check

# Force repository cleanup
docker exec restic restic prune

# Rebuild repository index (if corrupted)
docker exec restic restic rebuild-index
```

## Monitoring and Troubleshooting

### Status Checks

```bash
# View service status
docker service ls | grep restic

# Check container logs for issues
docker service logs nfs_backup_backup
docker service logs nfs_backup_prune
docker service logs nfs_backup_check
```

### Common Issues and Solutions

1. **Connection errors to S3**:
   - Verify S3 credentials in Ansible Vault
   - Check network connectivity to S3 endpoint
   - Ensure IAM permissions are configured correctly

2. **Backup failures**:
   - Check if source directories exist and are accessible
   - Verify sufficient disk space for temporary files
   - Look for permissions issues in container logs

3. **Repository locks**:
   - If a backup operation was interrupted: `docker exec restic restic unlock`

4. **"Invalid repo" errors**:
   - May indicate repository corruption or encryption key mismatch
   - Verify correct encryption key is being used

## Resource Considerations

- **Storage**: Plan S3 storage based on data size, change rate, and retention period
- **Network**: Initial backups transfer all data; subsequent backups only transfer changes
- **CPU/Memory**: Compression level affects resource usage during backup operations

## Disaster Recovery Procedure

1. Deploy a fresh instance of the backup stack
2. Configure with the same S3 credentials and encryption key
3. Use restore commands to recover data

## License and Contribution

This project is maintained as part of the home-lab infrastructure. Contributions via pull requests are welcome.
