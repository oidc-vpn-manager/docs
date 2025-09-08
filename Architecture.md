# OpenVPN Manager Architecture

This document provides a comprehensive overview of the OpenVPN Manager system architecture, component interactions, and technical design decisions.

## 🏗️ System Overview

OpenVPN Manager follows a microservice architecture pattern with clear separation of concerns between certificate management, signing operations, and audit logging. The system is designed for security, scalability, and operational simplicity.

```
┌─────────────┐    ┌──────────────┐    ┌─────────────────────┐
│   Frontend  │    │   Signing    │    │  Certificate        │
│   Service   │────│   Service    │────│  Transparency       │
│  (Web UI)   │    │ (Cert Gen)   │    │  Service (Audit)    │
└─────────────┘    └──────────────┘    └─────────────────────┘
       │                   │                       │
       │                   │                       │
┌─────────────┐    ┌──────────────┐    ┌─────────────────────┐
│ PostgreSQL  │    │ PKI Storage  │    │   PostgreSQL        │
│ (Frontend)  │    │ (Signing)    │    │ (Transparency)      │
└─────────────┘    └──────────────┘    └─────────────────────┘
```

## 🔧 Core Services

### Frontend Service

**Purpose**: User-facing web application and REST API gateway
**Repository**: `services/frontend/`
**Ports**: 
- 8600 (combined service)
- 8450 (user service) 
- 8540 (admin service)

#### Responsibilities
- **Web UI**: Responsive web interface for users and administrators
- **OIDC Authentication**: Integration with enterprise identity providers
- **User Management**: Profile generation, certificate requests, download tokens
- **Admin Functions**: PSK management, certificate transparency log viewing
- **API Gateway**: RESTful API for programmatic access
- **Request Orchestration**: Coordinates between signing and transparency services
- **Service Separation**: Configurable deployment patterns for user/admin service isolation

#### Key Components
- **Flask Web Framework**: Python-based web application
- **Jinja2 Templates**: Server-side rendered HTML with progressive enhancement
- **SQLAlchemy ORM**: Database abstraction and migration management
- **OIDC Client**: OpenID Connect integration for enterprise authentication
- **API Documentation**: Swagger/OpenAPI specification at `/swagger/`

#### Database Schema
- **Users**: OIDC identity mapping and session management
- **Pre-Shared Keys (PSKs)**: Server authentication tokens
- **Download Tokens**: Temporary tokens for secure profile downloads
- **Configuration**: Service settings and feature flags

#### API Endpoints
- `GET /` - Web UI interface and User profile generation interface  
- `POST /api/v1/profile` - Generate user certificate and OpenVPN profile
- `GET /api/v1/server-bundle/{psk}` - Server configuration bundle retrieval
- `GET /admin/certificates` - Certificate transparency log viewer (admin)
- `POST /admin/psk` - Pre-shared key generation (admin)

#### Service Separation Deployment Patterns

The frontend service supports three deployment configurations for security isolation and operational flexibility:

**Combined Service** (Default):
- **Configuration**: No separation environment variables configured
- **Behavior**: Serves both user and admin functionality in a single deployment
- **Use Case**: Small deployments, development, backward compatibility

**User Service**:
- **Configuration**: `ADMIN_URL_BASE` environment variable configured
- **Behavior**: Serves user routes, rejects admin routes (403), redirects admin users to admin service
- **Use Case**: User-facing deployment with high availability requirements

**Admin Service**:
- **Configuration**: `USER_URL_BASE` environment variable configured  
- **Behavior**: Serves admin routes, redirects user routes to user service (301)
- **Use Case**: Administrative deployment with enhanced security controls

Route access is controlled through Flask decorators that enforce service separation policies based on the configured environment variables.

### Signing Service

**Purpose**: Isolated certificate signing and cryptographic operations
**Repository**: `services/signing/`
**Port**: 8500

#### Responsibilities
- **Certificate Signing**: Generate certificates using intermediate CA
- **Cryptographic Operations**: Secure key management and signing
- **Certificate Transparency Logging**: Automatic logging of all issued certificates
- **API Security**: Token-based authentication for service-to-service communication

#### Key Components
- **Flask Microservice**: Lightweight API-only service
- **Cryptography Library**: Modern Python cryptography for certificate operations
- **PKI Management**: Intermediate CA key handling with passphrase protection
- **CT Client**: Automatic logging to Certificate Transparency service

#### Security Features
- **Isolated Key Storage**: CA private keys never leave the signing service
- **API Authentication**: Shared secret authentication between services
- **Automatic CT Logging**: All certificates logged for audit purposes
- **Secure Defaults**: Strong cryptographic parameters and key sizes

#### Certificate Types
- **User Certificates**: Individual client certificates for VPN access
- **Server Certificates**: OpenVPN server certificates with proper extensions
- **Intermediate CA**: Signs all certificates, signed by offline root CA

### Certificate Transparency Service

**Purpose**: Immutable audit log of all issued certificates
**Repository**: `services/certtransparency/`  
**Port**: 8800

#### Responsibilities
- **Certificate Logging**: Permanent record of all issued certificates
- **Public Audit Trail**: Read-only access to certificate history
- **Search and Filter**: Query certificates by user, date, fingerprint
- **Compliance Support**: Audit trail for security compliance requirements

#### Key Components
- **Flask API Service**: RESTful interface for certificate logging
- **PostgreSQL Storage**: Persistent storage for certificate records
- **Search Interface**: Query and filter capabilities
- **API Authentication**: Write operations protected by shared secret

