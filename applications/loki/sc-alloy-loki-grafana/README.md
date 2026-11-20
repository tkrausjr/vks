This will show Supervisor fluentbit--> Alloy --> Loki --> Grafana. Alloy's loki.source.syslog component parses RFC 5424 natively, over TCP or UDP. 

Choose a cluster to house Loki and Alloy for log collection and expose them outside the cluster (optional).
NOTE: We already have grafana running so do not need to configure these.

1. Add the helm repo for grafana.

```bash
helm repo add grafana https://grafana.github.io/helm-charts
    "grafana" has been added to your repositories

helm repo update                                           
    Hang tight while we grab the latest from your chart repositories...
    ...Successfully got an update from the "grafana" chart repository
    Update Complete. ⎈Happy Helming!⎈

helm search repo grafana/loki
    NAME                            CHART VERSION   APP VERSION     DESCRIPTION                                       
    grafana/loki                    7.7.0           3.6.20          Helm chart for Grafana Enterprise Logs supporti...
    grafana/loki-canary             0.14.1          2.9.1           Helm chart for Grafana Loki Canary                
    grafana/loki-distributed        0.80.6          2.9.13          Helm chart for Grafana Loki in microservices mode 
    grafana/loki-simple-scalable    1.8.11          2.6.1           Helm chart for Grafana Loki in simple, scalable...
    grafana/loki-stack              2.10.3          v2.9.3          Loki: like Prometheus, but for logs.   

```

2. Install Alloy
```bash
k create ns logging                         
    namespace/logging created

helm install alloy grafana/alloy -n logging -f ./applications/loki/alloy-loki-grafana/alloy-values.yaml
    NAME: alloy
    LAST DEPLOYED: Thu Nov 19 20:44:51 2026
    NAMESPACE: logging
    STATUS: deployed
    REVISION: 1
    DESCRIPTION: Install complete
    TEST SUITE: None
    NOTES:
    Welcome to Grafana Alloy!

The EXTERNAL-IP of the alloy Service is the address your devices send syslog to. In your lab, the load balancer provides it. 
```
3. 

4. Install Loki 

Use the loki-values.yaml and existing Grafana install. Change storageClass as needed.

```bash
helm install loki grafana/loki -n logging --create-namespace -f ./applications/loki/alloy-loki-grafana/loki-values.yaml 

kubectl -n logging patch svc loki -p '{"spec":{"type":"LoadBalancer"}}'
    service/loki patched
```


3. Test it
```bash
k get po,svc -n logging                                                
    NAME                                READY   STATUS    RESTARTS   AGE
    pod/alloy-5b4c6bd48c-8psxh          2/2     Running   0          19m
    pod/loki-0                          2/2     Running   0          4m35s
    pod/loki-canary-mndhs               1/1     Running   0          4m35s
    pod/loki-canary-qnflb               1/1     Running   0          4m35s
    pod/loki-canary-tgcsq               1/1     Running   0          4m35s
    pod/loki-gateway-77d8b6f5c4-2jm89   1/1     Running   0          4m35s

    NAME                      TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                                         AGE
    service/alloy             LoadBalancer   240.1.12.57    10.47.7.49    12345:30938/TCP,1514:32455/TCP,1515:32032/UDP   19m
    service/loki              LoadBalancer   240.1.14.211   10.47.7.48    3100:30562/TCP,9095:31642/TCP                   4m35s
    service/loki-canary       ClusterIP      240.1.15.65    <none>        3500/TCP                                        4m35s
    service/loki-gateway      ClusterIP      240.1.7.113    <none>        80/TCP                                          4m35s
    service/loki-headless     ClusterIP      None           <none>        3100/TCP                                        4m35s
    service/loki-memberlist   ClusterIP      None           <none>        7946/TCP  
```


Test Alloy from any machine that can reach that Alloy IP:
```bash
# RFC 5424 test message over TCP (the leading "<14>1" is priority and version)
echo '<14>1 2026-11-19T10:00:00Z testhost myapp 1234 ID47 - hello from rfc5424' | nc <EXTERNAL-IP> 1514
```

Test Loki from any machine that can reach that Loki IP:
```bash
curl http://<EXTERNAL-IP>:3100/ready
curl http://10.47.7.48:3100/ready                                              
    ready
```

In Grafana, add new Data Source --> Type=Loki --> URL=http://<LOKI-EXTERNAL-Service-IP>:3100

Then you can open Explore → Loki and run {job="syslog"}. You can filter with {app="myapp"} or {severity="info"}.

Notes

