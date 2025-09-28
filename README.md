# Kubernetes Deployment Project

This project contains Kubernetes deployment manifests for running data storage and processing services with persistent storage and internet accessibility.

## Project Overview

This repository provides production-ready Kubernetes deployments for two main services:

- **PostGIS** - A PostgreSQL database with PostGIS extensions for geospatial data
- **MinIO** - An S3-compatible object storage service

Both services are configured with:
- Persistent storage for data durability
- Internet accessibility through ingress controllers
- Security best practices
- Comprehensive documentation
- Easy deployment procedures

## Services

### 🗄️ PostGIS Database
**Directory**: `postgis/`

A complete PostgreSQL deployment with PostGIS extensions for geospatial capabilities.

**Features**:
- PostgreSQL 18 with PostGIS 3.6
- Persistent volume storage (20Gi default)
- Internet accessible via configured domain
- Health checks and monitoring
- Production-ready configuration

**Access**: Database accessible via configured domain on port 5432

### 📦 MinIO Object Storage
**Directory**: `minio/`

An S3-compatible object storage solution for file and data storage.

**Features**:
- S3-compatible API
- Web-based console interface
- Persistent volume storage (50Gi default)
- Internet accessible via configured domain
- Both API and console endpoints
- Production-ready configuration

**Access**:
- API: `http://your-domain/api`
- Console: `http://your-domain/console`

## Project Structure

```
/
├── README.md              # This file - project overview
├── postgis/               # PostGIS deployment manifests
│   ├── namespace.yml      # Namespace definition
│   ├── secret.yml         # Database credentials
│   ├── pvc.yml           # Persistent volume claim
│   ├── deployment.yml     # PostGIS deployment
│   ├── service.yml       # Internal service (ClusterIP)
│   ├── service-nodeport.yml  # External access service (NodePort)
│   ├── ingress.yml       # Internet access configuration
│   └── README.md         # Detailed documentation
└── minio/                # MinIO deployment manifests
    ├── namespace.yml     # Namespace definition
    ├── secret.yml        # MinIO credentials
    ├── pvc.yml          # Persistent volume claim
    ├── deployment.yml    # MinIO deployment
    ├── service.yml      # Internal service (ClusterIP)
    ├── service-nodeport.yml  # External access service (NodePort)
    ├── ingress.yml      # Internet access configuration
    └── README.md        # Detailed documentation
```

## Quick Start

### Prerequisites
- Kubernetes cluster
- kubectl configured to access your cluster
- Ingress controller installed (nginx recommended)
- Storage class configured for persistent volumes
- Domain name pointing to your cluster

### Deploy PostGIS

```bash
cd postgis
# Edit secret.yml with your database credentials
kubectl apply -f namespace.yml
kubectl apply -f secret.yml
kubectl apply -f pvc.yml
kubectl apply -f deployment.yml
kubectl apply -f service.yml
kubectl apply -f service-nodeport.yml  # Optional: For NodePort access
kubectl apply -f ingress.yml
```

### Deploy MinIO

```bash
cd minio
# Edit secret.yml with your MinIO credentials
kubectl apply -f namespace.yml
kubectl apply -f secret.yml
kubectl apply -f pvc.yml
kubectl apply -f deployment.yml
kubectl apply -f service.yml
kubectl apply -f service-nodeport.yml  # Optional: For NodePort access
kubectl apply -f ingress.yml
```

## Configuration

### Domain Configuration
Update the domain names in the respective `ingress.yml` files:
- PostGIS: `postgis.your-domain.com`
- MinIO: `minio.your-domain.com`

### Storage Configuration
Update the `storageClassName` in the `pvc.yml` files to match your cluster's storage class.

### Security
- Change default passwords in secret files
- Configure TLS/SSL certificates for production use
- Consider network policies for additional security

## NodePort Access

Both services include NodePort configurations for direct access to cluster nodes:

### PostGIS NodePort
- **Service**: `postgis-service-nodeport`
- **Node Port**: 30432
- **Access**: `http://any-node-ip:30432`

### MinIO NodePort
- **Service**: `minio-service-nodeport`
- **API Port**: 30090
- **Console Port**: 30091
- **API Access**: `http://any-node-ip:30090`
- **Console Access**: `http://any-node-ip:30091`

**Note**: NodePort services are optional and provide direct access to services without requiring ingress configuration.

## Monitoring and Troubleshooting

### Check Status
```bash
# All namespaces
kubectl get all -n postgis
kubectl get all -n minio

# Persistent volumes
kubectl get pvc -n postgis
kubectl get pvc -n minio

# Logs
kubectl logs -l app=postgis -n postgis
kubectl logs -l app=minio -n minio
```

### Common Issues
- Ensure ingress controller is properly configured
- Verify storage class exists and is accessible
- Check that domains point to correct IP addresses
- Validate secret base64 encoding

## Production Considerations

### Security
- Enable TLS/SSL encryption
- Configure proper network policies
- Use strong, unique credentials
- Regular security updates

### Performance
- Monitor resource usage
- Scale deployments based on load
- Configure appropriate resource limits
- Use high-performance storage classes

### Backup and Recovery
- Set up regular backups for PostGIS data
- Configure MinIO bucket policies
- Test disaster recovery procedures

### High Availability
- Consider using multiple replicas
- Configure load balancers
- Set up monitoring and alerting
- Plan for failover scenarios

## Documentation

Each service has its own detailed documentation:
- [PostGIS Documentation](postgis/README.md)
- [MinIO Documentation](minio/README.md)

## Support

For issues and questions:
1. Check the detailed documentation in each service directory
2. Review Kubernetes logs and events
3. Verify configuration against the provided examples
4. Ensure all prerequisites are met

## License

This project is provided as-is for educational and development purposes.