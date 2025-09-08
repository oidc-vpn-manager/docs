# 8. Swagger for API Documentation

Date: 2025-09-02

## Status

Approved

## Context

With a microservices architecture, the APIs between services act as contracts. Clear, consistent, and up-to-date documentation for these APIs is essential for developers to understand, use, and maintain the services effectively. Without it, development and integration would be slow and error-prone.

## Decision

All API endpoints must be documented using the Swagger (OpenAPI) specification. Each service will expose its own `swagger.yaml` file, and an API index page will link to the documentation for the implemented API version.

## Consequence

This provides a standardized, machine-readable format for API documentation. It enables the automatic generation of interactive API UIs, client SDKs, and server stubs, which can accelerate development. It enforces a design-first approach to API development and serves as a reliable reference for all inter-service communication.
