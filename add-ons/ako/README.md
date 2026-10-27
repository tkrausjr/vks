
## List the available addons and addonreleases you can install
https://techdocs.broadcom.com/us/en/vmware-cis/vcf/vcf-service-administration-and-development/9-0/managing-vsphere-kuberenetes-service-clusters-and-workloads/managing-add-ons-in-vks-clusters/view-available-addons.html

## Install AKO Add-on

1 - Create a addons-testing vSphere Namespace
```bash
cd ~/vks/add-ons/ako
k apply -f ./avi-ctls-creds-secret.yaml
k apply -f ./ako-addonconfig.yaml
k apply -f ./ako-addoninstall.yaml

k get addoninstalls -A -n addons-testing                       
    NAMESPACE                  NAME                                            ADDON                  PAUSED   AGE
    addons-testing             cluster-cert-manager                            cert-manager                    29s
    addons-testing             cluster-headlamp                                headlamp                        29s
    addons-testing             cluster-istio                                   istio                           29s
    addons-testing             cluster-prom                                    prometheus                      29s

kctx sup01-setlab:tk-test-2 
k get clusteraddon
k get addoninstall
k get addonconfig
```
## Label the workload cluster
```bash
k label cluster/goad-test-02a -n tk-test-2 environment=production ako-enabled=true --overwrite=true
    cluster.cluster.x-k8s.io/demo-cl01 labeled
```
## Validate that the add-ons were installed to the cluster.
```bash
vcf context use vcfa:addons-testing-vzd34j:default-project
vcf cluster list
   kubernetes-cluster-v32p
vcf cluster register-vcfa-jwt-authenticator kubernetes-cluster-v32p
vcf cluster kubeconfig get goad-test-02a --admin
    Credentials of cluster 'kubernetes-cluster-v32p' have been saved 
    You can now access the cluster by running 'kubectl config use-context goad-test-02a'
```
## Switch to the target cluster now to see what happened.
```bash
k config use-context goad-test-02a-admin@goad-test-02a
k get all -n avi-system
    k get all -n avi-system
    NAME                                    READY   STATUS    RESTARTS   AGE
    pod/ako-0                               2/2     Running   0          5m27s
    pod/ako-crd-operator-8445bd89dd-jhg4x   1/1     Running   0          5m27s
    NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/ako-crd-operator   1/1     1            1           5m27s
    NAME                                          DESIRED   CURRENT   READY   AGE
    replicaset.apps/ako-crd-operator-8445bd89dd   1         1         1       5m27s
    NAME                   READY   AGE
    statefulset.apps/ako   1/1     5m27s

k get app -A
k get pkgi -A
```
# Create a Gateway object.
```bash
k apply -f ./vks/Network/avi-gateway-api/gateway-metal-store.yaml 
k get gateway -A
    NAMESPACE   NAME          CLASS    ADDRESS      PROGRAMMED   AGE
    default     metal-store   avi-lb   10.47.7.25   True         17m
# # NOTE Now you will see a new VS in Avi on port 80 (Why not 443)

k apply -f ./github/vks/Network/avi-gateway-api/httproute-metal-store.yaml 
    httproute.gateway.networking.k8s.io/metal-store-frontend created

curl -vvv  metal-store.acme.gpu.set.lab
    * Could not resolve host: metal-store.acme.gpu.set.lab
    * Closing connection 0
    curl: (6) Could not resolve host: metal-store.acme.gpu.set.lab
```











In Firefox access AVI Controller
https://10.47.0.180

Looks like it works