#### Data Model
- **Certificate Records**: Complete certificate details and metadata
- **Issuance Metadata**: Timestamp, requesting service, certificate purpose
- **Search Indexes**: Optimized queries by common search criteria

#### API Endpoints
- `GET /api/v1/certificates` - List and search certificates (public read)
- `POST /api/v1/certificates` - Log new certificate (authenticated write)
- `GET /api/v1/certificates/{id}` - Retrieve specific certificate details

## 🛠️ Supporting Tools

### PKI Tool

**Purpose**: Offline root and intermediate CA generation
**Location**: `tools/pki_tool/`
**Type**: Command-line tool

#### Functionality
- **Root CA Generation**: Create offline root certificate authority
- **Intermediate CA Generation**: Create signing intermediate CA
- **Key Types**: Support for Ed25519 and RSA (2048/4096) keys
- **Security**: Encrypted private keys with passphrase protection

#### Usage
```bash
python generate_pki.py --help
python generate_pki.py --output-dir ./pki
```

### OpenVPN Config Tool

**Purpose**: Client CLI for profile and server bundle retrieval
**Location**: `tools/get_openvpn_config/`
**Type**: Command-line client

#### Functionality
- **User Profiles**: OIDC authentication and profile download for end users
- **Server Bundles**: PSK-based authentication for server configuration retrieval
- **Configuration Management**: Multi-environment configuration support
- **Secure Storage**: Safe handling of authentication tokens and PSKs

#### Usage
```bash
# User profile retrieval
python get_openvpn_config.py --server https://vpn.example.com
python get_openvpn_config.py --server https://vpn.example.com --options tcp

# Server bundle retrieval
python get_openvpn_config.py --server https://vpn.example.com --psk your-psk-here
```

## 🔄 Service Interactions

### User Certificate Generation Flow

1. **User Request**: User authenticates via OIDC and requests certificate
2. **Frontend Validation**: Frontend validates user session and permissions  
3. **Signing Request**: Frontend calls signing service with certificate request
4. **Certificate Generation**: Signing service generates certificate using intermediate CA
5. **CT Logging**: Signing service automatically logs certificate to transparency service
6. **Profile Generation**: Frontend combines certificate with OpenVPN configuration
7. **Secure Download**: User receives time-limited download token for profile

### Server Bundle Generation Flow

1. **PSK Authentication**: Server authenticates using pre-shared key
2. **Bundle Request**: Server requests configuration bundle via API
3. **Server Certificate**: Signing service generates server certificate
4. **CT Logging**: Certificate automatically logged for audit
5. **Bundle Assembly**: Frontend assembles complete server configuration
6. **Secure Delivery**: Server receives tar archive with certificates and configuration

### Certificate Transparency Audit Flow

1. **Certificate Issuance**: Any certificate generated by signing service
2. **Automatic Logging**: Signing service posts certificate details to CT service
3. **Immutable Storage**: CT service stores certificate in audit log
4. **Public Visibility**: Certificates visible through frontend admin interface
5. **Search and Filter**: Administrators can query certificates by various criteria

## 🗄️ Data Flow and Storage

### Frontend Database (PostgreSQL)
- User sessions and OIDC identity mapping
- Pre-shared keys for server authentication
- Download tokens for secure file transfer
- Service configuration and feature flags

### Signing Service Storage
- Intermediate CA private key (encrypted with passphrase)
- Intermediate CA certificate (public)
- Temporary certificate generation workspace

### Certificate Transparency Database (PostgreSQL)  
- Complete certificate records with full PEM data
- Certificate metadata (fingerprints, serial numbers, validity periods)
- Issuance metadata (timestamps, requesting service, purpose)
- Search indexes for efficient querying

## 🔐 Security Architecture

### Network Security
- **Service Isolation**: Each service runs in isolated containers/pods
- **API Authentication**: Shared secrets for service-to-service communication
- **TLS Termination**: HTTPS termination at load balancer/ingress
- **Database Encryption**: Encrypted connections to PostgreSQL

### Identity and Access Management
- **OIDC Integration**: Enterprise identity provider integration
- **Group-based Authorization**: Admin permissions via OIDC group membership
- **Session Management**: Secure session handling with proper expiration
- **API Keys**: Pre-shared keys for server authentication

### Certificate Security
- **Key Isolation**: Private keys never leave signing service
- **Encrypted Storage**: CA private keys encrypted with passphrases
- **Audit Trail**: Complete certificate issuance logging

### Operational Security
- **Health Checks**: Service health monitoring and automatic restart
- **Log Management**: Structured logging for security monitoring
- **Secret Management**: External secret management integration ready
- **Update Management**: Container-based updates with rollback capability

## 📊 Scalability Considerations

### Horizontal Scaling
- **Stateless Services**: Frontend and CT services are horizontally scalable
- **Database Scaling**: PostgreSQL read replicas for transparency service
- **Load Balancing**: Multiple frontend instances behind load balancer

### Vertical Scaling
- **Resource Allocation**: Configurable CPU and memory limits
- **Database Performance**: Optimized queries and indexing
- **Certificate Caching**: Reduced cryptographic operations through caching

### Performance Optimizations
- **Async Operations**: Background processing for non-critical operations
- **Connection Pooling**: Database connection pooling for efficiency
- **Static Asset Serving**: CDN-ready static asset delivery

---

This architecture provides a solid foundation for secure, scalable OpenVPN certificate management while maintaining simplicity and operational efficiency.