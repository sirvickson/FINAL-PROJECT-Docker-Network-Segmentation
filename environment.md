# Environment and Test Setup

## 1. Authorized Environment

This investigation was conducted in a learner-controlled Docker environment on Windows.

The testing scope is limited to the Docker containers, Docker network, and connectivity paths created for this project.

No production systems, unauthorized networks, real customer data, or external systems were tested.

---

## 2. Host Environment

- Operating system: Windows
- Docker Desktop: 4.83.0
- Docker Engine: 29.6.2
- Docker Compose: 5.3.1
- Docker context: desktop-linux

---

## 3. Docker Services

The project uses the following containers:

| Container | Image | Purpose |
|---|---|---|
| `medusa-postgres` | `postgres:15-alpine` | PostgreSQL database service |
| `medusa-redis` | `redis:7-alpine` | Redis service |
| `medusa-app-test` | `alpine:3.20` | Controlled application-side connectivity test client |

The `medusa-app-test` container is a controlled test client used to represent an application-side network participant.

It is not claimed to be the production Medusa application.

---

## 4. Docker Network

Network name:

```text
week1_medusa-net