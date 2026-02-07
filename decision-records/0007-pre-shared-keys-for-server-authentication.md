# 7. Pre-Shared Keys for Server Authentication

Date: 2025-09-02

## Status

Approved

## Context

Automated server provisioning requires a secure and non-interactive method for a new server to authenticate with the OIDC VPN Manager to obtain its initial configuration bundle. This process needs to be simple enough to be used in automated tools like Ansible, Puppet, or cloud-init scripts.

## Decision

We will use Pre-Shared Keys (PSKs) for the initial authentication of servers. A system administrator can generate a PSK via the frontend, which can then be used by the `get_openvpn_config` tool to authenticate and download a server configuration bundle without user interaction.

## Consequence

PSKs provide a straightforward mechanism for bootstrapping server authentication in an automated fashion. However, the security of this process depends on the secure distribution and storage of the PSK. The PSKs are single-use tokens for retrieving a server bundle, which limits their exposure.
