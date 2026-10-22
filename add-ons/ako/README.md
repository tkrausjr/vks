
# List the available addons and addonreleases you can install
https://techdocs.broadcom.com/us/en/vmware-cis/vcf/vcf-service-administration-and-development/9-0/managing-vsphere-kuberenetes-service-clusters-and-workloads/managing-add-ons-in-vks-clusters/view-available-addons.html

# Label the cluster according to the addons it needs
```
k label cluster/demo-cl01 -n shared-svcs-7w8d9 addons-install=headlamp    
    cluster.cluster.x-k8s.io/demo-cl01 labeled
```
# Modify the headlamp-labels.yaml with the label in the selector field

# Install

# To troubleshoot,
```
vcf context use supervisor-ctx
k get addoninstalls -n shared-svcs-7w8d9
    NAME       ADDON      PAUSED   AGE
    headlamp   headlamp            52m
k get addoninstalls -n shared-svcs-7w8d9 -oyaml | yq .status
# Note this shows a cluster was found and matched by the labels

# Switch to Cluster context
vcf context use demo-cl01
k get package -A
k get pkgi -A
k get po -A
k get app -A
    Shows headlamp in vmware-system-tkg Namespace with Reconcile failed.
k get app -n vmware-system-tkg demo-cl01-headlamp -oyaml 
k get pkgi demo-cl01-headlamp -n vmware-system-tkg -oyaml

# To test Navneets full nav-full-headlamp-exmpample-testing.yaml
# Note this -testing version looks for the addons-install=headlamp   LABEL and installs the addons on any cluster with this label.
```
1 - Create a addons-testing vSphere Namespace
```
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
```

k label cluster/goad-test-02a -n tk-test-2 environment=production ako-enabled=true --overwrite=true
    cluster.cluster.x-k8s.io/demo-cl01 labeled
```
## Validate that the add-ons were installed to the cluster.


vcf context use vcfa:addons-testing-vzd34j:default-project
vcf cluster list
   kubernetes-cluster-v32p
vcf cluster register-vcfa-jwt-authenticator kubernetes-cluster-v32p
vcf cluster kubeconfig get goad-test-02a --admin
    Credentials of cluster 'kubernetes-cluster-v32p' have been saved 
    You can now access the cluster by running 'kubectl config use-context goad-test-02a'


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




In Firefox access AVI Controller
https://10.47.0.180

Looks like it works
