# Kubernetes CheatSheet

## Cluster Management


### Run commands in multiple clusters

```
(
    clusters="context1 context2 context3 context4 context5 context6 context7 context8 context9";
    for cluster in "${clusters}"
    do
        echo "--- $cluster ---"
        k config use-context "${cluster}"
        
        <some command>
    done
)
```

### Upgrades

```
kubeadm upgrade diff

[upgrade/diff] Reading configuration from the cluster...
[upgrade/diff] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[controlplane] Adding extra host path mount "audit-policy" to "kube-apiserver"
[controlplane] Adding extra host path mount "vsphere" to "kube-apiserver"
[controlplane] Adding extra host path mount "vsphere" to "kube-controller-manager"
W0210 13:56:20.125641   14586 manifests.go:214] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"

```

## Manage Deployments


### Scale all deployments based on a label in a namespace

```
k get deploy -n <namespace> -l <label> -o name | xargs -I % kubectl scale % --replicas=<num-replicas> -n <namespace>
```

### Set env variable
```
kubectl set env daemonset/calico-node -n kube-system IP_AUTODETECTION_METHOD=interface=vlan9999
```

## Managing Pods

### Filter by Failed Pods
```
k get pod -n <namespace> --field-selector=status.phase==Failed
```

### Get only the pod name

```
k get pod -n <namespace> --field-selector=status.phase==Failed | awk '{print $1}'
```

### Delete failed pods

```
k get pod -n <namespace> --field-selector=status.phase==Failed | awk '{print $1}' | xargs k -n <namespace> delete pod
```

### Find all pods that are not fully running (eg some containers are failing)

k get po --all-namespaces | gawk 'match($3, /([0-9])+\/([0-9])+/, a) {if (a[1] < a[2] && $4 != "Completed") print $0}'

## Managing Storage

### Make the PV available to be reattached to a PVC.
```
kubectl patch pv <pvname> -p '{"spec":{"claimRef": null}}'
```

### Fix PVC stuck in terminating

```
kubectl patch pvc -n <namespace> <pvc-name> -p '{"metadata":{"finalizers":null}}'
```

### Reclaim all released PVs

```
k get pv | grep -i released | cut -d " " -f 1 | xargs -I{} kubectl patch pv {} -p '{"spec":{"persistentVolumeReclaimPolicy":"Delete"}}'
```

## Namespaces

### Check why namespace stuck terminating

```
k get namespace "stucked-namespace" -o json | jq '.spec = {"finalizers":[]}' > temp.json
```

### Reset the namespace finalizer

```
k get namespace "stucked-namespace" -o json \
  | tr -d "\n" | sed "s/\"finalizers\": \[[^]]\+\]/\"finalizers\": []/" \
  | kubectl replace --raw /api/v1/namespaces/stucked-namespace/finalize -f -
```

## Security

### Get all rolebindings and associated service accounts

```
kubectl get rolebindings,clusterrolebindings \
--all-namespaces  \
-o custom-columns='KIND:kind,NAMESPACE:metadata.namespace,NAME:metadata.name,SERVICE_ACCOUNTS:subjects[?(@.kind=="ServiceAccount")].name'
```

### Check authorization for ServiceAccounts

https://kubernetes.io/docs/concepts/policy/pod-security-policy/

```
alias kubectl-admin='kubectl -n psp-example'
alias kubectl-user='kubectl --as=system:serviceaccount:psp-example:fake-user -n psp-example'
```

```
kubectl auth can-i <verb> <resource> --as=system:serviceaccount:<namespace>:<serviceaccountname> [-n <namespace>]
```

### Check authorization for a group

```
k auth can-i get secrets -n <namespace> --as dummy --as-group=K8s-Namespace
yes

k auth can-i get secrets -n <namespace> --as dummy --as-group=K8s-Namespace
no
```

### Create a rolebinding from a clusterrole.

```
k create ns policy-demo
k create rolebinding psp --clusterrole=psp:privileged --serviceaccount=policy-demo:testsa -n policy-demo
```