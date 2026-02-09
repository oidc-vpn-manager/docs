# OIDC VPN Manager Deployment Guide

This guide covers production deployment of OIDC VPN Manager using both Docker Compose and Kubernetes/Helm options.

## 🏗️ Deployment Architecture

OIDC VPN Manager is designed to run in containerized environments with the following components:

- **Frontend Service**: Web UI and API gateway
- **Signing Service**: Certificate signing and cryptographic operations  
- **Certificate Transparency Service**: Audit logging of all issued certificates
- **PostgreSQL Databases**: Persistent storage for each service (can be external/managed)
- **Load Balancer/Ingress**: SSL termination and traffic routing

## 📋 Prerequisites

### Common Requirements
- **Container Runtime**: Docker 20.10+ or compatible OCI runtime
- **Database**: PostgreSQL 13+ (managed service, shared instance, or self-hosted)
- **OIDC Provider**: Enterprise identity provider (Azure AD, Okta, Keycloak, etc.)
- **PKI Materials**: Root and Intermediate CA certificates and keys
- **SSL Certificates**: Valid TLS certificates for your domain

### System Requirements
- **CPU**: Minimum 2 cores, recommended 4+ cores
- **Memory**: Minimum 4GB RAM, recommended 8GB+ RAM  
- **Storage**: Minimum 20GB, recommended 100GB+ for logs and certificates
- **Network**: HTTPS connectivity to OIDC provider and client access

## 🐳 Docker Compose Deployment

### Quick Start

1. **Prepare PKI Materials**
   ```bash
   # Generate PKI using the included tool
   cd tools/pki_tool
   python generate_pki.py --output-dir ../../deploy/docker/pki
   ```

2. **Configure Environment**
   ```bash
   cd deploy/docker
   
   # Generate production secrets
   ./generate-secrets.sh
   
   # Edit environment files with your settings
   nano .env.frontend
   nano .env.signing  
   nano .env.certtransparency
   ```

3. **Update Configuration**
   ```bash
   # Update OIDC settings in .env.frontend
   OIDC_DISCOVERY_URL=https://your-oidc-provider.com/.well-known/openid-configuration
   OIDC_CLIENT_ID=your-production-client-id

   # Update domain in docker-compose.yml and nginx.conf
   # Replace vpn.yourdomain.com with your actual domain
   ```

4. **Add SSL Certificates**
   ```bash
   mkdir ssl
   # Copy your SSL certificates
   cp /path/to/your/server.crt ssl/
   cp /path/to/your/server.key ssl/
   ```

5. **Deploy Services**
   ```bash
   docker-compose up -d
   ```

### Database Configuration Options

#### Option 1: Included PostgreSQL (Default)
The default `docker-compose.yml` includes PostgreSQL containers for each service. This is suitable for smaller deployments or testing.

#### Option 2: External PostgreSQL Database
For production environments, you may want to use a managed PostgreSQL service:

1. **Update Environment Files**:
   ```bash
   # .env.frontend
   DATABASE_URI=postgresql://frontend_user:password@your-db-host:5432/frontend_db
   
   # .env.certtransparency  
   DATABASE_URI=postgresql://ct_user:password@your-db-host:5432/ct_db
   ```

2. **Remove PostgreSQL Services** from `docker-compose.yml`:
   ```bash
   # Comment out or remove postgres-* services and their dependencies
   ```

3. **Create Databases** on your PostgreSQL instance:
   ```sql
   CREATE DATABASE frontend_db;
   CREATE DATABASE ct_db;
   CREATE USER frontend_user WITH PASSWORD 'secure_password';
   CREATE USER ct_user WITH PASSWORD 'secure_password';
   GRANT ALL PRIVILEGES ON DATABASE frontend_db TO frontend_user;
   GRANT ALL PRIVILEGES ON DATABASE ct_db TO ct_user;
   ```

### Production Configuration

#### Environment Variables

**Frontend Service (`.env.frontend`)**:
```bash
ENVIRONMENT=production
# For managed PostgreSQL:
DATABASE_URI=postgresql://frontend_user:password@managed-db.amazonaws.com:5432/frontend_db
# For included PostgreSQL:
# DATABASE_URI=postgresql://frontend_user:$(cat /run/secrets/postgres_frontend_password)@postgres-frontend:5432/frontend_db

LOG_LEVEL=INFO
SIGNING_SERVICE_URL=http://signing:8500
SIGNING_SERVICE_API_SECRET_FILE=/run/secrets/signing_api_secret
CERTTRANSPARENCY_SERVICE_URL=http://certtransparency:8800/api/v1
OIDC_DISCOVERY_URL=https://your-oidc-provider.com/.well-known/openid-configuration
OIDC_CLIENT_ID=your-production-client-id
OIDC_CLIENT_SECRET_FILE=/run/secrets/oidc_client_secret
OIDC_ADMIN_GROUP=vpn-admins
```

