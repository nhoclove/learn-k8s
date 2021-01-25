# Deploying Cassandra with StatefulSet

## 1. Create Headless Service
`$ kubectl apply -f cassandra-service.yaml`
## 2. Create Cassandra ring
`$ kubectl apply -f cassandra-statefulset.yaml`
