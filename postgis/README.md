# PostGIS Kubernetes Deployment

This directory contains Kubernetes manifests for deploying PostGIS with persistent storage and internet accessibility.

## Files Overview

- `namespace.yml` - Creates the postgis namespace
- `secret.yml` - Contains environment variables (database credentials)
- `pvc.yml` - Persistent volume claim for data storage
- `deployment.yml` - PostGIS deployment configuration
- `service.yml` - Service to expose PostGIS within the cluster (ClusterIP)
- `service-nodeport.yml` - Service for external NodePort access
- `ingress.yml` - Ingress for external internet access

## Deployment Instructions

### 1. Update Secret Values
Before deploying, update the base64 encoded values in `secret.yml`:

```bash
# Generate new base64 values for your credentials
echo -n "your_database_name" | base64
echo -n "your_username" | base64
echo -n "your_secure_password" | base64
```

### 2. Configure Storage Class
Update the `storageClassName` in `pvc.yml` to match your cluster's storage class:

```yaml
storageClassName: your-storage-class-name  # e.g., standard, fast-ssd, etc.
```

### 3. Configure Ingress
Update the `ingress.yml` file:

1. Replace `postgis.your-domain.com` with your actual domain
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

# Check all resources in postgis namespace
kubectl get all -n postgis

# Check persistent volume claim
kubectl get pvc -n postgis

# Check logs if needed
kubectl logs -l app=postgis -n postgis
```

## Configuration Notes

### Environment Variables
The following environment variables are configured in the secret:
- `POSTGRES_DB`: Database name
- `POSTGRES_USER`: Database username
- `POSTGRES_PASSWORD`: Database password
- `POSTGRES_HOST_AUTH_METHOD`: Authentication method

### Storage
- Default storage request: 20Gi
- Access mode: ReadWriteOnce
- Mount path: `/var/lib/postgresql/data`

### Networking
- Service port: 5432
- Container port: 5432
- Service type: ClusterIP (internal cluster access)
- NodePort: 30432 (external access via NodePort service)

### NodePort Access

The NodePort service provides direct access to PostGIS without requiring ingress configuration:

- **Service**: `postgis-service-nodeport`
- **Node Port**: 30432
- **Access**: `http://any-node-ip:30432`
- **Use Case**: Development, testing, or when ingress is not available

**Note**: The NodePort service is optional and can be deployed alongside the ingress for additional access flexibility.

### Security Considerations

1. **Change default passwords**: Update the base64 encoded password in `secret.yml`
2. **Configure TLS**: Uncomment and configure TLS in `ingress.yml` for encrypted connections
3. **Network policies**: Consider adding Kubernetes NetworkPolicies for additional security
4. **RBAC**: Consider adding Role-Based Access Control for the PostGIS service account

### Troubleshooting

1. **Check pod status**: `kubectl get pods -n postgis`
2. **View logs**: `kubectl logs -l app=postgis -n postgis`
3. **Describe resources**: `kubectl describe deployment/postgis-deployment -n postgis`
4. **Check persistent volume**: `kubectl get pv -n postgis`

### Scaling and High Availability

For production deployments, consider:
- Using a StatefulSet instead of Deployment for ordered pod management
- Adding multiple replicas with proper anti-affinity rules
- Configuring PostgreSQL replication for high availability
- Using a managed PostgreSQL service for critical production workloads

## Cleanup

To remove the deployment:

```bash
kubectl delete -f ingress.yml
kubectl delete -f service.yml
kubectl delete -f deployment.yml
kubectl delete -f pvc.yml  # This will delete persistent data
kubectl delete -f secret.yml
kubectl delete -f namespace.yml