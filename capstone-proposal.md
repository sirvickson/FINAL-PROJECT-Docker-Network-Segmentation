# Capstone Project Proposal

## Project Title

Docker Network Segmentation as a Control Against Unauthorized Host-to-Database Access

## 1. Problem Statement

This project investigates the risk of direct host-to-database network access in a controlled Docker environment. PostgreSQL and Redis are containerized services that may require communication with an application container but do not necessarily require direct network exposure to the Docker host.

The investigation will determine whether Docker network segmentation can reduce direct host-originated connectivity to PostgreSQL and Redis while preserving legitimate application-container connectivity to those services.

## 2. Project Scope

This project is limited to an authorized, learner-controlled Docker environment.

The investigation will examine:

- PostgreSQL running as a Docker container.
- Redis running as a Docker container.
- A controlled application test container used to represent legitimate application-side connectivity.
- Docker network configuration and network segmentation.
- Host-originated TCP connectivity to PostgreSQL and Redis.
- Application-container connectivity to PostgreSQL and Redis.
- Before-and-after connectivity results.
- Docker configuration, command output, logs, and screenshots as supporting evidence.

The primary control being investigated is the use of an internal Docker network together with removal of unnecessary host-published database ports.

The project will not include:

- Testing against production systems.
- Testing against systems or networks without authorization.
- Real customer or personal data.
- Real production credentials or secrets.
- Destructive actions.
- Denial-of-service testing.
- Exploitation of vulnerabilities unrelated to the research question.

## 3. Research Question

Does configuring PostgreSQL and Redis on an internal Docker network prevent direct host-to-database connectivity while preserving application-container connectivity to those services in the authorized Docker environment?

## 4. Hypothesis

If PostgreSQL and Redis are placed on an internal Docker network without host port exposure, then host-originated connectivity tests to their service ports will fail while application-container connectivity tests to PostgreSQL and Redis will succeed.

## 5. Alternative Explanation

A failed host connectivity test may have causes other than Docker network segmentation, including host firewall behavior, service configuration, routing behavior, or the database services not listening on the tested interface.

Therefore, test results will be interpreted together with Docker network configuration, port-publishing configuration, and application-container connectivity results.

## 6. Planned Comparison

The investigation will compare the environment before and after the network-segmentation configuration is applied.

### Baseline

The original Docker configuration will be tested to determine whether:

1. The host can connect to PostgreSQL.
2. The host can connect to Redis.
3. The application test container can connect to PostgreSQL.
4. The application test container can connect to Redis.

### Controlled Configuration

The Docker configuration will then be modified so that:

1. PostgreSQL is not exposed through a published host port.
2. Redis is not exposed through a published host port.
3. PostgreSQL and Redis remain connected to the Docker network.
4. The Docker network is configured as an internal network.
5. The application test container remains connected to the same network.

The same connectivity tests will then be repeated.

## 7. Expected Security Outcome

The expected outcome is that direct host-originated connectivity to PostgreSQL and Redis will be prevented while connectivity from the application test container to those services will remain available.

The investigation will not assume that the hypothesis is correct. The final conclusion will be based on observed evidence.

## 8. Evidence

Evidence will include, where applicable:

- Docker Compose configuration.
- Docker network configuration.
- Container status.
- Published-port information.
- Host connectivity test results.
- Application-container connectivity test results.
- Screenshots.
- Command output and logs.
- A test matrix documenting each test.
- An evidence index identifying each evidence item.

Sensitive information such as passwords, tokens, private keys, or other credentials will not be included in the evidence repository.

## 9. Safety and Authorization

All testing will be performed within the learner-controlled Docker environment. Testing will be limited to connectivity and configuration verification required to answer the research question.

No unauthorized systems, production environments, personal data, or destructive testing will be used.

## 10. Project Limitation

The investigation evaluates the tested Docker configuration and network paths. It will not establish that all possible methods of host-to-container access are prevented, nor will it establish the security of Docker as a platform generally.

Because the controlled configuration changes both host port exposure and the Docker network's internal setting, the experiment will evaluate the combined segmentation configuration rather than claiming that either individual setting alone caused the observed result.