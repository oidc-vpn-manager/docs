# 20. Three-Tier Testing Strategy

Date: 2025-09-02

## Status

Approved

## Context

To achieve the project's goal of high code quality and reliability, a structured and comprehensive approach to testing is required. Simply writing tests is not enough; they must be organized into a coherent strategy.

## Decision

We will adopt a formal three-tier testing strategy, consisting of:
1.  **Unit tests:** Testing individual functions and methods in isolation.
2.  **Functional/Integration tests:** Testing chains of functions, modules, and interactions with external systems like databases or service APIs.
3.  **End-to-End (E2E) tests:** Testing complete user workflows from the perspective of the user, validating the entire integrated system.

## Consequence

This layered approach ensures that all parts of the application are tested at the appropriate level of granularity. It helps to isolate failures more effectively, making debugging easier and faster. It provides a clear framework for developers when writing tests and contributes directly to meeting the project's quality assurance goals.
