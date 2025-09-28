# MinIO Kubernetes Deployment

This directory contains Kubernetes manifests for deploying MinIO with persistent storage and internet accessibility.

## Files Overview

- `namespace.yml` - Creates the minio namespace
- `secret.yml` - Contains MinIO credentials (root user and password)
- `pvc.yml` - Persistent volume claim for data storage
- `deployment.yml` - MinIO deployment configuration
- `service.yml` - Service to expose MinIO within the cluster (ClusterIP)
- `service-nodeport.yml` - Service for external NodePort access
- `ingress.yml` - Ingress for external internet access

## Deployment Instructions

### 1. Update Secret Values
Before deploying, update the base64 encoded values in `secret.yml`:

```bash
# Generate new base64 values for your credentials
echo -n "your_access_key" | base64
echo -n "your_secret_key" | base64
echo -n "your-domain.com" | base64
```

### 2. Configure Storage Class
Update the `storageClassName` in `pvc.yml` to match your cluster's storage class:

```yaml
storageClassName: your-storage-class-name  # e.g., standard, fast-ssd, etc.
```

### 3. Configure Ingress
Update the `ingress.yml` file:

1. Replace `minio.your-domain.com` with your actual domain
2. Uncomment and configure TLS if you have certificates
3. Add appropriate annotations for your ingress controller (nginx, traefik, etc.)

### 4. Deploy to Kubernetes

Deploy in order (namespace first, then dependencies):

```bash
kubectl apply -f namespace.yml
kubectl apply -f secret.yml
kubectl apply -f pvc.yml
kubectl apply -f deployment.yml
kubectl apply -f service.yml
kubectl apply -f service-nodeport.yml  # Optional: For NodePort access
kubectl apply -f ingress.yml
```

### 5. Verify Deployment

```bash
# Check namespace
kubectl get namespaces

# Check all resources in minio namespace
kubectl get all -n minio

# Check persistent volume claim
kubectl get pvc -n minio

# Check logs if needed
kubectl logs -l app=minio -n minio
```

## Access Information

After successful deployment:

### Ingress Access (Recommended for Production)
- **MinIO Console**: http://minio.your-domain.com/console (or https if TLS is configured)
- **MinIO API**: http://minio.your-domain.com/api (or https if TLS is configured)

### NodePort Access (Development/Testing)
- **MinIO Console**: http://any-node-ip:30091
- **MinIO API**: http://any-node-ip:30090

- **Default Credentials**: Use the values you set in `secret.yml`

## Configuration Notes

### Environment Variables
The following environment variables are configured in the secret:
- `MINIO_ROOT_USER`: Root access key for MinIO
- `MINIO_ROOT_PASSWORD`: Root secret key for MinIO
- `MINIO_DOMAIN`: Domain name for MinIO (optional)

### Storage
- Default storage request: 50Gi
- Access mode: ReadWriteOnce
- Mount path: `/data`

### Networking
- API port: 9000
- Console port: 9001
- Service type: ClusterIP (internal cluster access)
- NodePort API: 30090 (external access via NodePort service)
- NodePort Console: 30091 (external access via NodePort service)

### NodePort Access

The NodePort service provides direct access to MinIO without requiring ingress configuration:

- **Service**: `minio-service-nodeport`
- **API Port**: 30090
- **Console Port**: 30091
- **API Access**: `http://any-node-ip:30090`
- **Console Access**: `http://any-node-ip:30091`
- **Use Case**: Development, testing, or when ingress is not available

**Note**: The NodePort service is optional and can be deployed alongside the ingress for additional access flexibility.

### Security Considerations

1. **Change default credentials**: Update the base64 encoded credentials in `secret.yml`
2. **Configure TLS**: Uncomment and configure TLS in `ingress.yml` for encrypted connections
3. **Network policies**: Consider adding Kubernetes NetworkPolicies for additional security
4. **RBAC**: Consider adding Role-Based Access Control for the MinIO service account
5. **Firewall**: Configure your ingress controller firewall rules appropriately

### Troubleshooting

1. **Check pod status**: `kubectl get pods -n minio`
2. **View logs**: `kubectl logs -l app=minio -n minio`
3. **Describe resources**: `kubectl describe deployment/minio-deployment -n minio`
4. **Check persistent volume**: `kubectl get pv -n minio`
5. **Port forward for debugging**: `kubectl port-forward -n minio svc/minio-service 9000:9000`

### MinIO Console Access

The MinIO Console provides a web-based interface for:
- Bucket management
- User management
- Policy configuration
- Monitoring and metrics

Access it at: `http://your-domain.com/console`

### API Access

MinIO provides S3-compatible API access:
- Endpoint: `http://your-domain.com`
- Access Key: Your configured root user
- Secret Key: Your configured root password

### Scaling and High Availability

For production deployments, consider:
- Using multiple replicas with proper anti-affinity rules
- Configuring MinIO distributed mode for high availability
- Using external storage backends
- Setting up monitoring and alerting
- Regular backup strategies

## Cleanup

To remove the deployment:

```bash
kubectl delete -f ingress.yml
kubectl delete -f service-nodeport.yml  # If NodePort was deployed
kubectl delete -f service.yml
kubectl delete -f deployment.yml
kubectl delete -f pvc.yml  # This will delete persistent data
kubectl delete -f secret.yml
kubectl delete -f namespace.yml
```

## Additional Resources

- [MinIO Documentation](https://docs.min.io/)
- [MinIO Kubernetes Operator](https://docs.min.io/docs/minio-kubernetes-operator)
- [S3 API Compatibility](https://docs.min.io/docs/s3-api-compatibility)