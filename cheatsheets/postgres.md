# Postgres CheatSheet


## Resizing a Postgres operator cluster (outdated)
Change the size in git first.

```
k -n <namespace> patch pvc pgdata-db-cluster-{0..1} -p '{"spec": {"resources": {"requests": {"storage": "500Gi"}}}}'

k -n <namespace> get sts db-cluster -oyaml > /tmp/db-cluster-backup.yaml

k -n <namespace> delete sts db-cluster --cascade=false
```