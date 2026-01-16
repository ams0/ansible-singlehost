# Observability Stack Deployment

This directory contains the configuration for deploying a complete observability stack with:

- **Prometheus** with OTLP receiver for OpenTelemetry metrics
- **Grafana** for visualization and dashboards
- **Thanos** with local disk storage for long-term metrics retention
- **Alertmanager** for alert management
- **7-day retention** policy
- **HTTPRoute** for Grafana access via Istio Gateway

## Prerequisites

1. Kubernetes cluster with Gateway API support
2. Istio installed with a Gateway named `gateway` in the `istio-system` namespace
3. Helm 3.x installed
4. Storage class available for PVC provisioning

## Deployment Steps

### 1. Create the observability namespace

```bash
kubectl create namespace observability
```

### 2. Apply the Thanos object storage ConfigMap

This ConfigMap configures Thanos to use local filesystem storage:

```bash
kubectl apply -f thanos-objstore-config.yaml
```

### 3. Create the Thanos storage PVC

```bash
kubectl apply -f thanos-storage-pvc.yaml
```

### 4. Add the Prometheus Community Helm repository

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

### 5. Install the kube-prometheus-stack

```bash
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace observability \
  --values values.yaml
```

Or upgrade if already installed:

```bash
helm upgrade kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace observability \
  --values values.yaml
```

### 6. Deploy the Grafana HTTPRoute

```bash
kubectl apply -f grafana-httproute.yaml
```

**Note**: Update the `backendRefs.name` in `grafana-httproute.yaml` if you use a different Helm release name.

## Accessing Grafana

Once deployed, Grafana will be accessible at:

- **URL**: https://grafana.vps.kubespaces.cloud
- **Default Username**: admin
- **Default Password**: admin (configured in values.yaml - **change this in production!**)

## Configuration Details

### Prometheus

- **OTLP Receiver**: Enabled for OpenTelemetry metrics ingestion
- **Retention**: 7 days
- **Storage**: 50Gi PVC
- **Thanos Sidecar**: Enabled with local filesystem storage

### Thanos

- **Storage Type**: FILESYSTEM (local disk)
- **Storage Path**: `/thanos/store`
- **PVC Size**: 100Gi (adjust in `thanos-storage-pvc.yaml` as needed)

### Grafana

- **Default Dashboards**: Enabled
- **Persistence**: 10Gi PVC
- **Service Type**: ClusterIP
- **Access**: Via HTTPRoute through Istio Gateway

### Alertmanager

- **Retention**: 120h
- **Storage**: 10Gi PVC

## Customization

### Changing Storage Sizes

Edit the following files:

- `values.yaml` - Prometheus storage (line 49)
- `values.yaml` - Grafana storage (line 86)
- `values.yaml` - Alertmanager storage (line 101)
- `thanos-storage-pvc.yaml` - Thanos storage (line 10)

### Changing Retention Period

Edit `values.yaml` line 18:

```yaml
retention: 7d  # Change to desired retention (e.g., 14d, 30d)
```

### Changing Grafana Password

Edit `values.yaml` line 60 or use a Kubernetes Secret:

```yaml
adminPassword: "your-secure-password"
```

### Updating HTTPRoute Backend

If you use a different Helm release name, update `grafana-httproute.yaml` line 17:

```yaml
backendRefs:
  - name: <your-release-name>-grafana
```

## Verification

### Check all pods are running

```bash
kubectl get pods -n observability
```

### Check PVCs

```bash
kubectl get pvc -n observability
```

### Check HTTPRoute

```bash
kubectl get httproute -n observability
```

### Access Prometheus

Port-forward to access Prometheus UI:

```bash
kubectl port-forward -n observability svc/kube-prometheus-stack-prometheus 9090:9090
```

Then visit: http://localhost:9090

### Access Grafana

Visit: https://grafana.vps.kubespaces.cloud

## Troubleshooting

### Thanos sidecar not starting

Check the Thanos object storage ConfigMap:

```bash
kubectl get cm thanos-objstore-config -n observability -o yaml
```

### Grafana not accessible via HTTPRoute

1. Check the HTTPRoute status:
   ```bash
   kubectl describe httproute grafana -n observability
   ```

2. Verify the Gateway exists:
   ```bash
   kubectl get gateway gateway -n istio-system
   ```

3. Check Grafana service name:
   ```bash
   kubectl get svc -n observability | grep grafana
   ```

### OTLP receiver not working

Check Prometheus configuration:

```bash
kubectl get prometheus -n observability -o yaml | grep -A 10 otlp
```

## Uninstallation

```bash
# Delete HTTPRoute
kubectl delete -f grafana-httproute.yaml

# Uninstall Helm release
helm uninstall kube-prometheus-stack -n observability

# Delete PVC and ConfigMap
kubectl delete -f thanos-storage-pvc.yaml
kubectl delete -f thanos-objstore-config.yaml

# Delete namespace (optional)
kubectl delete namespace observability
```

## References

- [kube-prometheus-stack Helm Chart](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [Thanos Documentation](https://thanos.io/tip/thanos/getting-started.md/)
- [Gateway API Documentation](https://gateway-api.sigs.k8s.io/)
