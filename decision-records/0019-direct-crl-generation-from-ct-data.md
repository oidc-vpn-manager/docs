# 19. Direct CRL Generation from CT Data

Date: 2025-09-02

## Status

Approved

## Context

The Certificate Revocation List (CRL) must be generated based on the state of the Certificate Transparency (CT) log. An early design consideration, mentioned in development notes, was to recreate a traditional `index.txt` file from the CT log and use that as the input for the signing process, mimicking how OpenSSL's CA tools work.

## Decision

We will not use an intermediate `index.txt` file. Instead, the `frontend` service will query the `certtransparency` service for a structured list of revoked certificates. This list of records will be passed directly to the `signing` service, which will then use a cryptography library to build the CRL from this in-memory data.

## Consequence

This approach is more direct and avoids the overhead and potential for errors associated with parsing and maintaining the legacy `index.txt` format. It creates a cleaner, API-driven process between the services. The `signing` service contains the logic for building a CRL, while the `frontend` orchestrates the process.