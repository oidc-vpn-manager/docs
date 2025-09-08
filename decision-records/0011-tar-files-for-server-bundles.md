# 11. TAR files for Server Bundles

Date: 2025-09-02

## Status

Draft

## Context

The system needs to deliver a collection of files to an OpenVPN server as a single, convenient unit. This "Server Bundle" includes multiple OpenVPN configuration files, the server's unique certificate and key, CA chain files, and potentially a TLS key. A method is needed to package these files for simple distribution.

## Decision

We will package the Server Bundle as a TAR archive (`.tar` file). The `get_openvpn_config` tool will be responsible for downloading this single file and extracting its contents on the target server.

## Consequence

Using TAR is a standard, well-supported format on Linux systems, which are the primary target for OpenVPN servers. This simplifies the distribution and consumption process, as standard command-line tools can be used to handle the archive. This avoids the need to create a custom packaging format or require the client to make multiple downloads.
