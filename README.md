# MinIO Deployment Guide - RunAI Kubernetes

## Overview

This guide covers deploying MinIO object storage on a RunAI Kubernetes cluster using Helm charts.

## Prerequisites

- Access to a Kubernetes cluster with RunAI
- Helm 3.x installed
- `kubectl` configured to access your cluster
- Access to the `runai-test` namespace (or appropriate namespace)

## Deployment Steps

### 1. Prepare Your Environment

Ensure you're in the directory containing your Helm chart files:

```bash
cd helm-runai
```

Verify your chart structure:

```
helm-runai/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── deployment.yaml
    ├── pvc.yaml
    └── service.yaml
```

### 2. Deploy MinIO

Run the following Helm command to deploy MinIO:

```bash
helm install my-minio-app . -n runai-test --set inferenceServiceUrl=http://triton.runai-test.svc.cluster.local
```

**Expected output:**

```
NAME: my-minio-app
LAST DEPLOYED: Tue Jun 10 17:57:55 2025
NAMESPACE: runai-test
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

### 3. Verify Deployment

Check that the pods are running:

```bash
kubectl get pods -n runai-test
```

**Expected output:**

```
NAME                                      READY   STATUS    RESTARTS   AGE
minio-5f89c57d9b-r677m                    1/1     Running   0          80s
```

Check the services:

```bash
kubectl get svc -n runai-test
```

Check the persistent volume claims:

```bash
kubectl get pvc -n runai-test
```

## Accessing MinIO

### Web Console Access (Recommended for Users)

1. **Port Forward to Local Machine:**

   ```bash
   kubectl port-forward svc/minio 9090:9090 -n runai-test
   ```

2. **Open Browser:**
   
   Navigate to: `http://localhost:9090`

3. **Login Credentials:**
   
   - **Username:** `minioadmin`
   - **Password:** `minioadmin123`

### API Access

For programmatic access:

```bash
kubectl port-forward svc/minio 9000:9000 -n runai-test
```

**API Endpoint:** `http://localhost:9000`

### Internal Cluster Access

Other services within the cluster can access MinIO at:

- **API:** `http://minio.runai-test.svc.cluster.local:9000`
- **Console:** `http://minio.runai-test.svc.cluster.local:9090`

## Configuration Details

### Default Configuration

- **Storage:** 1Gi persistent volume
- **Replicas:** 1
- **Image:** `minio/minio:latest`
- **Storage Class:** `ebs-csi` (AWS EBS)

### Environment Variables

- `MINIO_ROOT_USER`: minioadmin
- `MINIO_ROOT_PASSWORD`: minioadmin123
- `MINIO_BROWSER`: on
- `MINIO_CONSOLE_ADDRESS`: :9090
- `INFERENCE_SERVICE_URL`: http://triton.runai-test.svc.cluster.local

## Common Operations

### Updating the Deployment

To update configuration:

```bash
helm upgrade my-minio-app . -n runai-test --set <parameter>=<new-value>
```

### Scaling (if needed)

To change the number of replicas:

```bash
helm upgrade my-minio-app . -n runai-test --set replicaCount=2
```

### Uninstalling

To remove the deployment:

```bash
helm uninstall my-minio-app -n runai-test
```

## Troubleshooting

### Pod Stuck in Pending State

**Common Issue:** PVC not found or volume binding issues

**Solution:**

1. Check pod status:
   
   ```bash
   kubectl describe pod -l app=minio -n runai-test
   ```

2. If you see PVC errors, ensure you're not referencing a non-existent PVC:
   
   ```bash
   kubectl get pvc -n runai-test
   ```

3. Let the chart create its own PVC by not specifying `existingClaim`:
   
   ```bash
   helm uninstall my-minio-app -n runai-test
   helm install my-minio-app . -n runai-test --set inferenceServiceUrl=http://triton.runai-test.svc.cluster.local
   ```

### Connection Issues

**Problem:** Cannot access MinIO console

**Solutions:**

1. Verify port-forward is active:
   
   ```bash
   kubectl port-forward svc/minio 9090:9090 -n runai-test
   ```

2. Check if the service is running:
   
   ```bash
   kubectl get svc minio -n runai-test
   ```

3. Verify pod is healthy:
   
   ```bash
   kubectl logs -l app=minio -n runai-test
   ```

### Storage Issues

**Problem:** Running out of storage space

**Solution:** Increase PVC size in `values.yaml`:

```yaml
storage:
  size: 10Gi  # Increase from 1Gi
```

Then upgrade:

```bash
helm upgrade my-minio-app . -n runai-test
```

## Security Considerations

### Production Recommendations

1. **Change Default Credentials:**
   
   ```bash
   helm upgrade my-minio-app . -n runai-test \
     --set minio.rootUser=your-admin-user \
     --set minio.rootPassword=your-secure-password
   ```

2. **Use Secrets for Credentials:**
   
   Consider storing credentials in Kubernetes secrets instead of values.yaml

3. **Network Policies:**
   
   Implement network policies to restrict access to MinIO

## Integration with Other Services

### Triton Inference Server

MinIO is configured to work with Triton at:
`http://triton.runai-test.svc.cluster.local`

### RunAI Jobs

MinIO can be used as storage backend for RunAI training jobs and inference workloads.

## Support

For issues related to:

- **Kubernetes/RunAI:** Contact your cluster administrator
- **MinIO Configuration:** Refer to [MinIO Documentation](https://min.io/docs/)
- **Helm Issues:** Check [Helm Documentation](https://helm.sh/docs/)

## Quick Reference

| Component | Access Method | URL/Command |
|-----------|--------------|-------------|
| Web Console | Port Forward | `kubectl port-forward svc/minio 9090:9090 -n runai-test` |
| API | Port Forward | `kubectl port-forward svc/minio 9000:9000 -n runai-test` |
| Logs | kubectl | `kubectl logs -l app=minio -n runai-test` |
| Status | kubectl | `kubectl get pods -l app=minio -n runai-test` |

---

**Document Version:** 1.0  
**Last Updated:** June 10, 2025  
**Namespace:** runai-test
