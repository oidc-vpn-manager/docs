# 10. UTC and ISO 8601 for Timestamps

Date: 2025-09-02

## Status

Approved

## Context

In a distributed system with multiple services, servers, and users potentially located in different timezones, handling timestamps inconsistently can lead to data corruption, incorrect ordering of events, and difficult-to-diagnose bugs. A standard must be set for how time is recorded and transmitted.

## Decision

All services will use the Coordinated Universal Time (UTC) timezone for all internal timestamps and date-based calculations. All timestamps exposed via APIs or stored in logs and databases will be formatted according to the ISO 8601 standard (e.g., `YYYY-MM-DDTHH:MM:SSZ`).

## Consequence

This ensures that all timestamps are consistent, unambiguous, and comparable across the entire system, regardless of the location of the servers or users. It simplifies all date and time calculations and makes debugging easier. Any frontend clients are responsible for converting UTC timestamps to the user's local timezone for display purposes if required.
