# Kubernetes Configuration Files for CtrlB Logging and Tracing

This guide provides separate YAML files for deploying logging and tracing to CtrlB on Kubernetes.

## File Structure

```
ctrlb-k8s-configs/
├── fluent-bit/
│   ├── 01-namespace.yaml
│   ├── 02-configmap.yaml
│   ├── 03-serviceaccount.yaml
│   ├── 04-clusterrole.yaml
│   ├── 05-clusterrolebinding.yaml
│   └── 06-daemonset.yaml
├── otel/
│   ├── 01-namespace.yaml
│   ├── 02-configmap.yaml
│   ├── 03-serviceaccount.yaml
│   ├── 04-clusterrole.yaml
│   ├── 05-clusterrolebinding.yaml
│   ├── 06-daemonset.yaml
│   └── 07-service.yaml
└── README.md
```

## Deployment Instructions

### Option 1: Deploy FluentBit Only

```bash
# Create directory and download files
mkdir -p ctrlb-k8s-configs/fluent-bit
cd ctrlb-k8s-configs/fluent-bit

# Edit the ConfigMap to add your credentials
# Replace <INSTANCE_HOST>, <STREAM_NAME>, and <API_TOKEN>
sed -i 's/<INSTANCE_HOST>/your-instance-host/g' 02-configmap.yaml
sed -i 's/<STREAM_NAME>/your-stream-name/g' 02-configmap.yaml
sed -i 's/<API_TOKEN>/your-api-token/g' 02-configmap.yaml

# Apply all configs
kubectl apply -f 01-namespace.yaml
kubectl apply -f 02-configmap.yaml
kubectl apply -f 03-serviceaccount.yaml
kubectl apply -f 04-clusterrole.yaml
kubectl apply -f 05-clusterrolebinding.yaml
kubectl apply -f 06-daemonset.yaml

# Verify deployment
kubectl get pods -n ctrlb-logging -l app=fluent-bit
kubectl logs -n ctrlb-logging -l app=fluent-bit --tail=50
```

### Option 2: Deploy OpenTelemetry Collector Only

```bash
# Create directory and download files
mkdir -p ctrlb-k8s-configs/otel
cd ctrlb-k8s-configs/otel

# Edit the ConfigMap to add your credentials
# Replace <INSTANCE_HOST>, <STREAM_NAME>, and <API_TOKEN>
sed -i 's/<INSTANCE_HOST>/your-instance-host/g' 02-configmap.yaml
sed -i 's/<STREAM_NAME>/your-stream-name/g' 02-configmap.yaml
sed -i 's/<API_TOKEN>/your-api-token/g' 02-configmap.yaml

# Apply all configs
kubectl apply -f 01-namespace.yaml
kubectl apply -f 02-configmap.yaml
kubectl apply -f 03-serviceaccount.yaml
kubectl apply -f 04-clusterrole.yaml
kubectl apply -f 05-clusterrolebinding.yaml
kubectl apply -f 06-daemonset.yaml
kubectl apply -f 07-service.yaml

# Verify deployment
kubectl get pods -n otel -l app=otel-collector
kubectl logs -n otel -l app=otel-collector --tail=50
```

### Option 3: Deploy All at Once

```bash
# Apply FluentBit
kubectl apply -f ctrlb-k8s-configs/fluent-bit/

# Apply OpenTelemetry
kubectl apply -f ctrlb-k8s-configs/otel/
```

---

## Quick Troubleshooting

```bash
# Check FluentBit logs
kubectl logs -n ctrlb-logging -l app=fluent-bit

# Check OTEL logs
kubectl logs -n otel -l app=otel-collector

# Check resource usage
kubectl top pods -n ctrlb-logging
kubectl top pods -n otel

# Test connectivity
kubectl run debug --image=curlimages/curl -it --restart=Never -- sh
curl -H "Authorization: Basic <API_TOKEN>" https://<INSTANCE_HOST>/api/default/<STREAM_NAME>/_json
```