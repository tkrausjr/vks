# Tested on VKS 3.7 and VCF 9.1
https://blogs.vmware.com/cloud-foundation/2025/03/06/istio-on-vsphere-kubernetes-service-vks-a-walkthrough/


# Latest VKS Addon for Istio seems to use Istio 1.30.x so wel will install that version of Kiali
cd ~/Downloads
git clone --branch release-1.30 --depth 1 https://github.com/istio/istio.git


# Configure a default storage class for the cluster for Kiali and Loki deployments needing a PVC
k get sc
kubectl patch storageclass cluster-wld01-01a-optimal-datastore-default-policy-autoraid -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
k apply -f ./istio/samples/addons

# Change the kiali service to type LoadBalancer or add Http route and GW 
k edit svc kiali -n istio-system
Change ClusterIP to LoadBalancer and save.

# Get the LB IP
k get svc kiali -n istio-system | awk '{print $4}'
   EXTERNAL-IP
   10.1.8.140

# Test in a browser
http://10.1.8.140:20001

SUCCESS