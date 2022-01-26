# Cheat Sheet for Helm

## Login to the desired cluster to refresh the id_token in kubeconfig

```
k oidc-login --kubeconfig=/home/homeuser/.kube/config.k8slab
```

## List Helm Charts
```
helm -n kube-system list \
      --kubeconfig ~/.kube/config.k8slab
```

## Template Helm Charts

```
helm template argocd . -f values.yaml -f values-env-lab.yaml -f values-k8slab.yaml
```

## Uninstall PV Dustman

```
helm -n kube-system uninstall argocd --kubeconfig ~/.kube/config.k8slab
```