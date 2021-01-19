## Configure Access to Multiple Cluster
---
### Cluster info
- Minikube
- Dev-cluster
### Basics
To connect to kubernetes clusters we need to define 3 things:
- Cluster
- User
- Context

> Notes: Kubernetes support multiple virtual clusters backed by the same physical cluster called [namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/). In this configuration is referred as context(cluster, user, namespace).

### 1. Add cluster details
- Minikube:

`$ kubectl config --kubeconfig=config-minikube set-cluster minikube --server=https://192.168.100.14:8443 --certificate-authority=$HOME/.minikube/ca.crt`

- Dev-cluster:

`$ kubectl config --kubeconfig=config-dev-cluster set-cluster dev-cluster --server=https://192.168.14.10:6443`

`$ kubectl config --kubeconfig=config-dev-cluster set clusters.dev-cluster.certificate-authority-data <ca_base64_encoded_data>`

### 2. Add user details
- Minikube:

`$ kubectl config --kubeconfig=config-minikube set-credentials minikube --client-certificate=$HOME/.minikube/client.crt --client-key=$HOME/.minikube/client.key`
- Dev-cluster:

`$ kubectl config --kubeconfig=config-dev-cluster set users.kubernetes-admin.client-certificate-data <client_cert_base64_encoded_data>`

`$ kubectl config --kubeconfig=config-dev-cluster set users.kubernetes-admin.client-key-data <client_key_base64_encoded_data>`

### 3. Add context details
- Minikube:

`$ kubectl config --kubeconfig=config-minikube set-context minikube --cluster=minikube --namespace=default --user=minikube`
- Dev-cluster:

`$ kubectl config --kubeconfig=config-dev-cluster set-context kubernetes-admin@dev-cluster --cluster=dev-cluster --namespace=default --user=kubernetes-admin`

### 4. Show configuration details
- Minikube:

`$ kubectl config --kubeconfig=config-minikube view`
- Dev-cluster:

`$ kubectl config --kubeconfig=config-dev-cluster view`
### 5. Set the current context
- Minikube:

`$ kubectl config --kubeconfig=config-minikube use-context minikube`
`$ kubectl config --kubeconfig=config-minikube view --minify`
- Dev-cluster:

`$ kubectl config --kubeconfig=config-dev-cluster use-context kubernetes-admin@dev-cluster`
`$ kubectl config --kubeconfig=config-dev-cluster view --minify`
### 6. Set the KUBECONFIG environment variable
`$ export KUBECONFIG=$KUBECONFIG:config-minikube:config-dev-cluster:$HOME/.kube/config`
Or add it in ~/.bashrc so it's available when you log in

### 7. Show all managed contexts
`$ kubectl config get-contexts`
```
NAME          STATUS   ROLES                  AGE    VERSION
masterk8s     Ready    control-plane,master   2d8h   v1.20.2
worker-1k8s   Ready    <none>                 2d8h   v1.20.2
worker-2k8s   Ready    <none>                 2d8h   v1.20.2
```

### References:
- https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/
- https://zepworks.com/posts/access-minikube-remotely-kvm/