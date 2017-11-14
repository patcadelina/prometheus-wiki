groups:
      - name: ./kube-apiserver.alerts
        rules:
        - alert: K8SApiServerLatency
          expr: histogram_quantile(0.99, sum(apiserver_request_latencies_bucket{subresource!="log",verb!~"^(?:CONNECT|WATCHLIST|WATCH|PROXY)$"})
            WITHOUT (instance, resource)) / 1e+06 > 1
          for: 10m
          labels:
            severity: warning
          annotations:
            description: 99th percentile Latency for {{ $labels.verb }} requests to the
              kube-apiserver is higher than 1s.
            summary: Kubernetes apiserver latency is high