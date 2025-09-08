# 17. API Key Authentication Between Services

Date: 2025-09-02

## Status

Approved

## Context

In our microservices architecture, services need to communicate with each other securely over the network. For example, the `frontend` needs to make trusted requests to the `signing` service, and the `signing` service needs to make trusted write requests to the `certtransparency` service. A machine-to-machine authentication mechanism is required.

## Decision

We will use static API keys for authentication between services. A service making a request to another will include a pre-configured, shared secret API key in the request headers. The receiving service will validate this key to authenticate the request.

## Consequence

API keys are a simple and widely understood method for service-to-service authentication that is easy to implement. However, this means the keys must be treated as secrets and managed securely during deployment. Key rotation is a manual process that must be managed by system administrators.
