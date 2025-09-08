# 2. Microservices Architecture

Date: 2025-09-02

## Status

Approved

## Context

The OpenVPN Manager application is designed as a suite of services to provide secure access to computers and services in a managed environment. The application is composed of several distinct components with specific responsibilities: a frontend for user interaction and API access, a signing service for handling cryptographic operations, and a certificate transparency service for logging all certificate-related actions. This separation of concerns requires a clear architectural pattern to manage the interactions between components.

## Decision

We will adopt a microservices architecture. Each core component (frontend, signing, certificate transparency) will be developed, deployed, and scaled as an independent service. These services will communicate with each other over the network via well-defined APIs.

## Consequence

This approach allows for greater flexibility and scalability. Each service can be developed and updated independently, using the most appropriate technology for its specific task. It also allows for horizontal scaling of individual services to meet demand. However, it introduces complexity in terms of deployment, monitoring, and ensuring reliable inter-service communication. Future decisions will need to account for network latency, service discovery, and data consistency across services.
