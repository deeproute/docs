# Ansible CheatSheet

## Run using python module
```
ansible all -i inventories/k8slab/hosts --private-key ~/.ssh/id_rsa_k8slab -m ping
```


## Run without python module
```
ansible all -i inventories/k8slab/hosts --private-key ~/.ssh/id_rsa_k8slab -m raw -a "uptime"
ansible all -i inventories/k8slab/hosts --private-key ~/.ssh/id_rsa_k8slab -m raw -a "shutdown -r now"
ansible all -i inventories/k8slab/hosts --private-key ~/.ssh/id_rsa_k8slab -m raw -a "uptime"

```

## Run as root with --become
```
ansible all -i inventories/k8slab/hosts --private-key ~/.ssh/id_rsa_k8slab --become -m shell -a "uptime"
ansible all -i inventories/k8slab/hosts --private-key ~/.ssh/id_rsa_k8slab --become -m shell -a "shutdown -r now"
```

## Remove all k8s related folders
```
ansible all -i inventories/k8slab/hosts --private-key ~/.ssh/id_rsa_k8slab --become -m raw -a "rm -rf /etc/kubernetes/* && rm -rf /var/lib/etcd/* && rm -rf /var/lib/kubelet/* && rm -rf /opt/install_python && rm -rf /opt/bin/active_python && rm -rf /opt/bin/python* && rm -rf /opt/bin/pyp && rm /opt/bin/kubeadm && rm /opt/bin/kubectl && rm /opt/bin/kubelet"
```

## Run Playbooks
```
ansible-playbook -i inventories/k8slab/hosts --private-key ~/.ssh/id_rsa_k8slab --become destroy-cluster.yaml
ansible-playbook -i inventories/k8slab/hosts --private-key ~/.ssh/id_rsa_k8slab --become install-cluster.yaml --extra-vars "docker_password='1234556' vault_token='s.xxxxxxxxxxx'"


```

