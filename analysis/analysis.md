# Analysis

## 1. Purpose of the Analysis

The purpose of this analysis is to determine whether the segmented Docker configuration changed host-to-database connectivity while preserving application-side connectivity.

The analysis is based on the documented baseline observations, segmented connectivity tests, Docker Compose configuration, container status, and Docker network inspection.

## 2. Baseline Condition

Before segmentation, PostgreSQL and Redis were configured with host port publishing.

The baseline observations showed:

- Windows host -> PostgreSQL: TCP connection succeeded.
- Windows host -> Redis: TCP connection succeeded.
- medusa-app-test -> PostgreSQL: connection reported open.
- medusa-app-test -> Redis: connection reported open.

This established that both database services were reachable from the Windows host and from the application-side test container before the control was applied.

## 3. Segmented Condition

The controlled configuration removed the published host ports for PostgreSQL and Redis and changed the Docker network configuration to internal: true.
Docker configuration evidence confirmed that the database services no longer had published host port mappings.

## 4. Host Connectivity Results

After segmentation, Windows host tests against the loopback addresses produced:

PostgreSQL 127.0.0.1:5432

TcpTestSucceeded : False

Redis 127.0.0.1:6379

TcpTestSucceeded : False

This represents a change from the baseline condition, where both host connections succeeded.

The result indicates that the Windows host could no longer use the previously published PostgreSQL and Redis endpoints.

## 5. Application-Side Connectivity Results

After segmentation, the application test container successfully connected to both services.

PostgreSQL:

medusa-postgres (172.19.0.3:5432) open

Redis:

medusa-redis (172.19.0.2:6379) open

This demonstrates that removing host port publishing did not prevent an attached container from reaching the database services through the Docker network.

## 6. Direct Container-IP Results

Additional tests were performed from the Windows host toward the container IP addresses.

PostgreSQL:

172.19.0.3:5432

Redis:

172.19.0.2:6379

Both TCP tests failed.

The Windows host also reported a destination-host-unreachable condition for these tested paths.

These results provide additional evidence that the tested Windows host path could not directly reach the database container IP addresses after segmentation.

However, these results must be interpreted within the Docker Desktop networking architecture.

They do not prove that every possible host-to-container access mechanism is blocked.

## 7. Before-and-After Security Comparison

| Connectivity Path | Before Segmentation | After Segmentation |
|---|---|---|
| Windows host -> PostgreSQL | Allowed | Blocked |
| Windows host -> Redis | Allowed | Blocked |
| App-test -> PostgreSQL | Allowed | Allowed |
| App-test -> Redis | Allowed | Allowed |

The control therefore changed the exposure of the database services without removing their tested application-side network reachability.

## 8. Interpretation

The results support the hypothesis for the tested environment.

The segmented configuration reduced direct connectivity from the Windows host to PostgreSQL and Redis while preserving connectivity from the application-side container.

This is consistent with the security objective of reducing unnecessary network exposure to database services.

The strongest evidence for the host-access reduction is the combination of:

1. Removal of host `ports` mappings.
2. `internal: true` Docker network configuration.
3. Failed host connections to `127.0.0.1:5432` and `127.0.0.1:6379`.
4. Failed direct tests toward the tested container IP addresses.

The strongest evidence that legitimate application-side connectivity was preserved is the successful `nc` tests from `medusa-app-test` to both database services.

## 9. Observation vs. Interpretation

### Direct Observations

- PostgreSQL and Redis had published host ports in the baseline configuration.
- Host TCP connectivity succeeded during baseline testing.
- Host TCP connectivity failed after segmentation.
- PostgreSQL and Redis remained reachable from `medusa-app-test`.
- The segmented Docker network reported `Internal: true`.
- `docker port` reported no host mappings for PostgreSQL and Redis after segmentation.

### Interpretation

These observations indicate that the segmented configuration reduced the tested host-to-database connectivity paths while maintaining application-side connectivity.

