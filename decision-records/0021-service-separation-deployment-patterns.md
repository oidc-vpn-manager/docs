# 21. Service Separation Deployment Patterns

Date: 2025-09-04

## Status

Approved

## Context

The OIDC VPN Manager frontend service was originally designed as a monolithic service serving both user-facing and administrator-facing functionality in a single deployment. As organizations scale their VPN operations, there is a need to separate user-facing services from administrative services for security isolation, different scaling requirements, and operational separation.

Users typically need to access profile generation and certificate downloads frequently, while administrators need access to certificate management, PSK creation, and audit logs. These different user types have different security requirements, traffic patterns, and scaling needs.

## Decision

We will implement service separation deployment patterns that allow the frontend service to be deployed in three configurations:

1. **Combined Service**: Single deployment serving both user and admin functionality (backward compatibility)
2. **User Service**: Deployment focused on user-facing functionality with admin routes rejected
3. **Admin Service**: Deployment focused on administrative functionality with user routes redirected

### Implementation Details

**Configuration-Based Separation**: Services determine their role through environment variables:
- `ADMIN_URL_BASE`: When configured, indicates this deployment is user-focused and should redirect admin users to the admin service
- `USER_URL_BASE`: When configured, indicates this deployment is admin-focused and should redirect user routes to the user service
- No separation URLs: Combined service mode (backward compatibility)

**Route Protection**: Flask decorators control access patterns:
- `admin_service_only`: Returns 403 Forbidden when accessed on user service
- `user_service_only`: Returns 301 redirect when accessed on admin service
- `redirect_admin_to_admin_service`: Redirects admin users from user service to admin service
- API-specific variants for programmatic clients

**Service Behavior**:
- **User Service**: Accepts user routes, rejects admin routes (403), redirects admin users to admin service
- **Admin Service**: Accepts admin routes, redirects user routes to user service (301)
- **Combined Service**: No restrictions, serves all routes normally

## Consequences

**Positive Consequences**:
- **Security Isolation**: Sensitive administrative functionality can be deployed separately from user-facing services
- **Independent Scaling**: User and admin services can scale independently based on their specific load patterns
- **Network Separation**: Services can be deployed in different network zones with appropriate security controls
- **Operational Flexibility**: Different teams can manage user vs admin services with different update cycles
- **Backward Compatibility**: Existing deployments continue to work without changes

**Negative Consequences**:
- **Deployment Complexity**: Additional configuration required for separated deployments
- **Cross-Service Dependencies**: Admin users accessing user service require proper redirect handling
- **URL Management**: Organizations must manage multiple service URLs and ensure proper DNS/load balancer configuration
- **Testing Complexity**: E2E tests must validate behavior across all three deployment patterns

**Implementation Requirements**:
- All routes must be properly decorated with appropriate service separation decorators
- E2E tests must validate all three deployment configurations
- Documentation must clearly explain configuration options and use cases
- Health checks must work across all deployment patterns