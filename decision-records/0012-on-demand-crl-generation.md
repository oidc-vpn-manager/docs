# 12. On-Demand CRL Generation

Date: 2025-09-02

## Status

SUPERCEDED BY [0019](0019-direct-crl-generation-from-ct-data.md)

## Context

The Certificate Revocation List (CRL) is a critical security component that allows OpenVPN servers to deny access to clients with revoked certificates. The CRL must always be up-to-date with the latest information from the Certificate Transparency (CT) log to be effective.

## Decision

The CRL will be generated on-demand. When an unauthenticated request for the CRL is received by the frontend, it will query the CT log for all current revocation events, generate a new CRL file, have it signed by the `signing` service, and then return it to the client.

## Consequence

This approach ensures that the provided CRL is always up-to-date, reflecting the exact state of the CT log at the time of the request. It eliminates the complexity of caching and potential cache invalidation issues. The trade-off is a potential for slightly increased latency on CRL requests, as the file is generated dynamically rather than being served from a static location.
