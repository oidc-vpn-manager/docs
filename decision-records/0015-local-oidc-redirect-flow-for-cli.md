# 15. Local OIDC Redirect Flow for CLI

Date: 2025-09-02

## Status

Approved

## Context

The `get_openvpn_config` command-line tool needs to support an interactive OIDC authentication flow for users who are not using a Pre-Shared Key. As a command-line application, it cannot directly handle browser-based HTTP redirects that are central to the OIDC protocol.

## Decision

The CLI tool will implement the following flow: it will start a temporary, local web server on a high-numbered port. It will then open the system's default web browser to initiate the OIDC authentication flow, providing a `redirect_uri` that points to the local server. After the user authenticates, the OIDC provider redirects back to the local server, which captures the authentication token, provides it to the CLI tool, and then shuts down.

## Consequence

This pattern allows a command-line application to securely leverage a standard, browser-based OIDC flow without handling user credentials directly. It provides a good user experience by using the familiar browser interface for authentication. This adds some complexity to the CLI tool but is a standard and secure pattern for this type of authentication challenge.
