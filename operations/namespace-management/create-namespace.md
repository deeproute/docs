# Create a Namespace in Kubernetes (non-prod)

- [Create a Namespace in Kubernetes (non-prod)](#create-a-namespace-in-kubernetes-non-prod)
  - [Introduction](#introduction)
  - [Pre-Requisites](#pre-requisites)
  - [1. Add an Azure Role to the Team’s Security group.](#1-add-an-azure-role-to-the-teams-security-group)
  - [2. Store the namespace infrastructure code in argocd](#2-store-the-namespace-infrastructure-code-in-argocd)
    - [2.1. Create a new branch.](#21-create-a-new-branch)
    - [2.2. Create a new template folder for the namespace](#22-create-a-new-template-folder-for-the-namespace)
    - [2.3. Add the namespace configuration in values.yaml](#23-add-the-namespace-configuration-in-valuesyaml)
    - [2.4. Add the namespace configuration in value-[cluster-name].yaml](#24-add-the-namespace-configuration-in-value-cluster-nameyaml)
  - [3. Create the Kubernetes Deployment for the namespace](#3-create-the-kubernetes-deployment-for-the-namespace)
  - [4. Authorize Vault Access for the namespace](#4-authorize-vault-access-for-the-namespace)
  - [5. Authorize Jenkins Access to the namespace](#5-authorize-jenkins-access-to-the-namespace)

&nbsp;

## Pre-Requisites

- Have a security group already created.

&nbsp;

## 1. Add an Azure Role to the Team’s Security group.

Ensure if the security group is already assigned a role by checking in Azure Portal if there is a role name with this namespace.


**Syntax:**
```
docker run containerhub.internal.company.org:6666/azurecli:0.1.1 addRole.sh <lab\non-prod\prod> <description> <appRoleName> <adGroup> [--vault-only --k8s-only]
```

**Template (Namespace all in uppercase. Group should be always checked in the ticket):**
```
NS=WORLD-FIXER
GROUP="AZ-AAD-Kubernetes-$NS-NON-PROD"
docker run containerhub.internal.company.org:6666/azurecli:0.1.1 addRole.sh non-prod "Kubernetes $NS Users" $NS $GROUP 
```

&nbsp;

## 2. Store the namespace infrastructure code in argocd


### 2.1. Create a new branch (task-id/ns-name)


### 2.2. Create a new template folder for the namespace

| Type of project   | Folder Location                   |
| -------------     | -------------                     |
| Back Office       | /argocd/back-office/templates     |
| Data Science      | /argocd/data-science/templates    |

&nbsp;

**Syntax:**

```
{{$data := dict "Namespace" "[ns-name]" "Values" .Values "Description" "[Ticket Short Description] - [Ticket Owner Name]"}} 
{{ include "libchart.argocd.define.project" $data }} 
--- 
{{ include "libchart.argocd.namespace.creation" $data }} 
```

```
{{$data := dict "Namespace" "world-fixer" "Values" .Values "Description" "The project who fixes all world problems! - The Fixer"}} 
{{ include "libchart.argocd.define.project" $data }} 
--- 
{{ include "libchart.argocd.namespace.creation" $data }} 
```

&nbsp;

### 2.3. Add the namespace configuration in values.yaml


Make sure to update the `url`, `ad_role`, `istio`, `public` and `sources` according to the ticket.

```
world-fixer: 
  enabled: false 
  url: ssh://git@bitbucket-p.internal.company.org:7999/kd/world-fixer.git 
  branch: master 
  bootstrapChart: bootstrap 
  syncAccessGroups: [] 
  namespace: 
    ad_role: K8s-WORLD-FIXER 
    istio: enabled 
    public: view 
  sources: 
    - ssh://git@bitbucket-p.internal.company.org:7999/world-fixer/* 
```

&nbsp;

### 2.4. Add the namespace configuration in value-[cluster-name].yaml

Make sure to update `syncAccessGroups`, `cpu`, `memory` and `storage values` accordingly with the task.


```
world-fixer:
  enabled: true
  syncAccessGroups:
  - AZ-AAD-Kubernetes-WORLD-FIXER-NON-PROD
  namespace:
    dev_access: admin
    quotas:
      cpu: 50
      memory: 50Gi
      storage:
        nfs-client: 100Gi
        block-storage: 100Gi
```

&nbsp;

## 3. Create the Kubernetes Deployment for the namespace


## 4. Authorize Vault Access for the namespace

```
k config use-context [cluster-name]
./namespace-vault-auth.sh world-fixer default Vault-WORLD-FIXER-non-prod
```

&nbsp;

## 5. Authorize Jenkins Access to the namespace
