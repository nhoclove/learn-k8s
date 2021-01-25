# DaemonSet

## 1. Why
Some nodes or all nodes need to run a copy of a Pod for:
- Running a cluster storage daemon on every node
- Running a logs collection daemon on every node
- Running a node monitoring daemon on every node

## 2. Run
`$ kubectl apply -f fluentd-elasticsearch-daemonset.yaml`