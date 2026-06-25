
# List the available addons and addonreleases you can install
https://techdocs.broadcom.com/us/en/vmware-cis/vcf/vcf-service-administration-and-development/9-0/managing-vsphere-kuberenetes-service-clusters-and-workloads/managing-add-ons-in-vks-clusters/view-available-addons.html

# Label the cluster according to the addons it needs
k label cluster/demo-cl01 -n shared-svcs-7w8d9 addons-install=headlamp    
    cluster.cluster.x-k8s.io/demo-cl01 labeled
# Modify the headlamp-labels.yaml with the label in the selector field

# Install

# To troubleshoot,

vcf context use supervisor-ctx
k get addoninstalls -n shared-svcs-7w8d9
    NAME       ADDON      PAUSED   AGE
    headlamp   headlamp            52m
 get addoninstalls -n shared-svcs-7w8d9 -oyaml | yq .status
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

# To test Navneets full nav-full-headlamp-exmpample.yaml
1 - Create a demo1 vSphere Namespace
cd ~/Downloads/vks/
k apply -f add-ons/nav-full-headlamp-example.yaml                 
    addoninstall.addons.kubernetes.vmware.com/cluster-headlamp created
    addonconfig.addons.kubernetes.vmware.com/workload-vsphere-vks1-headlamp created
    addoninstall.addons.kubernetes.vmware.com/cluster-cert-manager created
    addoninstall.addons.kubernetes.vmware.com/cluster-prom created
    addonconfig.addons.kubernetes.vmware.com/workload-vsphere-vks1-prometheus created
    addoninstall.addons.kubernetes.vmware.com/cluster-istio created
    addonconfig.addons.kubernetes.vmware.com/workload-vsphere-vks1-istio created
k get addoninstalls -A -n demo1                       
    NAMESPACE                  NAME                                            ADDON                  PAUSED   AGE
    demo1                      cluster-cert-manager                            cert-manager                    29s
    demo1                      cluster-headlamp                                headlamp                        29s
    demo1                      cluster-istio                                   istio                           29s
    demo1                      cluster-prom                                    prometheus                      29s

## Create the workload cluster
k apply -f add-ons/workload-vsphere-vks1-cluster.yaml
    cluster.cluster.x-k8s.io/workload-vsphere-vks1 created

## Validate that the add-ons were installed to the cluster.
