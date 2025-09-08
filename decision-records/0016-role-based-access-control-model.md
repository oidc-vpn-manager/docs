# 16. Role-Based Access Control Model

Date: 2025-09-02

## Status

Approved

## Context

The application requires a method to control user access to different features and data. A simple authenticated vs. unauthenticated model is insufficient given the variety of user responsibilities, from regular users requesting profiles to administrators managing the system. A clear authorization model is needed.

## Decision

We will implement a Role-Based Access Control (RBAC) model. A specific set of roles are defined: User, Auditor, Certificate Administrator, System Administrator, and Service Administrator. A user's role is determined by the group claims received from the OIDC provider during authentication, which are then mapped to these internal application roles.

## Consequence

This provides a clear and manageable way to control access to application features. Permissions are grouped logically by role, which simplifies administration and auditing. Changes to a user's permissions can be managed centrally by changing their group membership in the external identity provider, without requiring changes to the application itself.
