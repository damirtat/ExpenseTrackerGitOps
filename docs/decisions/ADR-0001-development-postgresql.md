# ADR-0001: Use in-cluster PostgreSQL temporarily for development

- Status: Accepted
- Date: 2026-08-09

## Context

The Hetzner K3s cluster is available, but a managed PostgreSQL provider has not been
selected or provisioned. The project needs a real deployed development environment
before continuing feature work.

## Decision

Run the official PostgreSQL image as a single-replica Kubernetes StatefulSet with a
persistent volume in the development namespace. Supply its credentials through
Doppler-synced Kubernetes Secrets.

## Consequences

- The development environment can run end-to-end now.
- A node or local-volume failure can make the development database unavailable or
  lose data. That is acceptable only for development.
- Production will use managed PostgreSQL later. The API continues to receive its
  connection string from Doppler, minimizing application-level changes at migration.
- No production deployment is permitted to use this StatefulSet.