**Signing Service (`.env.signing`)**:
```bash
ENVIRONMENT=production
LOG_LEVEL=INFO
INTERMEDIATE_CA_KEY_PASSPHRASE_FILE=/run/secrets/ca_key_passphrase
SIGNING_SERVICE_API_SECRET_FILE=/run/secrets/signing_api_secret
CERTTRANSPARENCY_SERVICE_URL=http://certtransparency:8800/api/v1
CT_SERVICE_API_SECRET_FILE=/run/secrets/ct_api_secret
```

**Certificate Transparency Service (`.env.certtransparency`)**:
```bash
ENVIRONMENT=production
# For managed PostgreSQL:
DATABASE_URI=postgresql://ct_user:password@managed-db.amazonaws.com:5432/ct_db
# For included PostgreSQL:
# DATABASE_URI=postgresql://ct_user:$(cat /run/secrets/postgres_ct_password)@postgres-certtransparency:5432/ct_db

LOG_LEVEL=INFO
CT_SERVICE_API_SECRET_FILE=/run/secrets/ct_api_secret
```

#### Monitoring and Maintenance

```bash
# Check service status
docker-compose ps

# View service logs  
docker-compose logs -f frontend
docker-compose logs -f signing
docker-compose logs -f certtransparency

# Update services
docker-compose pull
docker-compose up -d

# Backup databases (if using included PostgreSQL)
docker-compose exec postgres-frontend pg_dump -U frontend_user frontend_db > frontend_backup.sql
docker-compose exec postgres-certtransparency pg_dump -U ct_user ct_db > ct_backup.sql
```

## ☸️ Kubernetes Deployment with Helm

### Prerequisites

- **Kubernetes**: Version 1.19+ cluster
- **Helm**: Version 3.2.0+  
- **Storage**: StorageClass for persistent volumes (if using included PostgreSQL)
- **Ingress**: Ingress controller (nginx recommended)
- **Cert-Manager**: For automated SSL certificate management (optional)

### Installation Steps

1. **Add Helm Dependencies (Optional)**
   ```bash
   # Only if using included PostgreSQL
   helm repo add bitnami https://charts.bitnami.com/bitnami
   helm repo update
   ```

2. **Create Namespace**
   ```bash
   kubectl create namespace oidc-vpn-manager
   ```

3. **Prepare Configuration**
   ```bash
   cd deploy/helm/oidc-vpn-manager
   cp values.yaml values-production.yaml
   # Edit values-production.yaml with your settings
   ```

### Database Configuration Options

#### Option 1: Included PostgreSQL (Bitnami Chart)
Default configuration includes PostgreSQL via the Bitnami Helm chart:

```yaml
# values-production.yaml
postgresql:
  enabled: true
  auth:
    postgresPassword: "changeme"
  primary:
    persistence:
      size: 20Gi
```

#### Option 2: External/Managed PostgreSQL  
For production environments using managed databases:

```yaml
# values-production.yaml
postgresql:
  enabled: false  # Disable included PostgreSQL

# Configure external database connections
frontend:
  config:
    database:
      host: "managed-db.amazonaws.com"
      port: 5432
      name: "frontend_db"
      user: "frontend_user"

certtransparency:
  config:
    database:
      host: "managed-db.amazonaws.com" 
      port: 5432
      name: "ct_db"
      user: "ct_user"
```

Create database credentials secret:
```bash
kubectl create secret generic external-db-credentials \
  --from-literal=frontend-password='frontend_password' \
  --from-literal=ct-password='ct_password' \
  -n oidc-vpn-manager
```

4. **Create Application Secrets**
   ```bash
   # OIDC client secret
   kubectl create secret generic oidc-vpn-manager-oidc-client-secret \
     --from-literal=client-secret='your-oidc-client-secret' \
     -n oidc-vpn-manager
   
   # CA key passphrase
   kubectl create secret generic oidc-vpn-manager-ca-key-passphrase \
     --from-literal=passphrase='your-ca-key-passphrase' \
     -n oidc-vpn-manager
   
   # PKI materials
   kubectl create secret generic oidc-vpn-manager-pki \
     --from-file=root-ca.crt=path/to/root-ca.crt \
     --from-file=intermediate-ca.crt=path/to/intermediate-ca.crt \
     --from-file=intermediate-ca.key=path/to/intermediate-ca.key \
     -n oidc-vpn-manager
   ```

5. **Deploy Application**
   ```bash
   helm install oidc-vpn-manager ./oidc-vpn-manager \
     --namespace oidc-vpn-manager \
     --values values-production.yaml
   ```

### Production Values Configuration

Create a `values-production.yaml` file:

