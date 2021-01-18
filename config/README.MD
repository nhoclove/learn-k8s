## Configure Access to Multiple Cluster

### 1. Add cluster details to the configuration file
`$ kubectl config --kubeconfig=config-minikube set-cluster minikube --server=https://192.168.100.14:8443 --certificate-authority=$HOME/.minikube/ca.crt`

### 2. Add user details to the configuration file
`$ kubectl config --kubeconfig=config-minikube set-credentials minikube --client-certificate=$HOME/.minikube/client.crt --client-key=$HOME/.minikube/client.key`

### 3. Add context details to the configuration file
`$ kubectl config --kubeconfig=config-minikube set-context minikube --cluster=minikube --namespace=default --user=minikube`
### 4. Show configuration details
`$ kubectl config --kubeconfig=config-minikube view`

### 5. Set the current context
`$ kubectl config --kubeconfig=config-minikube use-context minikube`
`$ kubectl config --kubeconfig=config-minikube view --minify`

### 6. Set the KUBECONFIG environment variable
`$ export KUBECONFIG=$KUBECONFIG:config-minikube`
`$ export KUBECONFIG=$KUBECONFIG:$HOME/.kube/config`
### 7. Show all managed contexts
`$ kubectl config get-contexts`