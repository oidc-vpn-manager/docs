# 23. Secrets Management and Rotation Strategy

Date: 2025-09-04

## Status

Approved

## Context

The OpenVPN Manager system requires secure handling of multiple types of secrets: API keys for inter-service communication, OIDC client secrets, database passwords, CA private keys, and TLS certificates. These secrets have different rotation requirements, security sensitivity levels, and access patterns.

Current implementation stores secrets in environment variables and configuration files, which presents security risks for production deployments. A comprehensive secrets management strategy is needed to ensure secure storage, rotation, and access control.

## Decision

We will implement a **layered secrets management strategy** that supports different deployment environments while maintaining security best practices.

### Secret Categories and Handling

**High-Security Secrets** (CA Private Keys):
- **Storage**: Encrypted at rest with passphrase protection
- **Access**: Only signing service has access
- **Rotation**: Manual process with service restart required
- **Backup**: Encrypted offline storage with key escrow procedures

**Inter-Service API Keys**:
- **Storage**: Environment variables or external secrets manager
- **Access**: Shared between specific service pairs only
- **Rotation**: Automated rotation capability with graceful failover
- **Length**: Minimum 32 characters, cryptographically random

**OIDC and External Integration Secrets**:
- **Storage**: Environment variables or external secrets manager  
- **Access**: Frontend service only
- **Rotation**: Coordinated with external identity provider
- **Validation**: Startup-time validation of secret validity

**Database Credentials**:
- **Storage**: Environment variables or external secrets manager
- **Access**: Service-specific database users with minimal privileges
- **Rotation**: Database-native rotation with connection pool refresh

### Deployment Environment Support

**Development/Testing**:
- **Method**: Environment variables and `.env` files
- **Justification**: Simplicity and developer productivity
- **Security**: Non-production secrets, container isolation

**Production**:
- **Method**: External secrets management system (Kubernetes Secrets, HashiCorp Vault, AWS Secrets Manager)
- **Justification**: Centralized management, audit trails, rotation automation
- **Integration**: Environment variable injection with secrets manager integration

### Secret Rotation Procedures

**API Key Rotation**:
1. Generate new API key
2. Configure both old and new keys in receiving service
3. Update sending service to use new key
4. Remove old key from receiving service after validation period
5. Monitor for authentication failures during transition

**Database Credential Rotation**:
1. Create new database user with identical permissions
2. Update service configuration with new credentials
3. Restart service with new credentials
4. Remove old database user after confirmation

**CA Key Protection**:
- CA private keys are encrypted with passphrases
- Passphrase stored separately from encrypted key
- Key material never transmitted over network
- Regular backup verification procedures

## Consequences

**Positive Consequences**:
- **Security**: Reduced risk of secret exposure in code repositories and logs
- **Compliance**: Audit trail for secret access and rotation activities
- **Operational**: Automated rotation reduces manual effort and human error
- **Flexibility**: Supports different deployment patterns from development to enterprise

**Negative Consequences**:
- **Complexity**: Additional infrastructure required for production secret management
- **Dependencies**: Reliance on external systems for secret storage and retrieval
- **Development Overhead**: Developers must understand secret management patterns

**Implementation Requirements**:
- Services must gracefully handle secret rotation without downtime
- Startup health checks must validate secret availability and validity
- Logging must never expose secret values
- Documentation must provide clear procedures for secret rotation in each environment