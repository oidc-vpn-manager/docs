# 22. Database Technology and Schema Management Strategy

Date: 2025-09-04

## Status

Approved

## Context

The OpenVPN Manager microservices architecture requires persistent data storage for multiple services. The frontend service needs to store user sessions, PSKs, download tokens, and configuration. The Certificate Transparency service needs to store an immutable audit log of all certificate operations with efficient search capabilities.

Different services have different data consistency, performance, and scalability requirements. The choice of database technology and schema management approach affects deployment complexity, operational overhead, and system reliability.

## Decision

We will use **PostgreSQL as the primary database technology** for all services requiring relational data storage, with **SQLite for development and testing environments**.

### Database Assignment by Service

**Frontend Service Database**:
- **Technology**: PostgreSQL (production), SQLite (development/testing)
- **Purpose**: User sessions, OIDC identity mapping, PSKs, download tokens, service configuration
- **Schema**: Relational with foreign key constraints
- **Migration Strategy**: Flask-Migrate with Alembic for schema versioning

**Certificate Transparency Service Database**:
- **Technology**: PostgreSQL (all environments)
- **Purpose**: Immutable certificate audit log with search indexes
- **Schema**: Optimized for append-only operations with search indexes
- **Migration Strategy**: Flask-Migrate with Alembic for schema versioning

**Signing Service Storage**:
- **Technology**: File system with encrypted storage
- **Purpose**: Intermediate CA private key and certificate storage
- **Justification**: PKI materials require file-based storage patterns, minimal metadata

### Schema Management Approach

**Migration Strategy**:
- Use Flask-Migrate (Alembic) for all database schema changes
- Version controlled migration scripts in each service repository
- Automated migration execution during service startup
- Rollback capability for schema changes

**Development vs Production**:
- **Development/Testing**: SQLite for simplicity and portability
- **Production**: PostgreSQL for reliability, performance, and concurrent access
- **CI/CD**: PostgreSQL containers for integration testing

**Data Consistency**:
- Each service owns its database schema independently
- No direct database access between services (API-only communication)
- Eventual consistency acceptable between services via API calls

## Consequences

**Positive Consequences**:
- **Operational Simplicity**: Single database technology to learn and maintain
- **ACID Compliance**: PostgreSQL provides strong consistency guarantees for critical data
- **Performance**: Optimized queries and indexing capabilities for audit log searches
- **Development Productivity**: SQLite enables lightweight development and testing
- **Schema Evolution**: Versioned migrations enable safe database updates

**Negative Consequences**:
- **PostgreSQL Dependency**: Additional operational complexity compared to file-based storage
- **Resource Requirements**: PostgreSQL requires more memory and storage than simpler alternatives
- **Development Overhead**: Migration scripts must be maintained for schema changes

**Operational Requirements**:
- PostgreSQL backup and recovery procedures must be established
- Database connection pooling required for production deployments
- Monitoring and alerting for database performance and availability
- Regular maintenance tasks (VACUUM, index optimization) must be scheduled