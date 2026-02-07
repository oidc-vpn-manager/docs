# Using OIDC VPN Manager - Administrator Guide

This guide covers administrative functions in OIDC VPN Manager, including pre-shared key (PSK) management, certificate transparency review, and system monitoring.

## 🔐 Administrator Access

Administrative functions require special permissions granted through your OIDC identity provider. Users must be members of the configured admin group (typically `vpn-admins` or similar) to access administrative features.

### Accessing Admin Functions

1. **Authenticate** using your admin account
2. **Navigate** to the admin section (admin menu will appear for privileged users)
3. **Access** admin-only features like PSK management and certificate review

Admin functions are clearly marked and only visible to users with appropriate permissions.

## 🔑 Pre-Shared Key (PSK) Management

Pre-shared keys enable OpenVPN servers to authenticate with the management system and retrieve their configuration bundles. Each server deployment should have its own unique PSK.

### Creating New PSKs

1. **Navigate** to Admin → Pre-Shared Keys
2. **Click** "Generate New PSK"
3. **Fill out** the PSK details:
   - **Description**: Meaningful identifier (e.g., "Production Server - US East")
   - **Purpose**: Brief explanation of intended use
   - **Expiry**: Optional expiration date for automatic cleanup

4. **Generate** the PSK
5. **Copy** the generated key immediately (it won't be shown again)
6. **Store** the PSK securely in your configuration management system

### PSK Security Best Practices

- **Unique PSKs**: Generate separate PSKs for each server or environment
- **Secure Storage**: Store PSKs in your organization's secret management system
- **Regular Rotation**: Regenerate PSKs periodically as part of security maintenance
- **Access Control**: Limit who can view and generate PSKs
- **Audit Trail**: Monitor PSK usage through certificate transparency logs

### Managing Existing PSKs

The PSK management interface allows you to:

- **View** all active PSKs with their descriptions and creation dates
- **Revoke** compromised or unused PSKs
- **Set expiration dates** for automatic cleanup
- **Monitor usage** by reviewing associated server certificate requests

### Server Bundle Retrieval

Servers use PSKs to authenticate and retrieve their configuration bundles:

```bash
# Server-side command (using the CLI tool)
python get_openvpn_config.py get-psk-profile \
  --server-url https://vpn.yourcompany.com \
  --hostname server01.vpn.yourcompany.com \
  --psk your-generated-psk-here \
  --target-dir /etc/openvpn
```

This command:
- Authenticates using the PSK
- Requests a server certificate for the specified hostname
- Downloads the complete server configuration bundle
- Extracts files to the appropriate directories

## 📋 Certificate Transparency Review

The Certificate Transparency service maintains an immutable audit log of all issued certificates. This feature supports compliance requirements and security monitoring.

### Viewing Certificate Logs

1. **Navigate** to Admin → Certificate Transparency
2. **Browse** issued certificates with details including:
   - **Certificate fingerprint** (SHA-256)
   - **Subject** (username or hostname)
   - **Issuer** (Intermediate CA details)
   - **Validity period** (not before / not after dates)
   - **Serial number** (unique certificate identifier)
   - **Issuance timestamp** (when the certificate was issued)

### Search and Filter Options

Use the certificate search interface to:

- **Filter by date range**: Find certificates issued within specific timeframes
- **Search by username**: Locate all certificates for a specific user
- **Filter by certificate type**: Distinguish between user and server certificates
- **Search by hostname**: Find server certificates for specific hosts
- **Sort by various fields**: Order results by date, user, or other attributes

### Certificate Details View

Click on any certificate to view comprehensive details:

- **Full certificate content** in PEM format
- **Certificate chain** showing trust path to root CA
- **Key usage extensions** and certificate purposes
- **Subject Alternative Names** (SANs) for server certificates
- **Requesting service** that initiated certificate generation

### Compliance and Auditing

The Certificate Transparency logs support:

- **SOX compliance**: Complete audit trail of certificate issuance
- **Security incident response**: Tracking certificate usage during investigations
- **Access reviews**: Verifying appropriate certificate distribution
- **Compliance reporting**: Generating reports for auditors and management

## 🖥️ Server Configuration Management

### OpenVPN Server Setup

For each OpenVPN server deployment:

1. **Generate a PSK** using the admin interface
2. **Deploy the PSK** to your server configuration management
3. **Configure the server** to retrieve its bundle on startup
4. **Verify connectivity** to the signing and certificate transparency services

### Server Bundle Contents

Each server bundle includes:

- **Server certificate**: Unique certificate for the specific hostname
- **Certificate chain**: Intermediate and root CA certificates
- **OpenVPN configurations**: Both UDP (port 1194) and TCP (port 443) configurations
- **TLS authentication**: Additional security layer for OpenVPN connections

### Automated Server Deployment

Integrate server bundle retrieval into your deployment pipeline:

```yaml
# Example Kubernetes deployment snippet
apiVersion: apps/v1
kind: Deployment
metadata:
  name: openvpn-server
spec:
  template:
    spec:
      initContainers:
      - name: fetch-config
        image: openvpn-config-fetcher:latest
        command:
        - python
        - get_openvpn_config.py
        - get-psk-profile
        - --server-url
        - https://vpn.yourcompany.com
        - --hostname
        - $(HOSTNAME)
        - --psk
        - $(PSK_SECRET)
        - --target-dir
        - /shared/openvpn-config
        volumeMounts:
        - name: config-volume
          mountPath: /shared/openvpn-config
        env:
        - name: HOSTNAME
          value: "vpn-server-01.yourcompany.com"
        - name: PSK_SECRET
          valueFrom:
            secretKeyRef:
              name: openvpn-psk
              key: psk
```

## 📊 System Monitoring

### Health Monitoring

Monitor OIDC VPN Manager component health:

- **Service availability**: Ensure all microservices are responding
- **Database connectivity**: Verify PostgreSQL connections
- **Certificate generation rate**: Monitor signing service performance
- **Authentication success rate**: Track OIDC authentication metrics

### Key Metrics to Track

1. **Certificate Issuance Metrics**:
   - Certificates issued per day/hour
   - Success vs failure rate
   - Average generation time
   - Peak usage periods

2. **Authentication Metrics**:
   - Successful OIDC authentications
   - Failed authentication attempts
   - Session duration statistics
   - Admin vs user access patterns

3. **System Performance**:
   - Service response times
   - Database query performance
   - Memory and CPU utilization
   - Storage usage trends

### Log Management

Configure centralized logging for:

- **Security events**: Authentication failures, certificate access
- **Operational events**: Service starts, stops, errors
- **Audit events**: Admin actions, certificate issuance
- **Performance events**: Slow queries, high resource usage

### Alerting

Set up alerts for:

- **Service failures**: Any microservice becoming unavailable
- **Authentication anomalies**: Unusual failed login patterns
- **Certificate generation failures**: Issues with signing service
- **Database connectivity**: PostgreSQL connection problems
- **Resource exhaustion**: High CPU, memory, or disk usage

## 🔧 Maintenance Tasks

### Regular Maintenance

**Weekly Tasks**:
- Review certificate transparency logs for anomalies
- Check PSK usage and clean up unused keys
- Monitor service health and performance metrics
- Review authentication logs for security issues

**Monthly Tasks**:
- Rotate service-to-service API secrets
- Review and update admin group memberships
- Audit certificate issuance patterns
- Update documentation and procedures

**Quarterly Tasks**:
- Review and rotate PSKs for active servers
- Audit admin access logs and permissions
- Test disaster recovery procedures
- Review security configurations and updates

### Security Maintenance

1. **Secret Rotation**:
   - Rotate API secrets between services
   - Update PSKs for server authentication
   - Refresh OIDC client secrets
   - Update database passwords

2. **Certificate Management**:
   - Monitor intermediate CA certificate expiration
   - Plan root CA renewal procedures (multi-year horizon)
   - Review certificate validity periods
   - Test certificate chain validation

3. **Access Review**:
   - Audit admin group memberships
   - Review PSK assignments and usage
   - Verify OIDC group mappings
   - Check service account permissions

## ❗ Troubleshooting

### Common Admin Issues

**PSK Authentication Failures**:
- Verify PSK is correctly configured on server
- Check network connectivity from server to management service
- Review PSK expiration status
- Confirm hostname matches certificate request

**Certificate Generation Errors**:
- Check signing service health and logs
- Verify intermediate CA certificate and key are accessible
- Confirm Certificate Transparency service connectivity
- Review signing service API authentication

**Database Issues**:
- Monitor PostgreSQL connection limits
- Check database disk space and performance
- Review migration status and schema versions
- Verify database backup and recovery procedures

### Emergency Procedures

**Service Outage Response**:
1. Check overall system health and dependencies
2. Review service logs for error patterns
3. Verify database connectivity and status
4. Coordinate with infrastructure team for resolution
5. Communicate status to affected users

**Security Incident Response**:
1. Review Certificate Transparency logs for unauthorized certificates
2. Identify potentially compromised PSKs or certificates
3. Revoke compromised authentication materials
4. Generate new PSKs and certificates as needed
5. Document incident and lessons learned

## 📞 Getting Help

### Internal Resources

- **System logs**: Check service logs for detailed error information
- **Health endpoints**: Use service health checks for quick status
- **Database monitoring**: Review PostgreSQL performance metrics
- **Certificate transparency**: Search CT logs for certificate history

### External Support

For issues requiring external support:

- **Infrastructure problems**: Contact your cloud or infrastructure provider
- **OIDC integration**: Work with your identity provider support team
- **Security concerns**: Engage your organization's security team
- **Database issues**: Consult PostgreSQL support resources

---

**Administrative Excellence**: Effective OIDC VPN Manager administration ensures secure, reliable certificate management for your organization. Regular monitoring, maintenance, and security practices keep the system running smoothly and securely.

## 🤖 AI Assistance Disclosure

This documentation was developed with assistance from AI tools. While released under a permissive license that allows unrestricted reuse, we acknowledge that portions of the implementation may have been influenced by AI training data. Should any copyright assertions or claims arise regarding uncredited imported code, the affected portions will be rewritten to remove or properly credit any unlicensed or uncredited work.