# WebMethods API Gateway with OpenSearch on AKS

This directory contains Kubernetes manifests for deploying WebMethods API Gateway, OpenSearch, and OpenSearch Dashboards on Azure Kubernetes Service (AKS).

## Directory Structure

```
k8s/
├── opensearch/
│   └── opensearch-statefulset.yaml      # OpenSearch cluster deployment
├── opensearch-dashboard/
│   └── opensearch-dashboard-deployment.yaml  # OpenSearch Dashboards UI
├── api-gateway/
│   └── api-gateway-deployment.yaml      # WebMethods API Gateway
├── kustomization.yaml                   # Kustomize configuration
└── README.md                            # This file
```

## Prerequisites

1. AKS cluster running and accessible
2. Azure Container Registry (ACR) with images pushed
3. kubectl configured with AKS context
4. Helm 3+ installed (for Helm deployments)

## Deployment Options

### Option 1: Using Kustomize

Deploy all components using Kustomize:

```bash
kubectl apply -k k8s/
```

### Option 2: Using kubectl

Deploy each component individually:

```bash
# Create namespace
kubectl create namespace webmethods-apigw

# Deploy OpenSearch
kubectl apply -f k8s/opensearch/opensearch-statefulset.yaml

# Deploy OpenSearch Dashboard
kubectl apply -f k8s/opensearch-dashboard/opensearch-dashboard-deployment.yaml

# Deploy API Gateway
kubectl apply -f k8s/api-gateway/api-gateway-deployment.yaml
```

### Option 3: Using Helm

Deploy using Helm (requires Helm chart):

```bash
helm install webmethods-stack helm/ \
  --namespace webmethods-apigw \
  --create-namespace \
  -f helm/values.yaml
```

## Accessing Services

### API Gateway
- **HTTP**: `http://<load-balancer-ip>:5555`
- **HTTPS**: `https://<load-balancer-ip>:5543`

Get the external IP:
```bash
kubectl get svc api-gateway -n webmethods-apigw
```

### OpenSearch API
- **Endpoint**: `http://opensearch:9200` (internal)
- **ClusterIP Service**: `opensearch-api` on port 9200

### OpenSearch Dashboard
- **Port**: `5601`
- **Access via Port Forward**:
```bash
kubectl port-forward svc/opensearch-dashboards 5601:5601 -n webmethods-apigw
```
- **URL**: `http://localhost:5601`
- **Default Credentials**: admin/admin

## Configuration

### API Gateway Configuration
Edit `k8s/api-gateway/api-gateway-deployment.yaml`:
- Adjust `JAVA_OPTS` for memory settings
- Modify `replicas` for scaling
- Configure resource requests/limits

### OpenSearch Configuration
Edit `k8s/opensearch/opensearch-statefulset.yaml`:
- Adjust storage size in `volumeClaimTemplates`
- Modify `JAVA_OPTS` for heap size
- Configure node roles and settings

### OpenSearch Dashboard Configuration
Edit `k8s/opensearch-dashboard/opensearch-dashboard-deployment.yaml`:
- Update OpenSearch connection details
- Modify resource limits

## Monitoring

### View Logs
```bash
# API Gateway logs
kubectl logs -f deployment/api-gateway -n webmethods-apigw

# OpenSearch logs
kubectl logs -f statefulset/opensearch -n webmethods-apigw

# OpenSearch Dashboard logs
kubectl logs -f deployment/opensearch-dashboards -n webmethods-apigw
```

### Check Pod Status
```bash
kubectl get pods -n webmethods-apigw
kubectl describe pod <pod-name> -n webmethods-apigw
```

## Persistence

OpenSearch uses StatefulSet with PersistentVolumes for data persistence:
- Storage Class: default (AKS managed disks)
- Size: 10Gi (configurable)
- Access Mode: ReadWriteOnce

## Security Considerations

1. **ACR Access**: Ensure ACR credentials are configured (imagePullSecrets)
2. **RBAC**: Configure appropriate RBAC policies
3. **Network Policies**: Implement network policies for inter-pod communication
4. **Secrets Management**: Store sensitive data in Kubernetes Secrets
5. **SSL/TLS**: Configure proper SSL certificates for production

## Cleanup

To remove all resources:

```bash
# Using Kustomize
kubectl delete -k k8s/

# Or using Helm
helm uninstall webmethods-stack -n webmethods-apigw

# Delete namespace
kubectl delete namespace webmethods-apigw
```

## Troubleshooting

### Pods not starting
```bash
kubectl describe pod <pod-name> -n webmethods-apigw
kubectl logs <pod-name> -n webmethods-apigw
```

### Image pull errors
```bash
# Check image pull secrets
kubectl get secrets -n webmethods-apigw

# Verify ACR access
az acr login --name wmaksdevsVXS
```

### PVC issues
```bash
kubectl get pvc -n webmethods-apigw
kubectl describe pvc <pvc-name> -n webmethods-apigw
```

## Support

For WebMethods API Gateway support: https://knowledge.softwareag.com
For OpenSearch support: https://opensearch.org/docs/latest/

## License

WebMethods components are licensed under Software AG licensing terms.
OpenSearch is licensed under Elastic License and SSPL.
