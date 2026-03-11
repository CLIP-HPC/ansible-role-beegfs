# vbc.beegfs

Deploy and manage a BeeGFS cluster (management, metadata, storage, client, and optional monitoring) with Ansible.

## Example

Inventory groups expected by this role:

```ini
[cluster_beegfs_mgmt]
bgfs1

[cluster_beegfs_mds]
bgfs1

[cluster_beegfs_oss]
bgfs1

[cluster_beegfs_client]
bgfs1
```

Playbook example aligned with the current Molecule converge scenario:

```yaml
---
- name: Deploy BeeGFS
  hosts: all
  roles:
    - role: vbc.beegfs
      beegfs_enable:
        mon: false
        mgmt: "{{ inventory_hostname in groups['cluster_beegfs_mgmt'] }}"
        meta: "{{ inventory_hostname in groups['cluster_beegfs_mds'] }}"
        oss: "{{ inventory_hostname in groups['cluster_beegfs_oss'] }}"
        tuning: "{{ inventory_hostname in groups['cluster_beegfs_oss'] }}"
        client: "{{ inventory_hostname in groups['cluster_beegfs_client'] }}"
      beegfs_oss:
        8003:
          devices: ["/dev/sdb"]
      beegfs_client_mounts:
        - path: "/mnt/beegfs"
          port: 8004
      beegfs_meta_dev: "/dev/sdc"
      beegfs_oss_max_sectors_kb: 128
      beegfs_auth_secret: "<set-a-secret>"
      beegfs_disable_tls: true
      beegfs_meta_max_sectors_kb: 128
      beegfs_mgmt_host: "{{ groups['cluster_beegfs_mgmt'] | first }}"
      beegfs_fstype: "xfs"
      beegfs_force_format: false
      beegfs_interfaces: ["enp0s8"]
      beegfs_rdma: false
```

Create:

```bash
ansible-playbook beegfs.yml -i inventory-beegfs -e beegfs_state=present
```

Destroy:

```bash
ansible-playbook beegfs.yml -i inventory-beegfs -e beegfs_state=absent
```

## Development Setup (uv)

After cloning the repository, install Python dependencies with uv:

```bash
uv venv
uv sync
```

Run tooling through uv:

```bash
uv run molecule test
```

## Key Variables

Service toggles:

- `beegfs_enable.mgmt`
- `beegfs_enable.meta`
- `beegfs_enable.oss`
- `beegfs_enable.client`
- `beegfs_enable.mon`
- `beegfs_enable.tuning`

Core behavior:

- `beegfs_state` (`present` or `absent`)
- `beegfs_mgmt_host`
- `beegfs_interfaces`
- `beegfs_rdma`
- `beegfs_add_repos`
- `beegfs_update`

Storage and metadata:

- `beegfs_oss` (map keyed by storage service port with `devices`)
- `beegfs_oss_path_prefix`
- `beegfs_oss_tunable`
- `beegfs_fstype`
- `beegfs_filesystem_opts`
- `beegfs_mount_opts`
- `beegfs_force_format`
- `beegfs_meta_dev`
- `beegfs_meta_path`
- `beegfs_meta_fstype`

Client and mount:

- `beegfs_client_mounts` (list of mount entries with `path` and `port`)
- `beegfs_enable_quota`
- `beegfs_client_scope_config`

Security and tuning:

- `beegfs_auth_secret`
- `beegfs_disable_tls`
- `beegfs_meta_tune_num_workers`
- `beegfs_oss_tune_num_workers`
- `beegfs_conn_rdma_buf_num`
- `beegfs_conn_rdma_buf_size`
- `beegfs_conn_meta_max_internode_num`
- `beegfs_conn_oss_max_internode_num`
- `beegfs_conn_client_max_internode_num`

## Testing

This role includes a Molecule scenario under `molecule/default`.

Vagrant driver requirements:

- Vagrant
- VirtualBox, Parallels, VMware Fusion, VMware Workstation, or VMware Desktop

```bash
molecule test
```