```yaml
# Production configuration
global:
  storageClass: "gp3"

frontend:
  replicaCount: 3
  config:
    oidc:
      discoveryUrl: "https://auth.yourcompany.com/.well-known/openid-configuration"
      clientId: "oidc-vpn-manager-prod"
      adminGroup: "vpn-administrators"

ingress:
  hosts:
    - host: vpn.yourcompany.com
      paths:
        - path: /
          pathType: Prefix
          service: frontend
  tls:
    - secretName: vpn-yourcompany-com-tls
      hosts:
        - vpn.yourcompany.com

# Option 1: Included PostgreSQL
postgresql:
  enabled: true
  primary:
    persistence:
      size: 20Gi
    resources:
      limits:
        cpu: 1
        memory: 2Gi

# Option 2: External PostgreSQL (disable above and use this)
# postgresql:
#   enabled: false
# 
# externalDatabase:
#   host: managed-db.amazonaws.com
#   port: 5432
#   frontend:
#     database: frontend_db
#     username: frontend_user
#     existingSecret: external-db-credentials
#     existingSecretPasswordKey: frontend-password
#   certtransparency:
#     database: ct_db  
#     username: ct_user
#     existingSecret: external-db-credentials
#     existingSecretPasswordKey: ct-password

monitoring:
  serviceMonitor:
    enabled: true
    namespace: monitoring
```

### Operational Commands

```bash
# Check deployment status
kubectl get pods -n oidc-vpn-manager
helm status oidc-vpn-manager -n oidc-vpn-manager

# View application logs
kubectl logs -n oidc-vpn-manager deployment/oidc-vpn-manager-frontend
kubectl logs -n oidc-vpn-manager deployment/oidc-vpn-manager-signing

# Scale services
kubectl scale deployment oidc-vpn-manager-frontend --replicas=5 -n oidc-vpn-manager

# Update deployment
helm upgrade oidc-vpn-manager ./oidc-vpn-manager \
  --namespace oidc-vpn-manager \
  --values values-production.yaml

# Database access (if using included PostgreSQL)
kubectl exec -it -n oidc-vpn-manager deployment/oidc-vpn-manager-postgresql -- psql -U postgres
```

## 🔐 Security Considerations

### Network Security
- **TLS Everywhere**: All external communication over HTTPS/TLS
- **Network Policies**: Restrict pod-to-pod communication in Kubernetes
- **Firewall Rules**: Limit external access to necessary ports only

### Secret Management
- **External Secrets**: Integration with cloud secret managers (AWS Secrets Manager, Azure Key Vault, etc.)
- **Rotation**: Regular secret rotation procedures
- **Least Privilege**: Minimal permissions for service accounts
- **Audit**: All secret access logged and monitored

### Database Security
- **Encryption**: Database encryption at rest and in transit
- **Access Control**: Database-level user permissions and network restrictions
- **Backup Security**: Encrypted database backups with secure storage
- **Monitoring**: Database access logging and anomaly detection

### Certificate Security
- **Offline Root CA**: Root CA kept offline and secure
- **Certificate Transparency**: All certificates logged for audit
- **Monitoring**: Alert on suspicious certificate issuance patterns

## 🚀 Performance Tuning

### Database Optimization
```sql
-- PostgreSQL configuration for production
shared_buffers = 256MB
effective_cache_size = 1GB
maintenance_work_mem = 64MB
checkpoint_completion_target = 0.7
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 1.1
```

### Application Tuning
```bash
# Frontend service
FLASK_ENV=production
GUNICORN_WORKERS=4
GUNICORN_CONNECTIONS=2048
DATABASE_POOL_SIZE=20

# Signing service  
SIGNING_WORKERS=2
CRYPTO_CACHE_SIZE=1000
```

### Infrastructure Scaling
- **Load Balancing**: Multiple frontend instances
- **Database Read Replicas**: Scale read operations  
- **CDN**: Static asset delivery optimization
- **Caching**: Redis for session and certificate caching

## 🔍 Monitoring and Alerting

### Metrics Collection
- **Prometheus**: Application and infrastructure metrics
- **Grafana**: Visualization and dashboards
- **AlertManager**: Critical alert notifications

### Key Metrics to Monitor
- Certificate issuance rate and success/failure ratios
- Service response times and availability
- Database performance and connection pool usage
- Authentication success/failure rates
- Resource utilization (CPU, memory, storage)

### Log Aggregation
- **Centralized Logging**: ELK stack or cloud logging services
- **Structured Logs**: JSON format for parsing
- **Log Retention**: Compliance-appropriate retention policies
- **Security Monitoring**: Failed authentication attempts, suspicious patterns

---

This deployment guide provides flexible options for running OIDC VPN Manager in production environments with both self-hosted and managed database options.