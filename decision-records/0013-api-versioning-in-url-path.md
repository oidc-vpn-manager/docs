# 13. API Versioning in URL Path

Date: 2025-09-02

## Status

Approved

## Context

As the application evolves, the APIs between services will inevitably change. A strategy is required to manage these changes gracefully, allowing clients to adapt over time without causing breaking changes for existing integrations.

## Decision

All API endpoints will be versioned using a mandatory path segment in the URL. The format will be `/api/vX/`, where `X` is the major version number of the API (e.g., `/api/v1/`).

## Consequence

This is a clear and explicit method of API versioning that is easy for both clients and servers to understand and implement. It allows clients to pin to a specific version of an API and upgrade at their own pace. It also enables the application to serve multiple versions of an API simultaneously, facilitating a smoother transition period when a new version is released.
