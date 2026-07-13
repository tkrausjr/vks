```
kubectl create secret docker-registry regcred \
  --docker-server=https://index.docker.io/v1/ \
    --docker-username=<your-username> \
      --docker-password=<your-password-or-token> \
        --docker-email=<your-email>
```

# Then reference the secret in the Deployment Manifest in the POD spec.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: private-image-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: private-app
  template:
    metadata:
      labels:
        app: private-app
    spec:
      containers:
      - name: my-container
        image: <your-username>/private-repo:latest
      imagePullSecrets:
      - name: regcred # Must match the secret name created in Step 1
```
