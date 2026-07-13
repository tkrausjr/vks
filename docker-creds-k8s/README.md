kubectl create secret docker-registry regcred \
  --docker-server=https://index.docker.io/v1/ \
    --docker-username=<your-username> \
      --docker-password=<your-password-or-token> \
        --docker-email=<your-email>


-----

# Then reference the secret in the Deployment Manifest in the POD spec.
