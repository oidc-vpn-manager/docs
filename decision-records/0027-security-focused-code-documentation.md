# 27. Security-Focused Code Documentation

Date: 2025-09-21

## Status

Approved

## Context

During the comprehensive docstring audit, we identified inconsistent documentation of security considerations throughout the codebase. Critical security functions lacked documentation about timing attack prevention, input validation strategies, and cryptographic algorithm choices.

Security documentation gaps included:
- Timing attack prevention in authentication functions
- Template injection prevention in configuration rendering
- Cryptographic algorithm selection rationale
- Input validation and sanitization strategies
- Cross-service authentication mechanisms
- Constant-time comparison implementations

The OpenVPN Manager handles sensitive cryptographic operations and certificate management, making security documentation critical for maintaining system integrity and enabling security reviews.

## Decision

We will implement security-focused documentation for all security-relevant code:

1. **Cryptographic Functions:**
   - Algorithm selection rationale (Ed25519 vs RSA vs ECDSA)
   - Key generation entropy requirements
   - Signature verification processes
   - Certificate validation procedures
   - Random number generation security

2. **Authentication and Authorization:**
   - Timing attack prevention mechanisms
   - Constant-time comparison usage
   - Session management security
   - OIDC integration security considerations
   - PSK verification security measures

3. **Input Validation and Sanitization:**
   - Template injection prevention
   - Path traversal prevention
   - SQL injection prevention
   - Parameter validation strategies
   - XSS prevention measures

4. **Cross-Service Communication:**
   - API authentication mechanisms
   - Secret sharing and rotation
   - Request/response validation
   - Error handling without information disclosure

5. **Documentation Requirements:**
   - Explicit security considerations in function docstrings
   - Rationale for security-related implementation choices
   - References to relevant security standards (OWASP Top 10)
   - Example attack scenarios and prevention mechanisms
   - Performance vs security trade-off explanations

## Consequences

### Positive
- Enhanced security awareness during development
- Improved security review efficiency with documented considerations
- Better understanding of security implementation rationale
- Easier identification of security-critical code sections
- Reduced risk of security regressions during maintenance
- Clear guidance for security-related code modifications

### Negative
- Additional documentation overhead for security-related functions
- Need for security expertise during documentation reviews
- Potential information disclosure if documentation is too detailed

### Implementation Guidelines

1. **Documentation Depth:**
   - Focus on "why" security measures are implemented
   - Avoid exposing specific attack vectors in detail
   - Reference standard security practices and frameworks
   - Include performance implications of security measures

2. **Review Process:**
   - Security team review for cryptographic function documentation
   - Code review enforcement of security documentation standards
   - Regular audits of security documentation completeness
   - Update documentation when security measures change

3. **Examples of Required Documentation:**
   ```python
   def verify_key(self, plaintext_key):
       """
       Verify a plaintext key against stored hash using constant-time comparison.

       Uses hmac.compare_digest for timing-attack-resistant comparison.
       This prevents attackers from inferring key values through timing analysis.

       Security Notes:
           - Constant-time comparison prevents timing side-channel attacks
           - SHA256 hashing provides one-way key storage security
           - No key material exposure in error messages or logs
       """
   ```

### Success Metrics
- All cryptographic functions include algorithm selection rationale
- All authentication functions document timing attack prevention
- All input validation includes injection prevention documentation
- Security review time reduction through comprehensive documentation
- Zero security regressions due to undocumented security assumptions