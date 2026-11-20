helm install loki grafana/loki -n logging --create-namespace -f loki-values.yaml

helm install grafana grafana/grafana -n logging \
  --set persistence.enabled=false \
  --set 'datasources.datasources\.yaml.apiVersion=1' \
  --set 'datasources.datasources\.yaml.datasources[0].name=Loki' \
  --set 'datasources.datasources\.yaml.datasources[0].type=loki' \
  --set 'datasources.datasources\.yaml.datasources[0].url=http://loki.logging.svc:3100'

The Loki push endpoint is then http://loki.logging.svc:3100/loki/api/v1/push.

Send logs from the cluster

Install Fluent Bit (a VKS add-on, or Helm) with a Loki output:

[OUTPUT]
    Name   loki
    Match  kube.*
    Host   loki.logging.svc
    Port   3100
    Labels job=fluent-bit, namespace=$kubernetes['namespace_name'], pod=$kubernetes['pod_name']

Alternatively, install Grafana Alloy or Promtail with helm install alloy grafana/alloy, which already ships with Kubernetes log discovery.

To see logs, run kubectl -n logging port-forward svc/grafana 3000:80. Log in with the admin password from the grafana Secret, then go to Explore → Loki, and query {namespace="music-store"}.