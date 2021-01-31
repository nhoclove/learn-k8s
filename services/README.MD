# Services
> Expose an application running in Pods to outside of your cluster

## Expose whoami service
1. Create whoami service

    `$ kubectl apply -f services/whoami-services.yaml`

2. Get service status

    `$ kubectl get svc`
    ```
    NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
    kubernetes       ClusterIP   10.96.0.1       <none>        443/TCP    10d
    whoami-service   ClusterIP   10.101.31.236   <none>        8080/TCP   2m30s
    ```
3. Test whoami service

    `$ curl -v http://192.168.49.2:30001/whoami`
    ```
    *   Trying 192.168.49.2:30001...
    * TCP_NODELAY set
    * Connected to 192.168.49.2 (192.168.49.2) port 30001 (#0)
    > GET /whoami HTTP/1.1
    > Host: 192.168.49.2:30001
    > User-Agent: curl/7.68.0
    > Accept: */*
    > 
    * Mark bundle as not supporting multiuse
    < HTTP/1.1 200 OK
    < Date: Sat, 30 Jan 2021 17:24:59 GMT
    < Content-Length: 104
    < Content-Type: text/plain; charset=utf-8
    < 
    {"host":"whoami-deployment-7999bd8c6b-djmxt","ip":"172.17.0.3","time":"2021-01-30T17:24:59.233461683Z"}
    * Connection #0 to host 192.168.49.2 left intact
    ```
## Access the Service
- Environment variables
- DNS

## Expose the Service
- NodePorts
- LoadBalancers