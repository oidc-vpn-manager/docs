# 24. Container Orchestration and Deployment Strategy

Date: 2025-09-04

## Status

Approved

## Context

The OIDC VPN Manager microservices architecture requires orchestration and deployment strategies that support different environments from development to enterprise production. The system consists of multiple services (frontend, signing, Certificate Transparency) that need coordinated deployment, health checking, and scaling.

Current implementation provides Docker Compose configurations for development and testing, with Helm charts available for Kubernetes deployments. A clear strategy is needed to define the supported deployment patterns and their appropriate use cases.

## Decision

We will support **multiple container orchestration strategies** based on deployment requirements and organizational capacity.

### Supported Orchestration Platforms

**Docker Compose** (Primary for Small-Scale Deployments):
- **Use Cases**: Development, testing, small production deployments (<10 users)
- **Benefits**: Simple configuration, minimal infrastructure requirements, easy debugging
- **Limitations**: Single-host deployment, manual scaling, basic health checking
- **Configuration**: `docker-compose.yml` files in `/tests` and `/deploy/with-docker`

**Kubernetes with Helm** (Primary for Production Deployments):  
- **Use Cases**: Production deployments, high availability, enterprise scale
- **Benefits**: Automatic scaling, rolling updates, service discovery, health checking
- **Requirements**: Kubernetes cluster, Helm package manager
- **Configuration**: Helm charts in `/deploy/with-helm`

### Container Strategy

**Base Images**:
- **Python Services**: Official Python Alpine images for minimal attack surface
- **Database**: Official PostgreSQL images with version pinning
- **Updates**: Regular base image updates with security patch monitoring

**Multi-Stage Builds**:
- **Build Stage**: Include development dependencies, run tests, build assets
- **Runtime Stage**: Minimal runtime environment with only production dependencies
- **Benefits**: Smaller production images, reduced attack surface

**Health Checks**:
- **Application Health**: Each service exposes `/health` endpoint
- **Dependency Health**: Services check database connectivity and inter-service communication
- **Startup Probes**: Kubernetes-compatible health check endpoints
- **Graceful Shutdown**: Proper signal handling for container lifecycle management

### Deployment Patterns

**Development Environment**:
- **Method**: Docker Compose with development configurations
- **Features**: Hot reload, debug ports, test databases, mock OIDC provider
- **Command**: `docker-compose -f tests/docker-compose.yml up -d`

**Testing Environment**:
- **Method**: Docker Compose with production-like configurations
- **Features**: End-to-end testing, service separation testing, realistic networking
- **Integration**: CI/CD pipeline compatible

**Production Environment**:
- **Method**: Kubernetes with Helm charts
- **Features**: Rolling deployments, horizontal pod autoscaling, persistent volumes
- **High Availability**: Multiple replicas, node affinity rules, disruption budgets

### Service Discovery and Communication

**Docker Compose**:
- **Method**: Docker network with service names as hostnames
- **Configuration**: Environment variables for service URLs

**Kubernetes**:
- **Method**: Kubernetes Services with DNS-based service discovery
- **Configuration**: ConfigMaps for service URLs and configuration
- **Load Balancing**: Kubernetes Service load balancing for multiple replicas

## Consequences

**Positive Consequences**:
- **Flexibility**: Organizations can choose appropriate orchestration based on scale and expertise
- **Scalability**: Kubernetes support enables enterprise-scale deployments
- **Development Productivity**: Docker Compose provides fast development iterations
- **Operational Maturity**: Production deployments benefit from Kubernetes ecosystem tools

**Negative Consequences**:
- **Complexity**: Supporting multiple orchestration platforms requires testing and documentation overhead
- **Learning Curve**: Teams must understand their chosen orchestration platform
- **Configuration Drift**: Multiple deployment methods risk configuration inconsistencies

**Implementation Requirements**:
- All services must support containerized deployment with proper health checks
- Configuration must be externalized through environment variables and config files
- Services must handle graceful startup and shutdown in container environments
- Documentation must provide clear deployment instructions for each supported platform
- CI/CD pipelines must test deployments on both Docker Compose and Kubernetes