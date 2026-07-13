# NFS Server has to run in privileged mode
k label namespace default pod-security.kubernetes.io/enforce=privileged

# Update the storage class to match your Clusters storageclass
# Create the test server
k apply -f ./nfs-pvcs/nfs-server.yaml

# Create the Static PV and PVC to the NFS Server mount
# Change the SERVER to match the Service type LB External IP of the Load Balancer
# For the NFS Server
k apply -f ./nfs-pvcs/static-pv-to-nfs.yaml


# Create POD 1
k apply -f ./nfs-pvcs/test-nfs-client-pod-1.yaml
k exec -it nfs-client-pod-1 -- /bin/sh
# Put test data into a file in /usr/share/nginx/html
vi /usr/share/nginx/html/nfs-client-pod-1-test-file.txt

# Create POD 2 and test you can see the data.
k apply -f ./nfs-pvcs/test-nfs-client-pod-2.yaml
k exec -it nfs-client-pod-2 -- /bin/sh
cat /usr/share/nginx/html/nfs-client-pod-1-test-file.txt
