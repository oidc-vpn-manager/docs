# 18. Convention for Naming Profile Templates

Date: 2025-09-02

## Status

Approved

## Context

The system needs a predictable, file-based method to find and select the correct OpenVPN profile template for a user based on their group membership as determined by the OIDC provider. The selection process needs to be ordered by priority.

## Decision

We will use a specific file naming convention for profile templates stored on the filesystem: `XXXX.GroupName.ovpn`. `XXXX` is a four-digit number that determines priority (lower numbers are checked first), and `GroupName` is the name of the user group. A `9999.default.ovpn` template will be used as a fallback if no group-specific template matches.

## Consequence

This convention provides a simple and transparent mechanism for mapping user groups to configuration templates without requiring a database. The file system itself acts as the source of truth for this logic. The application's template selection logic is directly tied to this naming scheme, and administrators must follow it precisely when adding or modifying templates.
