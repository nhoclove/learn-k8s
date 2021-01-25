# StatefulSets
## 1. Stateful application
Applications which save/remember information about the previous operations such as:
user details(profile, preferences, actions...)
## 2. Characteristics of stateful application
- Stable, unique network identifiers
- Stable, persistent storage
- Ordered, graceful deployment and scaling
- Ordered, automated rolling updates
## 3. Scaling a StatefulSet
### 1. Scaling Up
`$ kubectl scale sts web --replicas=5`
### 2. Scaling Down
`$ kubectl patch sts web -p '{"spec":{"replicas":3}}'`
## 4. References:
- https://www.xenonstack.com/insights/stateful-and-stateless-applications/
- https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/
- https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/