# Analysis

## 1. Purpose of the Analysis

The purpose of this analysis is to determine whether the segmented Docker configuration changed host-to-database connectivity while preserving application-side connectivity.

The analysis is based on the documented baseline observations, segmented connectivity tests, Docker Compose configuration, container status, and Docker network inspection.

---

## 2. Baseline Condition

Before segmentation, PostgreSQL and Redis were configured with host port publishing.

The baseline observations showed:

- Windows host → PostgreSQL: TCP connection succeeded.
- Windows host → Redis: TCP connection succeeded.
- `medusa-app-test` → PostgreSQL: connection reported `open`.
- `medusa-app-test` → Redis: connection reported `open`.

This established that both database services were reachable from the Windows host and from the application-side test container before the control was applied.

---

## 3. Segmented Condition

The controlled configuration removed the published host ports for PostgreSQL and Redis and changed the Docker network configuration to:

```yaml
internal: true