### Inference

Reducing unnecessary host exposure can reduce the attack surface available to processes operating on the host or paths originating from the host.

This is a defense-in-depth benefit rather than a complete security boundary.

## 10. Alternative Explanations

A failed host connectivity test does not automatically prove that Docker network segmentation was the only cause.

Possible contributing factors include:

- Removal of host port publishing.
- Docker's internal network behavior.
- Docker Desktop networking architecture.
- Host firewall or routing behavior.
- Service listening configuration.
- Other platform-specific networking controls.

Because host port publishing and the `internal` network setting were changed together, the experiment cannot isolate the individual effect of each change.

## 11. Hypothesis Evaluation

The hypothesis predicted:

> If PostgreSQL and Redis are placed on an internal Docker network without host port exposure, then host-originated connectivity tests will fail while application-container connectivity tests will succeed.

The observed results matched both expected outcomes.

### Host Connectivity

- PostgreSQL: failed after segmentation.
- Redis: failed after segmentation.

**Result: Supported.**

### Application Connectivity

- PostgreSQL: remained open.
- Redis: remained open.

**Result: Supported.**

Therefore, the hypothesis is **supported for the tested Docker environment and configuration**.

## 12. Security Significance

The results demonstrate the principle of least exposure at the network layer.

Database services generally do not need to be directly exposed to the host when the application that consumes them can communicate through a dedicated container network.

Removing unnecessary host port publishing reduces the number of directly reachable endpoints.

The internal Docker network adds another layer of network segmentation around the services.

The control should therefore be viewed as a defense-in-depth measure that complements other security controls.

## 13. Risk Reduction

The baseline configuration allowed direct Windows host connectivity to PostgreSQL and Redis.

That creates an additional network path to services that are intended primarily for application use.

After segmentation, the tested host paths were no longer reachable while the application-side paths remained available.

The resulting risk reduction can be summarized as:

Baseline:

Windows Host
     |
     +---- PostgreSQL
     |
     +---- Redis

App-test
     |
     +---- PostgreSQL
     |
     +---- Redis


Segmented:

Windows Host
     |
     X---- PostgreSQL
     |
     X---- Redis

App-test
     |
     +---- PostgreSQL
     |
     +---- Redis

The segmented configuration therefore reduces unnecessary direct access while maintaining the required tested application communication paths.

## 14. Limitations

### Combined Changes

Host port publishing and the Docker network `internal` setting were changed simultaneously.

The experiment therefore demonstrates the behavior of the combined control configuration rather than attributing the result to one configuration setting.

### Historical Baseline Evidence

The original baseline results were observed while the baseline configuration was active, but permanent evidence files were not captured before the configuration was changed.

The baseline results are therefore recorded as historical observations rather than newly captured baseline log files.

### Docker Desktop Architecture

The direct container-IP tests were performed from Windows against containers running through Docker Desktop's Linux container environment.

The results represent the tested Windows-to-container network path and should not be generalized to every Docker deployment architecture.

### Application Test Client

The `medusa-app-test` container is a controlled connectivity test client and not the complete production application.

Successful connectivity therefore demonstrates network reachability but does not prove that a full application stack has been completely validated.

### Limited Test Surface

Only the documented PostgreSQL and Redis ports and connectivity paths were tested.

The investigation does not establish that every possible host-originated access method is blocked.

## 15. Overall Finding

The investigation found that the tested segmented Docker configuration successfully changed the observed network exposure:

- Host access to PostgreSQL changed from allowed to blocked.
- Host access to Redis changed from allowed to blocked.
- Application-side access to PostgreSQL remained allowed.
- Application-side access to Redis remained allowed.

The findings support the use of Docker network segmentation and removal of unnecessary host port exposure as a defense-in-depth control against direct host-to-database access.

The finding applies only to the tested configuration, environment, and connectivity paths.

