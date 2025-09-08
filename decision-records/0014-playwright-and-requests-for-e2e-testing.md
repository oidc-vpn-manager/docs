# 14. Playwright and Requests for E2E Testing

Date: 2025-09-02

## Status

Approved

## Context

The project requires a comprehensive end-to-end (E2E) testing strategy to validate full user workflows and service interactions. This necessitates tools for automating browser interactions for the web UI, and for making direct API calls to test service boundaries.

## Decision

We will use Playwright for testing all web interface interactions and the Python `requests` library for testing API endpoints during E2E tests. Tests will be written in Python using the `pytest` framework.

## Consequence

This standardizes the toolset for E2E testing, ensuring consistency in test development, execution, and maintenance. Playwright provides robust and reliable cross-browser automation capabilities. The `requests` library is a simple and effective way to interact with the service APIs. Developers writing E2E tests will need to be familiar with these specific tools.
