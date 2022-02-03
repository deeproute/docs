# Hashicorp Vault

## Vault HA Server with Raft

### Install using helm chart

- values.yaml

```yaml
injector:
    enabled: false

server:
    affinity: "" # Use only for dev
    standalone:
        enabled: false
    ha:
        enabled: true
        replicas: 3
        raft:
            enabled: true
            setNodeId: true
        dataStorage:
            size: 100Mi
            storageClass: standard
        ingress:
            enabled: true
            ingressClassName: "nginx"
            pathType: Prefix
            activeService: true
            hosts:
                - host: vault-server.vault
                paths: []
            extraPaths: []
            tls: []

ui:
    enabled: true
```

Install it with helm:

```sh
helm repo add hashicorp https://helm.releases.hashicorp.com

helm install vault-server hashicorp/vault -f values.yaml
```

### 2. Setup Vault (without auto-unseal)

- When the chart is installed the pods will run with `not ready` status. To fix this we need to `initialize` & `unseal` vault in all the pods one by one.

- To use the [Vault UI](http://localhost:8200/ui/vault/init), you need to port forward directly to the pod and follow the init steps:

```sh
k -n vault port-forward pod/vault-server-0 8200:8200
```

- Save all the keys and token in a secure place. You will need to do the same for the remaining pods.

- Join the other pods to the raft cluster:

```sh
k exec -n vault vault-server-1 -- vault operator raft join http://vault-server-0.vault-server-internal:8200

k exec -n vault vault-server-2 -- vault operator raft join http://vault-server-0.vault-server-internal:8200
```

- `Unseal` all the pods. If the key threshold is 3 then you need to do 3 times unseal on each of them:

```sh
# run this 3 times with different keys
k exec -n vault vault-server-0 -- vault operator unseal
Unseal Key (will be hidden): [copy paste one of the keys here]

```

## Vault CSI Driver [link](https://learn.hashicorp.com/tutorials/vault/kubernetes-secret-store-driver?in=vault/kubernetes)


### Install Secret Store CSI Driver

1. Create a `values.yaml`

```yaml
syncSecret:
  enabled: true
enableSecretRotation: true
```

2. Install it with helm chart

```sh
helm repo add secrets-store-csi-driver https://raw.githubusercontent.com/kubernetes-sigs/secrets-store-csi-driver/master/charts

helm install csi secrets-store-csi-driver/secrets-store-csi-driver -f values.yaml
```

### Install Vault CSI

1. Create a `values.yaml`

```yaml
server:
  enabled: false
csi:
  enabled: true
```

2. Install it with helm chart

```sh
helm repo add hashicorp https://helm.releases.hashicorp.com

helm install vault-csi hashicorp/vault -f values.yaml
```


### Enable Kubernetes Auth so the pods can connect to Vault

- Enable kubernetes auth:

```sh
kubectl exec -it vault-0 -- /bin/sh

vault auth enable kubernetes

vault write auth/kubernetes/config \
    issuer="https://kubernetes.default.svc.cluster.local" \
    token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
    kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443" \
    kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
```

### Create a policy & role for your app

```sh
vault policy write internal-app - <<EOF
path "secret/data/db-pass" {
  capabilities = ["read"]
}
EOF

vault write auth/kubernetes/role/database \
    bound_service_account_names=webapp-sa \
    bound_service_account_namespaces=default \
    policies=internal-app \
    ttl=20m
```

### Create a sample app

1. Create a Secret:
```sh
vault kv put secret/db-pass password="db-secret-password"
```

2. Define the SecretProviderClass

```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1alpha1
kind: SecretProviderClass
metadata:
  name: vault-database-secret
spec:
  provider: vault
  parameters:
    vaultAddress: "http://vault-server.vault-server:8200"
    roleName: "database"
    objects: |
      - secretPath: "secret/data/db-pass"
        objectName: "db-password"
        secretKey: "password"
  secretObjects:
  - secretName: "db-password"
    type: Opaque
    labels:
      env: "test"
    data:
    - objectName: "db-password"
      key: "password"
```

```sh
kubectl apply -f spc-vault-database.yaml
```

3. Create pod with secret mounted:

```sh
kubectl create serviceaccount webapp-sa
```

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: webapp
spec:
  serviceAccountName: webapp-sa
  containers:
  - image: jweissig/app:0.0.1
    name: webapp
    volumeMounts:
    - name: secrets-store-inline
      mountPath: "/mnt/secrets-store"
      readOnly: true
  volumes:
    - name: secrets-store-inline
      csi:
        driver: secrets-store.csi.k8s.io
        readOnly: true
        volumeAttributes:
          secretProviderClass: "vault-database"
```

```sh
kubectl apply -f webapp-pod.yaml
```

4. Test if the secret is really mounted

```sh
kubectl exec webapp -- cat /mnt/secrets-store/db-password
```