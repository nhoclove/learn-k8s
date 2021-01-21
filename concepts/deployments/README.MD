## Deployment
---
An abstraction over `Pods` and `ReplicaSet`
> A Deployment provides declarative updates for Pods and ReplicaSets

### Create a Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: # The deployment name
  labels: # key-value pairs for label
spec:
  replicas: # Replica number
  selector: # Defind how the Deployment find which Pods to manage
  template: # Pod template
```
`$ kubectl apply -f concepts/deployments/nginx-deployment.yaml`
### Update a Deployment