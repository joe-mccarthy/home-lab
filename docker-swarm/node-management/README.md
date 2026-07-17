# 🐳 Docker Swarm Node Management

Focused Ansible runbooks for changing Docker Swarm membership one node at a
time. Use these playbooks when you want to add, drain, demote, or remove a
single node without running the full cluster create or destroy flow.

> [!IMPORTANT]
> Run every command from the repository root so `ansible.cfg`, inventory, vault
> settings, and relative role paths resolve consistently.

---

## ✨ What These Playbooks Do

| Action | Playbook | Target group | Result |
| --- | --- | --- | --- |
| ➕ Add manager | `add-manager.yml` | `manager` | Mounts NFS, installs Docker, joins as manager, applies labels |
| ➕ Add worker | `add-worker.yml` | `docker` | Mounts NFS, installs Docker, joins as worker, applies labels |
| ➖ Remove manager | `remove-manager.yml` | `manager` | Drains, demotes, leaves swarm, removes node entry |
| ➖ Remove worker | `remove-worker.yml` | `docker` | Drains, leaves swarm, removes node entry |

---

## 🧭 Before You Run

- 🗂️ Keep the target host in inventory for the whole operation.
- 🧑‍✈️ Put manager nodes in the `manager` group.
- 🧰 Put worker nodes in the `docker` group.
- 🧱 Keep all Swarm nodes under the `cluster` group.
- 💾 Keep the NFS server in `nfs_servers`.
- 🔑 Make sure SSH works for the target and the control manager.
- 🗳️ For manager removal, confirm quorum will remain after the change.

The first host in `manager` is used as the control manager by default. If that
host is the node being changed, pass `swarm_manager=<existing-manager>`.

> [!TIP]
> Keep the inventory change and the playbook run together. Add a new host before
> running an add playbook, and remove a decommissioned host only after the remove
> playbook succeeds.

---

## ⚙️ Variables

| Variable | Required | Default | Purpose |
| --- | --- | --- | --- |
| `target_node` | ✅ Yes | None | Inventory hostname to add or remove |
| `swarm_manager` | No | `groups['manager'][0]` | Existing manager used for join tokens and node operations |
| `swarm_manager_addr` | No | `<swarm_manager default IPv4>:2377` | Join address advertised to new nodes |
| `swarm_node_name` | No | Target inventory short hostname | Docker Swarm node name, when it differs from inventory |
| `skip_node_leave` | No | `false` | Skip SSH to an offline target during removal |

---

## 🚀 Add A Manager

Add the new host to the `manager` group in `inventory.yml`, then run:

```bash
ansible-playbook -i inventory.yml docker-swarm/node-management/add-manager.yml \
  -e target_node=<new-manager-hostname> \
  --ask-vault-pass
```

If the new host is first in the `manager` group, choose an existing manager for
the control-plane work:

```bash
ansible-playbook -i inventory.yml docker-swarm/node-management/add-manager.yml \
  -e target_node=<new-manager-hostname> \
  -e swarm_manager=<existing-manager-hostname> \
  --ask-vault-pass
```

---

## 🧰 Add A Worker

Add the new host to the `docker` group in `inventory.yml`, then run:

```bash
ansible-playbook -i inventory.yml docker-swarm/node-management/add-worker.yml \
  -e target_node=<new-worker-hostname> \
  --ask-vault-pass
```

---

## 🧑‍✈️ Remove A Manager

Keep the manager in inventory for the removal run:

```bash
ansible-playbook -i inventory.yml docker-swarm/node-management/remove-manager.yml \
  -e target_node=<manager-hostname> \
  --ask-vault-pass
```

If the target is first in the `manager` group, use another active manager for
control-plane operations:

```bash
ansible-playbook -i inventory.yml docker-swarm/node-management/remove-manager.yml \
  -e target_node=<manager-hostname> \
  -e swarm_manager=<existing-manager-hostname> \
  --ask-vault-pass
```

After a successful run, remove the host from inventory if it is being
decommissioned.

> [!WARNING]
> Do not remove the last manager. For steady-state clusters, prefer an odd
> number of managers and confirm the remaining managers are healthy before
> changing membership.

---

## 🧹 Remove A Worker

Keep the worker in inventory for the removal run:

```bash
ansible-playbook -i inventory.yml docker-swarm/node-management/remove-worker.yml \
  -e target_node=<worker-hostname> \
  --ask-vault-pass
```

After a successful run, remove the host from inventory if it is being
decommissioned.

---

## 📴 Offline Targets

If the target is already offline, skip the SSH step that asks the node to leave
the swarm and force-remove the node entry from a manager:

```bash
ansible-playbook -i inventory.yml docker-swarm/node-management/remove-worker.yml \
  -e target_node=<worker-hostname> \
  -e skip_node_leave=true \
  --ask-vault-pass
```

The same option works for manager removal:

```bash
ansible-playbook -i inventory.yml docker-swarm/node-management/remove-manager.yml \
  -e target_node=<manager-hostname> \
  -e swarm_manager=<existing-manager-hostname> \
  -e skip_node_leave=true \
  --ask-vault-pass
```

> [!CAUTION]
> Offline manager removal still changes manager membership and can affect
> quorum. Use an active manager for `swarm_manager` and verify the cluster state
> afterwards.

---

## ✅ Verification

Check membership from a manager after each run:

```bash
ansible -i inventory.yml <manager-hostname> -b -a "docker node ls"
```

For add operations, verify the new node is `Ready` and `Active`.

For remove operations, verify the removed node no longer appears in
`docker node ls`, then update inventory if the host is gone for good.
