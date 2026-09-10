# Docker Network Segmentation as a Control Against Unauthorized Host-to-Database Access

## Project Overview

This project investigates whether Docker network segmentation can reduce unauthorized host-to-database connectivity while preserving legitimate application-container access to database services.

The investigation uses a controlled, learner-authorized Docker environment containing:

- PostgreSQL
- Redis
- An Alpine-based application test container
- A Docker bridge network

The experiment compares connectivity before and after segmentation. The baseline configuration publishes PostgreSQL and Redis ports to the Windows host. The segmented configuration removes host port publishing and configures the Docker network as internal.

The project uses connectivity tests, Docker configuration output, network inspection, command output, and evidence logs to evaluate the security control.

---

## Research Question

> Does configuring PostgreSQL and Redis on an internal Docker network prevent direct host-to-database connectivity while preserving application-container connectivity to those services in the authorized Docker environment?

---

## Hypothesis

If PostgreSQL and Redis are placed on an internal Docker network without host port exposure, then host-originated connectivity tests to their service ports will fail while application-container connectivity tests to PostgreSQL and Redis will succeed.

The expected security benefit is that database services remain reachable by the application container while direct connectivity from the Windows host is restricted.

---

## Scope

This project is limited to the authorized Docker environment used for the investigation.

The investigation includes:

- Docker Compose configuration
- PostgreSQL connectivity
- Redis connectivity
- Application-container connectivity
- Windows host connectivity
- Docker network configuration
- Docker container and network inspection
- Connectivity test results
- Logs and screenshots used as evidence

The investigation does not include:

- Production systems
- Unauthorized networks
- Real customer or personal data
- Real production credentials
- Destructive testing
- Exploitation of external systems
- Attempts to bypass security controls outside the authorized environment

---

## Environment

### Host Environment

- Operating system: Windows
- Docker Desktop
- Docker Engine
- Docker Compose
- Docker Desktop Linux container environment

### Containers

| Container | Image | Purpose |
|---|---|---|
| `medusa-postgres` | `postgres:15-alpine` | PostgreSQL database service |
| `medusa-redis` | `redis:7-alpine` | Redis service |
| `medusa-app-test` | `alpine:3.20` | Controlled application-side connectivity test client |

The `medusa-app-test` container is a controlled test client. It represents an application-side network participant and is not claimed to be the production Medusa application.

### Docker Network

Network:

```text
week1_medusa-net
```

Network configuration:

- Baseline: Docker bridge network with host port publishing.
- Segmented: Docker bridge network configured as `internal: true`.
- PostgreSQL: TCP 5432.
- Redis: TCP 6379.

---

## Security Control

The implemented control combines two configuration changes:

1. PostgreSQL and Redis no longer publish ports to the Windows host.
2. The Docker bridge network is configured as an internal network.

This configuration is intended to prevent direct host-originated access to the database service ports while preserving connectivity between containers attached to the network.

Because both changes were applied together, the experiment evaluates the segmented configuration as a combined control rather than attributing the result to `internal: true` alone.

---

## Test Plan

| Test ID | Connectivity Path | Expected Result |
|---|---|---|
| T-001 | Windows host -> PostgreSQL | Allowed before segmentation |
| T-002 | Windows host -> Redis | Allowed before segmentation |
| T-003 | `medusa-app-test` -> PostgreSQL | Allowed |
| T-004 | `medusa-app-test` -> Redis | Allowed |
| T-005 | Windows host -> PostgreSQL | Blocked after segmentation |
| T-006 | Windows host -> Redis | Blocked after segmentation |
| T-007 | `medusa-app-test` -> PostgreSQL | Allowed after segmentation |
| T-008 | `medusa-app-test` -> Redis | Allowed after segmentation |
| T-009 | Windows host -> PostgreSQL container IP | Blocked in tested host path |
| T-010 | Windows host -> Redis container IP | Blocked in tested host path |

---

## Evidence

Evidence is stored in the `evidence/` directory.

- `evidence-index.md` maps evidence IDs to files and tests.
- `tests/test-matrix.md` contains the complete test matrix.
- `evidence/logs/` contains command and connectivity output.
- `evidence/` contains supporting screenshots.

No passwords, tokens, private keys, or real customer data should be included in the repository.

---

## Repository Structure

```text
week1/
|-- README.md
|-- capstone-proposal.md
|-- docker-compose.yml
|-- docker-compose.baseline.yml
|-- environment.md
|-- analysis/
|   `-- analysis.md
|-- diagrams/
|   |-- Baseline_architecture.png
|   `-- Segmented_architecture.png
|-- evidence/
|   |-- evidence-index.md
|   |-- logs/
|   |-- screenshots/
|   `-- redacted-data/
`-- tests/
    `-- test-matrix.md


