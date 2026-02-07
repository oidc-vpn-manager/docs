# 9. 100% Test Coverage

Date: 2025-09-02

## Status

Approved

## Context

The OIDC VPN Manager is a security-critical application responsible for managing network access. Bugs or regressions could have significant security implications. Therefore, a high degree of confidence in the correctness and reliability of the code is required.

## Decision

A 100% test coverage score is a required quality gate for all code in the services and tools. While not automatically enforced by local tooling, the expectation is that all code is fully tested, and pull requests will be evaluated against this standard. Any exceptions must be explicitly reviewed and approved by the project lead.

## Consequence

This decision forces developers to write testable code and ensures that every line of code is exercised by the test suite, significantly reducing the likelihood of regressions. While it increases initial development effort, it improves long-term code quality, maintainability, and reliability. It also provides a safety net for refactoring and adding new features.
