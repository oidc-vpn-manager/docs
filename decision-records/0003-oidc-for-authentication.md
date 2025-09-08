# 3. OIDC for Authentication

Date: 2025-09-02

## Status

Approved

## Context

The frontend service requires a robust, secure, and standardized method to authenticate users and determine their roles and permissions within the application. A custom authentication solution would be complex to build and maintain, and could introduce security vulnerabilities.

## Decision

We will use OpenID Connect (OIDC) as the exclusive protocol for user authentication. The frontend service will delegate the authentication process to an external, OIDC-compliant Identity Provider (IdP). User roles will be determined by the group claims returned in the OIDC token.

## Consequence

This decision decouples the application from a specific identity provider, allowing for integration with various IdPs (e.g., Keycloak, Okta, Google). It simplifies the application by offloading the complexities of password management and authentication flows. All user-facing components that require authentication must be able to initiate and handle the OIDC flow.
