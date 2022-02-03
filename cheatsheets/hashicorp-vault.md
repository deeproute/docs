# Vault CheatSheet

## Raft Cluster

### Raft Configuration

https://www.vaultproject.io/api-docs/system/storage/raft#read-raft-configuration

```sh
curl \
    --header "X-Vault-Token: ..." \
    http://127.0.0.1:8200/v1/sys/storage/raft/configuration
```

### Snapshot/Restore

https://www.vaultproject.io/api-docs/system/storage/raft#take-a-snapshot-of-the-raft-cluster

```sh
curl \
    --header "X-Vault-Token: ..." \
    --request GET \
    http://127.0.0.1:8200/v1/sys/storage/raft/snapshot > raft.snap
```

https://www.vaultproject.io/api-docs/system/storage/raft#restore-raft-using-a-snapshot

```sh
curl \
    --header "X-Vault-Token: ..." \
    --request POST \
    --data-binary @raft.snap \
    http://127.0.0.1:8200/v1/sys/storage/raft/snapshot
```

## Authorization

### List all groups in Vault

```sh
vault list identity/group/name/
```