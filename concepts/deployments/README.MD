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
#### 1. Update the Pod's template
`$ kubectl --record deployment.apps/nginx-deployment set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1`

or

`$ kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1 --record`

or update in `deployment.yaml` file

`$ kubectl edit deployment.v1.apps/nginx-deployment`
#### 2. See the rollout status
`$ kubectl rollout status deployment/nginx-deployment`

### Roll back a Deployment
#### 1. Check Rollout history
`$ kubectl rollout history deployment.v1.apps/nginx-deployment`
#### 2. See the details of each revision
`$ kubectl rollout history deployment.v1.apps/nginx-deployment --revision=2`
#### 3. Rollback to the previous revision
`kubectl rollout undo deployment.v1.apps/nginx-deployment --to-revision=2`
### Deployement status
`progressing` --> `complete` or `fail to progress`