This will show Supervisor fluentbit--> Alloy --> Loki --> Grafana. Alloy's loki.source.syslog component parses RFC 5424 natively, over TCP or UDP. 

1. Loki 

Use the loki-values.yaml and Grafana install from my previous answer, in namespace logging.

2. Alloy as the syslog receiver

cat > alloy-values.yaml <<'EOF'
alloy:
  configMap:
    content: |
      loki.source.syslog "rfc5424" {
        listener {
          address       = "0.0.0.0:1514"
          protocol      = "tcp"
          syslog_format = "rfc5424"
          use_incoming_timestamp = true
          labels        = { job = "syslog" }
        }
        listener {
          address       = "0.0.0.0:1515"
          protocol      = "udp"
          syslog_format = "rfc5424"
          labels        = { job = "syslog" }
        }
        relabel_rules = loki.relabel.syslog.rules
        forward_to    = [loki.write.local.receiver]
      }

      loki.relabel "syslog" {
        forward_to = []
        rule { source_labels = ["__syslog_message_hostname"] target_label = "host" }
        rule { source_labels = ["__syslog_message_app_name"] target_label = "app" }
        rule { source_labels = ["__syslog_message_severity"] target_label = "severity" }
        rule { source_labels = ["__syslog_connection_ip_address"] target_label = "source_ip" }
      }

      loki.write "local" {
        endpoint { url = "http://loki.logging.svc:3100/loki/api/v1/push" }
      }
  extraPorts:
    - { name: syslog-tcp, port: 1514, targetPort: 1514, protocol: TCP }
    - { name: syslog-udp, port: 1515, targetPort: 1515, protocol: UDP }
controller:
  type: deployment
service:
  type: LoadBalancer
EOF
helm install alloy grafana/alloy -n logging -f alloy-values.yaml
kubectl -n logging get svc alloy

The EXTERNAL-IP of the alloy Service is the address your devices send syslog to. In your lab, the load balancer provides it. Use NodePort instead if you don't have one.

3. Test it

From any machine that can reach that IP:

# RFC 5424 test message over TCP (the leading "<14>1" is priority and version)
echo '<14>1 2026-11-19T10:00:00Z testhost myapp 1234 ID47 - hello from rfc5424' | nc <EXTERNAL-IP> 1514

In Grafana, open Explore → Loki and run {job="syslog"}. You can filter with {app="myapp"} or {severity="info"}.

Notes

- Framing: TCP syslog usually uses octet-counting or newline framing. Alloy accepts both, so most senders work as is.
- TLS: the listener supports TLS through a tls_config block. Skip it for a first test and add it later.
- Labels: I mapped hostname, app name, severity and source IP to labels. Don't turn high-cardinality fields, such as message IDs, into labels, because that bloats Loki.
- Pointing something at it: to test the Supervisor or vCenter later, set the syslog target to <EXTERNAL-IP>:1514 over TCP. Check that the sender emits RFC 5424 and not RFC 3164. If it sends 3164, change syslog_format to rfc3164.
