# 6. Offline Root CA with PKI Tool

Date: 2025-09-02

## Status

Approved

## Context

The security of the entire system relies on the trust of the Certificate Authority (CA). The Root CA is the ultimate source of trust and must be protected from any potential compromise. An online Root CA would present a significant and unnecessary security risk.

## Decision

We will use an offline Root CA. The `signing` service will use an Intermediate CA, signed by the offline Root, for its day-to-day certificate signing operations. A dedicated command-line `pki_tool` will be used to generate the Root CA and any Intermediate CAs.

## Consequence

Keeping the Root CA offline drastically reduces its attack surface, as it is never exposed to the network. This architecture allows the Intermediate CA to be rotated or replaced without invalidating the entire trust chain. It introduces an operational requirement to securely store and manage the offline Root CA key.
