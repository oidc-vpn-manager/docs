# 5. Jinja2 for Configuration Templating

Date: 2025-09-02

## Status

Approved

## Context

The frontend service needs to dynamically generate OpenVPN configuration files (`.ovpn`). These files must be customized based on user roles (derived from OIDC groups) and user-selected options from the web interface. A flexible and powerful templating engine is required to handle this logic.

## Decision

We will use the Jinja2 templating language to construct the `.ovpn` configuration files. Templates will be stored in the filesystem and selected based on OIDC group membership, with a fallback to a default template.

## Consequence

Jinja2 provides a clear and powerful syntax for embedding logic within the configuration files, separating the data and presentation layers effectively. This makes the templates easy to read, maintain, and extend. The system will depend on the Jinja2 library, and developers will need to be familiar with its syntax and features.
