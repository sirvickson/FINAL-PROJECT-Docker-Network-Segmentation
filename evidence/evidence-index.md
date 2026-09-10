# Evidence Index

This index maps project evidence to the tests and events documented in the investigation.

All testing was conducted in the learner-controlled Docker environment.

## Evidence Records

| Evidence ID | File / Source | Test / Event | Date / Time | Description | Status |
|---|---|---|---|---|---|
| E-001 | Historical command output recorded during baseline testing | T-001 | 2026-09-09 | Windows host successfully connected to PostgreSQL on `127.0.0.1:5432` before segmentation | Historical observation |
| E-002 | Historical command output recorded during baseline testing | T-002 | 2026-09-09 | Windows host successfully connected to Redis on `127.0.0.1:6379` before segmentation | Historical observation |
| E-003 | Historical command output recorded during baseline testing | T-003 | 2026-09-09 | Application test container successfully connected to PostgreSQL before segmentation | Historical observation |
| E-004 | Historical command output recorded during baseline testing | T-004 | 2026-09-09 | Application test container successfully connected to Redis before segmentation | Historical observation |
| E-005 | `evidence/logs/T-005-host-postgres.txt` | T-005 | 2026-09-09 | Segmented Windows host connectivity test to PostgreSQL port 5432 | Valid controlled evidence |
| E-006 | `evidence/logs/T-006-host-redis.txt` | T-006 | 2026-09-09 | Segmented Windows host connectivity test to Redis port 6379 | Valid controlled evidence |
| E-007 | `evidence/logs/T-007-app-postgres.txt` | T-007 | 2026-09-09 | Segmented application test container connectivity to PostgreSQL | Valid controlled evidence |
| E-008 | `evidence/logs/T-008-app-redis.txt` | T-008 | 2026-09-09 | Segmented application test container connectivity to Redis | Valid controlled evidence |
| E-009 | `evidence/logs/T-009-host-postgres-ip.txt` | T-009 | 2026-09-09 | Segmented Windows host test toward PostgreSQL container IP | Valid controlled evidence |
| E-010 | `evidence/logs/T-010-host-redis-ip.txt` | T-010 | 2026-09-09 | Segmented Windows host test toward Redis container IP | Valid controlled evidence |
| E-011 | `evidence/logs/segmented-compose-config.txt` | Configuration validation | 2026-09-09 | Docker Compose configuration showing removal of published database ports and internal network configuration | Requires credential redaction before publication |
| E-012 | `evidence/logs/segmented-compose-ps.txt` | Environment validation | 2026-09-09 | Docker Compose service status after segmentation | Valid |
| E-013 | `evidence/logs/segmented-network-inspect.txt` | Network validation | 2026-09-09 | Docker network inspection showing internal network and attached containers | Valid |
| E-014 | `evidence/logs/segmented-postgres-ports.txt` | Port exposure validation | 2026-09-09 | No host port mapping reported for PostgreSQL after segmentation | Valid |
| E-015 | `evidence/logs/segmented-redis-ports.txt` | Port exposure validation | 2026-09-09 | No host port mapping reported for Redis after segmentation | Valid |

## Invalid Baseline Capture Artifacts

The following files were created after segmentation and therefore must **not** be used as baseline evidence:

| File | Reason |
|---|---|
| `evidence/logs/T-001-host-postgres-baseline.txt` | Captured after segmentation; reports `TcpTestSucceeded : False` |
| `evidence/logs/T-002-host-redis-baseline.txt` | Captured after segmentation; reports `TcpTestSucceeded : False` |
| `evidence/logs/T-003-app-postgres-baseline.txt` | Captured after segmentation; does not represent the original baseline condition |
| `evidence/logs/T-004-app-redis-baseline.txt` | Captured after segmentation; does not represent the original baseline condition |

These files are retained for auditability but are not cited as evidence of the baseline condition.

## Evidence Handling Notes

- Historical baseline observations are distinguished from permanent post-segmentation evidence.
- No current file is falsely presented as a baseline capture.
- The segmented-condition test files T-005 through T-010 are the primary permanent connectivity evidence.
- Configuration evidence must be reviewed and redacted before publication if credentials are present.
- Empty `docker port` output files are intentionally retained because the absence of output demonstrates that no host port mapping was reported.
- Evidence must not contain passwords, tokens, private keys, or other secrets.