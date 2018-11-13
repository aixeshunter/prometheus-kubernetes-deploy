Monitoring Kubernetes clusters on normal Linux server.
===

# Prometheus Deploy in k8s

## Prometheus Summary

### Architecture

![avatar](arch.png)

### Sequence Chart

![avatar](prometheus-sc.png)

## Deploy

    ./deploy

Now you can access the dashboards locally using `kubectl port-forward`command, or expose the services using a ingress or a LoadBalancer. Please check the `./tools` directory to quickly configure a ingress or proxy the services to localhost.

To remove everything, just execute the `./teardown` script.


## Deploy other addon exporter

If you want to monitor GPU device, GlusterFS Cluster or Ceph Cluster, the `yaml` files are in `manifests/ceph-exporter/`, `manifests/gluster-exporter/`, `manifests/gpu-exporter/`.

You should create them manually.

```sh
kubectl apply -f manifests/(ceph|gluster|gpu)-exporter/*.yaml
```

## Updating configurations

  * **update alert rules:** add or change the rules in `assets/prometheus/rules/` and execute `scripts/generate-rules-configmap.sh`. Then apply the changes using `kubectl apply -f manifests/prometheus/prometheus-k8s-rules.yaml -n monitoring`
  * **update grafana dashboards:** add or change the existing dashboards in `assets/grafana/` and execute `scripts/generate-dashboards-configmap.sh`. Then apply the changes using `kubectl apply -f manifests/grafana/grafana-dashboards.cm.yaml`.

**Note:** all the Grafana dashboards should have names ending in `-dashboard.json`.

## Custom settings

All the components versions can be configured using the interactive deployment script. Same for the SMTP account or the Slack token.

Some other settings that can be changed before deployment:
  * **Prometheus replicas:** default **2** ==> `manifests/prometheus/prometheus-k8s.yaml`
  * **persistent volume size:** default **40Gi** ==> `manifests/prometheus/prometheus-k8s.yaml`
  * **allocated memory for Prometheus pods:** default **2Gi** ==> `manifests/prometheus/prometheus-k8s.yaml`
  * **Alertmanager replicas:** default **3** ==> `manifests/alertmanager/alertmanager.yaml`
  * **Alertmanager configuration:** ==> `assets/alertmanager/alertmanager.yaml`
  * **custom Grafana dashboards:** add yours in `assets/grafana/` with names ending in `-dashboard.json`
  * **custom alert rules:**  ==> `assets/prometheus/rules/`

**Note:** please commit your changes before deployment if you wish to keep them. The `deploy` script will remove the changes on most of the files.

## Prometheus Rules

```yaml
  - name: node-memory-usage
    rules:
    - alert: NodeMemoryUsageOvercommit
      annotations:
        summary: 'Overcommited Memory Usage Percnet on Node {{ $labels.instance }} for Some Time (current value: {{ printf "%.2f" $value }}%)'
        description: 'High Node Memory Usage'
      expr: instance:node_memory_utilisation:usage * 100 > 75.00         #PromQL expression
      for: 5m
      labels:
        severity: critical                                               #alert level, can be user-defined
        value: '{{ printf "%.2f" $value }}'
```
