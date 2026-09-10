#Docker Network Segmentation as a Control Against Unauthorized Host-to-Database Access

##Abstract

This project investigates whether Docker network segmentation can reduce unauthorized host-to-database connectivity while preserving legitimate application-container access to PostgreSQL and Redis. The investigation was conducted in a learner-controlled Docker Desktop environment using a PostgreSQL container, a Redis container, and a controlled Alpine Linux application-test container.

A baseline configuration exposed PostgreSQL and Redis through published host ports. Host-side TCP tests successfully reached both database services, while the application-test container also successfully reached both services over the Docker bridge network. The controlled configuration removed host port publishing and changed the Docker network to an internal network. After the change, host-side TCP tests to PostgreSQL and Redis failed, including tests directed at the containers' Docker IP addresses, while application-container connectivity to both services remained successful.

The results support the hypothesis for the tested configuration: Docker network segmentation combined with removal of host port exposure prevented the tested Windows host from directly reaching PostgreSQL and Redis while preserving container-to-container connectivity. The experiment does not establish that internal: true alone caused the result because host port exposure and the network setting were changed together. Additional limitations include the use of a controlled application-test container rather than a production application and the limited connectivity test surface.

Executive Summary

The security problem examined in this project is unauthorized direct access from a Docker host to database services that should primarily be accessed by an application container.

The project tested two configurations:

A baseline configuration in which PostgreSQL and Redis were attached to a standard Docker bridge network and their service ports were published to the host.

A segmented configuration in which the host port mappings were removed and the Docker network was configured with internal: true.

The baseline demonstrated that the Windows host could connect to PostgreSQL on TCP port 5432 and Redis on TCP port 6379. The application-test container could also connect to both services.

After segmentation, the host could no longer establish TCP connections to PostgreSQL or Redis through 127.0.0.1. Direct tests against the containers' Docker IP addresses also failed. At the same time, the application-test container continued to connect successfully to PostgreSQL and Redis using Docker service names.

The primary security finding is that the tested segmented configuration reduced the host's direct network reachability to the database services while maintaining the required application-to-database communication path.

The result should be interpreted as evidence for the specific configuration tested rather than proof of complete isolation. In particular, the experiment changed two related controls at the same time: host port exposure was removed and the Docker network was marked internal. Therefore, the experiment cannot independently attribute the observed result to either control by itself.

1. Introduction

1.1 Project Background

Containerized applications commonly separate application services from supporting services such as databases and caches. Network configuration is an important security boundary because it determines which systems can communicate with those services.

PostgreSQL and Redis are normally intended to serve application components rather than provide unrestricted direct access from the host or other external systems. Publishing database ports to the host can create an unnecessary network path that increases the reachable attack surface.

This project examines whether Docker network segmentation can remove that unnecessary path while preserving the connectivity required by an application container.

1.2 Security Problem

The security problem is unauthorized host-to-database access.

In the baseline configuration, PostgreSQL and Redis were reachable through published host ports:

PostgreSQL: TCP 5432

Redis: TCP 6379

This created a direct connectivity path from the Windows host to services that were intended to be accessed by containers on the Docker network.

The investigation therefore focuses on whether the database services can remain available to an application container without remaining directly reachable from the host.

1.3 Project Objective

The objective is to determine whether the tested Docker network segmentation configuration:

prevents direct host-originated connectivity to PostgreSQL;

prevents direct host-originated connectivity to Redis;

preserves application-container connectivity to PostgreSQL; and

preserves application-container connectivity to Redis.

The project uses before-and-after testing to compare the baseline and segmented configurations.

1.4 Research Question

Does configuring PostgreSQL and Redis on an internal Docker network prevent direct host-to-database connectivity while preserving application-container connectivity to those services in the authorized Docker environment?

1.5 Hypothesis

If PostgreSQL and Redis are placed on an internal Docker network without host port exposure, then host-originated connectivity tests to their service ports will fail while application-container connectivity to PostgreSQL and Redis will succeed, because the internal network restricts external connectivity while permitting communication between attached containers.

1.6 Alternative Explanation

A failed host connectivity test could have causes other than Docker network segmentation. Possible alternative explanations include:

host firewall behavior;

database service configuration;

routing behavior;

the service not listening on the tested interface; or

other Docker Desktop networking behavior.

For this reason, the investigation includes configuration inspection, published-port inspection, host-side tests, application-container tests, and direct container-IP tests rather than relying on a single connectivity result.

1.7 Scope

This project investigates an authorized, learner-controlled Docker environment using Docker configuration, container connectivity tests, host-side connectivity tests, application-to-database connectivity tests, logs, command output, and screenshots as evidence.

The project examines whether Docker network segmentation prevents direct host access to PostgreSQL and Redis while preserving legitimate application-container access to those services.

The project does not include:

testing against production systems;

testing against unauthorized networks;

real personal data;

unauthorized credentials;

destructive actions; or

attacks against third-party infrastructure.

2. Background and Technical Context

2.1 Docker Networking

Docker networking allows containers to communicate with one another through virtual networks. Containers attached to the same Docker network can generally communicate using container or service names and the ports on which their services listen.

The baseline environment used a Docker bridge network named week1_medusa-net. The network used the 172.19.0.0/16 subnet and had an Internal value of false.

The PostgreSQL, Redis, and application-test containers were attached to this network.

2.2 PostgreSQL

PostgreSQL is the relational database service used in the project. It listens on TCP port 5432 inside the container.

The database was configured with a PostgreSQL database named medusa-store. The project did not use real personal or production data.

2.3 Redis

Redis is the in-memory data store used in the project. It listens on TCP port 6379 inside the container.

Redis was included because the project evaluates whether network segmentation can protect multiple supporting data services rather than only a single database service.

2.4 Application-Test Container

The original project environment did not contain an active application container that could be used for controlled connectivity testing.

A minimal Alpine Linux container named medusa-app-test was therefore added as a controlled application-side test client.

The container was attached to the same Docker network as PostgreSQL and Redis and was kept running with:

sleep infinity

This container is not represented as the production Medusa application. It is a controlled test client used to verify that legitimate container-to-container communication remains possible after segmentation.

2.5 Security Control

The tested segmentation configuration used two related changes:

PostgreSQL and Redis no longer published ports to the Docker host.

The shared Docker network was configured with:

networks:
medusa-net:
driver: bridge
internal: true

The combination was intended to remove unnecessary host-to-database network paths while retaining communication between containers attached to the internal network.

3. Threat and Control Context

3.1 Threat Scenario

The threat scenario considered by this project is a host-originated connection to a database service that does not need to be directly exposed to the host.

In the baseline configuration, the following paths existed:

Windows host
|
+---- TCP 5432 ----> PostgreSQL
|
+---- TCP 6379 ----> Redis

Docker network: week1_medusa-net
|
+---- app-test ----> PostgreSQL
|
+---- app-test ----> Redis

The host-to-database paths existed because the database ports were published from the containers to the host.

The desired segmented architecture removes those published host paths while preserving application-container connectivity:

Windows host
|
X---- TCP 5432 ----> PostgreSQL
|
X---- TCP 6379 ----> Redis

Docker internal network: week1_medusa-net
|
+---- app-test ----> PostgreSQL
|
+---- app-test ----> Redis

The X symbols represent failed host-originated TCP tests in the segmented configuration.

3.2 Threat and Attack Surface

A published database port creates an additional reachable interface between the host and a service running inside a container. If the service is not intended to be accessed directly by the host, that path represents unnecessary exposure.

The relevant attack surface in this project is therefore not the PostgreSQL or Redis application itself. The focus is the network path that makes those services reachable from the host.

The project evaluates exposure at the connectivity layer rather than attempting credential attacks, exploitation, brute force, or destructive database actions.

3.3 Security Control Objective

The control objective is least-privilege network reachability.

The application-test container should be able to reach PostgreSQL and Redis because that represents the legitimate service dependency being tested. The Windows host should not have a direct TCP path to those services.

This is a segmentation objective: permit the required internal communication while removing an unnecessary external or host-originated path.

3.4 Expected Security Effect

The expected effect of the control is:

Connectivity Path

Baseline

Segmented Configuration

Windows host -> PostgreSQL:5432

Allowed

Blocked

Windows host -> Redis:6379

Allowed

Blocked

App-test -> PostgreSQL:5432

Allowed

Allowed

App-test -> Redis:6379

Allowed

Allowed

Windows host -> PostgreSQL container IP:5432

Reachable path tested

Blocked

Windows host -> Redis container IP:6379

Reachable path tested

Blocked

This expected result directly maps to the research question and hypothesis.

4. Methodology, Ethics, and Safety

4.1 Investigation Design

The investigation used a controlled before-and-after design.

The baseline configuration was first observed with PostgreSQL and Redis exposed through published host ports. Connectivity was tested from both the Windows host and the application-test container.

The configuration was then changed by removing the published host ports and setting the Docker network to internal: true.

The containers were recreated and the same relevant connectivity paths were tested again.

The investigation therefore compares:

baseline network exposure; and

segmented network exposure.

4.2 Authorized Environment

Testing was limited to the learner-controlled Docker Desktop environment used for the project.

The environment was run locally on Windows using Docker Desktop. No production systems, third-party hosts, or unauthorized networks were targeted.

4.3 Environment

The principal environment components were:

Component

Configuration / Role

Host

Windows workstation

Container platform

Docker Desktop

Docker context

desktop-linux

PostgreSQL

postgres:15-alpine

Redis

redis:7-alpine

Test client

alpine:3.20

PostgreSQL container

medusa-postgres

Redis container

medusa-redis

Test client container

medusa-app-test

Docker network

week1_medusa-net

Network driver

bridge

Baseline network internal setting

false

Segmented network internal setting

true

Network subnet

172.19.0.0/16

Network gateway

172.19.0.1

PostgreSQL service port

5432

Redis service port

6379

The Docker environment reported Docker Desktop 4.83.0, Docker Engine 29.6.2, and Docker Compose 5.3.1 during the investigation.

4.4 Variables

The primary outcome variables were TCP connectivity results.

Independent configuration changes:

publication of PostgreSQL host port 5432;

publication of Redis host port 6379; and

Docker network internal setting.

Observed dependent variables:

host-to-PostgreSQL TCP connectivity;

host-to-Redis TCP connectivity;

application-container-to-PostgreSQL connectivity;

application-container-to-Redis connectivity; and

host-to-container-IP TCP connectivity.

4.5 Tools

The investigation used:

Docker Compose;

Docker CLI;

Windows PowerShell Test-NetConnection;

Alpine nc for TCP connectivity testing;

Docker network inspection;

Docker port inspection;

command output logs; and

screenshots.

The tests were designed to verify connectivity rather than exploit services.

4.6 Evidence Handling

Evidence was stored under the project evidence directory.

Evidence included:

configuration output;

container status;

network inspection;

published-port inspection;

host-side connectivity results;

application-container connectivity results; and

screenshots.

Credentials and other sensitive values were not intentionally included in evidence. Any configuration evidence containing a credential field should be redacted before being committed to a public repository.

The baseline backup docker-compose.baseline.yml was retained so that the original configuration could be reconstructed without relying solely on memory.

4.7 Safety Boundaries

The investigation did not use:

destructive database commands;

credential guessing;

exploitation;

denial-of-service activity;

scans against unrelated systems;

production data; or

unauthorized infrastructure.

The test procedure was stopped if the environment behaved unexpectedly or if testing would have affected systems outside the defined scope.

5. Technical Implementation

5.1 Baseline Configuration

The baseline Compose configuration used a bridge network and published the database service ports to the host.

The relevant exposure was:

postgres:
ports:
- "5432:5432"

redis:
ports:
- "6379:6379"

networks:
medusa-net:
driver: bridge

Under this configuration, Docker published PostgreSQL and Redis ports to the Windows host.

The baseline docker compose ps output showed:

PostgreSQL: 0.0.0.0:5432->5432/tcp

Redis: 0.0.0.0:6379->6379/tcp

The same baseline was also reflected by docker port.

5.2 Baseline Network

The baseline Docker network was:

Name:     week1_medusa-net
Driver:   bridge
Subnet:   172.19.0.0/16
Gateway:  172.19.0.1
Internal: false

The observed container addresses were:

medusa-redis: 172.19.0.2

medusa-postgres: 172.19.0.3

medusa-app-test: 172.19.0.4

5.3 Baseline Connectivity

The baseline host tests produced successful TCP connections:

Host -> 127.0.0.1:5432
TcpTestSucceeded : True

and:

Host -> 127.0.0.1:6379
TcpTestSucceeded : True

The application-test container also successfully reached the database services:

medusa-postgres (172.19.0.3:5432) open

and:

medusa-redis (172.19.0.2:6379) open

These results established the baseline condition before segmentation.

5.4 Segmentation Configuration

The controlled configuration removed the published ports: mappings from PostgreSQL and Redis.

The Docker network was changed to:

networks:
medusa-net:
driver: bridge
internal: true

The resulting services retained their internal service ports, but those ports were no longer published to the Windows host.

This distinction is important. The services still listen on TCP 5432 and TCP 6379 inside their containers. What changed was the host-facing exposure.

5.5 Configuration Validation

Before recreating the environment, the Compose configuration was validated with:

docker compose config

The resulting configuration showed:

no ports: section for PostgreSQL;

no ports: section for Redis; and

internal: true on medusa-net.

After validation, the environment was recreated using:

docker compose down
docker compose up -d
docker compose ps

The resulting service status showed:

medusa-app-test running;

medusa-postgres healthy; and

medusa-redis running.

5.6 Segmented Network

The recreated network remained:

Name:     week1_medusa-net
Driver:   bridge
Subnet:   172.19.0.0/16
Gateway:  172.19.0.1
Internal: true

The containers retained the observed addresses:

medusa-redis: 172.19.0.2

medusa-postgres: 172.19.0.3

medusa-app-test: 172.19.0.4

The unchanged service IPs and network range helped maintain a straightforward before-and-after comparison, although Docker networking should not be assumed to preserve these values in every recreation.

5.7 Published-Port Validation

The following commands were used:

docker port medusa-postgres
docker port medusa-redis

Both commands produced no port mappings in the segmented configuration.

The empty outputs were retained as evidence because the absence of a published port is itself relevant to the control being tested.

6. Test Procedure and Results

6.1 Test Matrix

Test ID

Condition

Expected Result

Observed Result

Evidence ID

Interpretation

T-001

Baseline host -> PostgreSQL:5432

Connection succeeds

TCP succeeded

E-001

Host had direct access

T-002

Baseline host -> Redis:6379

Connection succeeds

TCP succeeded

E-002

Host had direct access

T-003

Baseline app-test -> PostgreSQL:5432

Connection succeeds

Port open

E-003

Internal application path worked

T-004

Baseline app-test -> Redis:6379

Connection succeeds

Port open

E-004

Internal application path worked

T-005

Segmented host -> PostgreSQL:5432

Connection fails

TCP failed

E-008

Host access removed

T-006

Segmented host -> Redis:6379

Connection fails

TCP failed

E-009

Host access removed

T-007

Segmented app-test -> PostgreSQL:5432

Connection succeeds

Port open

E-010

Legitimate container path preserved

T-008

Segmented app-test -> Redis:6379

Connection succeeds

Port open

E-011

Legitimate container path preserved

T-009

Segmented host -> PostgreSQL container IP:5432

Connection fails

TCP failed

E-012

Direct container-IP path also failed

T-010

Segmented host -> Redis container IP:6379

Connection fails

TCP failed

E-013

Direct container-IP path also failed

The baseline test results in T-001 through T-004 are historical baseline observations captured during the earlier baseline phase. Later files with baseline in their filenames were captured after segmentation and are not treated as historical baseline evidence.

6.2 Test T-001: Baseline Host to PostgreSQL

The baseline Windows host test used:

powershell -Command "Test-NetConnection -ComputerName 127.0.0.1 -Port 5432"

Observed result:

TcpTestSucceeded : True

This established that PostgreSQL was reachable through the host's loopback address before segmentation.

6.3 Test T-002: Baseline Host to Redis

The baseline Windows host test used:

powershell -Command "Test-NetConnection -ComputerName 127.0.0.1 -Port 6379"

Observed result:

TcpTestSucceeded : True

This established that Redis was reachable through the host's loopback address before segmentation.

6.4 Test T-003: Baseline Application-Test to PostgreSQL

The application-side test used:

docker exec medusa-app-test sh -c "nc -zv medusa-postgres 5432"

Observed result:

medusa-postgres (172.19.0.3:5432) open

This confirmed that the application-test container could reach PostgreSQL using the Docker service/container name.

6.5 Test T-004: Baseline Application-Test to Redis

The application-side test used:

docker exec medusa-app-test sh -c "nc -zv medusa-redis 6379"

Observed result:

medusa-redis (172.19.0.2:6379) open

This confirmed that the application-test container could reach Redis before segmentation.

6.6 Test T-005: Segmented Host to PostgreSQL

The segmented Windows host test used:

powershell -Command "Test-NetConnection -ComputerName 127.0.0.1 -Port 5432"

Observed result:

TcpTestSucceeded : False

The failed result demonstrates that the previously available loopback path to PostgreSQL was no longer available after the configuration change.

6.7 Test T-006: Segmented Host to Redis

The segmented Windows host test used:

powershell -Command "Test-NetConnection -ComputerName 127.0.0.1 -Port 6379"

Observed result:

TcpTestSucceeded : False

The failed result demonstrates that the previously available loopback path to Redis was no longer available after the configuration change.

6.8 Test T-007: Segmented Application-Test to PostgreSQL

The segmented application-side test used:

docker exec medusa-app-test sh -c "nc -zv medusa-postgres 5432"

Observed result:

medusa-postgres (172.19.0.3:5432) open

This demonstrates that PostgreSQL remained reachable from the application-side container after segmentation.

6.9 Test T-008: Segmented Application-Test to Redis

The segmented application-side test used:

docker exec medusa-app-test sh -c "nc -zv medusa-redis 6379"

Observed result:

medusa-redis (172.19.0.2:6379) open

This demonstrates that Redis remained reachable from the application-side container after segmentation.

6.10 Test T-009: Segmented Host to PostgreSQL Container IP

The host test used the observed Docker IP directly:

powershell -Command "Test-NetConnection -ComputerName 172.19.0.3 -Port 5432"

Observed result:

TcpTestSucceeded : False

The test also reported that the ping attempt failed with a destination-host-unreachable result.

This result provides additional evidence that the Windows host did not have a working TCP path to the PostgreSQL container through the tested Docker IP.

6.11 Test T-010: Segmented Host to Redis Container IP

The host test used:

powershell -Command "Test-NetConnection -ComputerName 172.19.0.2 -Port 6379"

Observed result:

TcpTestSucceeded : False

The test also reported a destination-host-unreachable result for the ping portion.

This provides additional evidence that the Windows host did not have a working TCP path to the Redis container through the tested Docker IP.

7. Evidence and Findings

7.1 Evidence Inventory

The primary evidence set is organized under evidence/logs.

Evidence ID

File / Source

Test or Event

Description

Status

E-001

Historical baseline observation

T-001

Baseline host -> PostgreSQL succeeded

Historical observation

E-002

Historical baseline observation

T-002

Baseline host -> Redis succeeded

Historical observation

E-003

Historical baseline observation

T-003

Baseline app-test -> PostgreSQL succeeded

Historical observation

E-004

Historical baseline observation

T-004

Baseline app-test -> Redis succeeded

Historical observation

E-005

segmented-compose-config.txt

Configuration validation

Segmented Compose configuration

Valid controlled evidence

E-006

segmented-compose-ps.txt

Configuration deployment

Segmented container status

Valid controlled evidence

E-007

segmented-network-inspect.txt

Network validation

Internal network, subnet, gateway, container membership

Valid controlled evidence

E-008

T-005-host-postgres.txt

T-005

Segmented host -> PostgreSQL failed

Valid controlled evidence

E-009

T-006-host-redis.txt

T-006

Segmented host -> Redis failed

Valid controlled evidence

E-010

T-007-app-postgres.txt

T-007

Segmented app-test -> PostgreSQL succeeded

Valid controlled evidence

E-011

T-008-app-redis.txt

T-008

Segmented app-test -> Redis succeeded

Valid controlled evidence

E-012

T-009-host-postgres-ip.txt

T-009

Segmented host -> PostgreSQL container IP failed

Valid controlled evidence

E-013

T-010-host-redis-ip.txt

T-010

Segmented host -> Redis container IP failed

Valid controlled evidence

E-014

segmented-postgres-ports.txt

Port validation

No published PostgreSQL host port

Valid controlled evidence

E-015

segmented-redis-ports.txt

Port validation

No published Redis host port

Valid controlled evidence

Screenshots provide additional visual evidence for the segmented test results.

7.2 Evidence Quality

The strongest evidence is the combination of configuration state and observed connectivity.

Configuration evidence shows that:

the network was set to internal: true;

PostgreSQL had no published host port; and

Redis had no published host port.

Connectivity evidence independently shows that:

host-to-PostgreSQL failed;

host-to-Redis failed;

application-test-to-PostgreSQL succeeded; and

application-test-to-Redis succeeded.

This combination is stronger than relying on configuration alone because it demonstrates the observed security behavior.

7.3 Baseline Evidence Limitation

A methodological issue occurred during evidence collection.

The original baseline connectivity results were observed during the baseline testing phase, but some files later named as baseline test captures were created after the segmentation change. Those later files contain failed host connections and therefore do not represent the historical baseline condition.

Those files are retained for auditability but are explicitly excluded from the baseline evidence set.

The report therefore relies on the documented historical baseline observations for T-001 through T-004 rather than treating the later false-result files as proof of the baseline state.

This limitation reduces the strength of the evidence archive for the baseline phase, but it does not change the observed controlled-phase results.

8. Analysis

8.1 Baseline Analysis

The baseline established the security condition that the project intended to change.

Both database services were reachable from the Windows host:

PostgreSQL through TCP 5432;

Redis through TCP 6379.

At the same time, both services were reachable from the application-test container.

Therefore, the baseline provided both required comparison paths:

a host-originated path that represented unnecessary exposure; and

an application-container path that represented legitimate service communication.

8.2 Segmented Configuration Analysis

After segmentation, the host-originated loopback tests failed for both database services.

The direct container-IP tests also failed.

Meanwhile, the application-test container continued to reach both services using Docker names.

This indicates that the configuration did not simply make PostgreSQL and Redis unavailable. Instead, it changed which network origin could reach them.

That distinction is central to the security objective.

8.3 Hypothesis Evaluation

The hypothesis predicted:

host -> PostgreSQL would fail;

host -> Redis would fail;

app-test -> PostgreSQL would succeed; and

app-test -> Redis would succeed.

All four predicted outcomes were observed.

Therefore, the hypothesis is supported for the tested configuration.

The correct conclusion is not that Docker networking always prevents host access. The supported conclusion is narrower:

In the tested Docker Desktop environment, the segmented configuration consisting of an internal Docker network and removal of published database ports prevented the tested Windows host from establishing TCP connections to PostgreSQL and Redis while preserving application-container connectivity.

8.4 Alternative Explanation Analysis

The alternative explanations were considered using multiple forms of evidence.

A host firewall alone would not explain why the application-test container could continue to connect to both services while the host failed. It is still possible that host-side filtering contributed to the result, but the configuration evidence shows that the published host ports were also removed.

A database listener problem is unlikely to explain the entire result because PostgreSQL and Redis continued to accept connections from the application-test container.

A service failure is similarly inconsistent with the successful application-side tests.

Routing and Docker Desktop networking remain relevant limitations because the direct container-IP tests failed with routing-related information, and Docker Desktop networking on Windows involves a virtualization layer.

The evidence therefore supports the segmented configuration as the principal explanation, while not eliminating every possible underlying networking mechanism.

8.5 Causal Attribution Limitation

The experiment changed two controls together:

removal of the ports: mappings; and

setting the network to internal: true.

Because both changed at the same time, the experiment cannot isolate which control was necessary or sufficient for each observed result.

For example, removal of published ports alone may have been sufficient to eliminate the 127.0.0.1 host paths. Conversely, the internal network setting may contribute to preventing direct access to the container network.

The project therefore evaluates the combined segmentation configuration rather than making a causal claim about internal: true alone.

8.6 Security Interpretation

The result demonstrates a practical least-privilege principle:

A service should not be exposed through a network path that is not required by its intended consumers.

In the baseline, PostgreSQL and Redis had host-published ports even though the application-test container could communicate with them through the Docker network.

In the segmented configuration, the required application-side communication remained functional while the tested host-originated paths were removed.

This is a useful reduction in network attack surface.

8.7 What the Test Does Not Prove

The experiment does not prove:

complete Docker host isolation;

that every host-originated access method is blocked;

that PostgreSQL or Redis are immune to exploitation;

that credentials are secure;

that application authorization is correctly configured;

that the configuration is appropriate for every Docker Desktop or Linux environment;

that internal: true alone caused the result; or

that the environment is equivalent to a production deployment.

The findings are limited to the tested connectivity paths and configuration.

9. Risk and Control Assessment

9.1 Baseline Risk

The baseline configuration exposed PostgreSQL and Redis to the host through published ports.

This created an unnecessary network path.

The principal risk was increased reachability: a process able to initiate connections from the host could attempt to communicate with database services without first going through the intended application-layer path.

The project did not attempt exploitation, so it does not assign a measured probability of compromise. The finding is therefore a network-exposure finding rather than an exploitability measurement.

9.2 Segmented Risk

The segmented configuration reduced the tested host-originated reachability.

The application-test container retained access, which means the control did not simply isolate all services from one another.

This represents a more restrictive network architecture aligned with least privilege.

9.3 Control Effectiveness

Control Objective

Result

Assessment

Remove published PostgreSQL host port

No published port observed

Effective in tested configuration

Remove published Redis host port

No published port observed

Effective in tested configuration

Prevent host -> PostgreSQL

TCP test failed

Effective for tested path

Prevent host -> Redis

TCP test failed

Effective for tested path

Preserve app-test -> PostgreSQL

TCP port open

Effective

Preserve app-test -> Redis

TCP port open

Effective

Demonstrate complete host isolation

Not tested exhaustively

Not established

9.4 Residual Risk

Even with network segmentation, residual risks remain.

These include:

compromised application containers;

compromised database credentials;

vulnerable PostgreSQL or Redis versions;

misconfigured additional Docker networks;

accidental future port publication;

Docker daemon or host compromise;

application-layer authorization failures; and

configuration drift.

Network segmentation should therefore be treated as one layer of defense rather than a complete security solution.

10. Recommendations

10.1 Keep Database Ports Unpublished

PostgreSQL and Redis should remain without host ports: mappings when direct host access is not a legitimate requirement.

Internal service ports can remain available to containers on the appropriate Docker network.

10.2 Use Dedicated Internal Networks

Application and data services should be attached only to the networks required for their communication.

An internal network can be used when services should communicate with one another without providing unnecessary external connectivity.

10.3 Apply Least Privilege to Network Membership

Containers should not automatically be attached to every network.

A more restrictive architecture would place only the application and required data services on the network paths they actually need.

10.4 Validate Published Ports During Deployment

Deployment checks should verify that sensitive service ports have not accidentally been published.

Useful checks include:

docker compose ps
docker port medusa-postgres
docker port medusa-redis
docker network inspect week1_medusa-net

These checks can be incorporated into a repeatable security validation procedure.

10.5 Protect Credentials

Database credentials should be managed separately from source-controlled configuration whenever possible.

Credentials should not be stored in public repositories, screenshots, evidence files, or reports.

The project evidence should use redaction wherever a configuration command could reveal a password or other secret.

10.6 Add Application-Level Controls

Network segmentation should be combined with:

strong database authentication;

least-privilege database accounts;

Redis authentication where appropriate;

secure secret management;

service-specific authorization; and

timely patching.

Network reachability controls cannot replace authentication and authorization.

10.7 Test Configuration Drift

A future improvement would automatically check for unexpected published ports and unauthorized network attachments as part of deployment or CI/CD validation.

This would help detect regression from the segmented state.

11. Limitations and Future Work

11.1 Application-Test Container Limitation

The project used medusa-app-test as a controlled Alpine test client because no active application container was available in the original environment.

The test therefore proves container-to-database connectivity rather than demonstrating a complete production application workflow.

A future experiment should use the actual application container and verify application-level database operations.

11.2 Combined Variable Limitation

The experiment changed both host port exposure and the network internal setting together.

A stronger experimental design would use additional configurations:

baseline: published ports + non-internal network;

remove published ports only;

internal network only where technically appropriate;

both controls together.

This would allow the individual contribution of each control to be evaluated.

11.3 Platform Limitation

The test was performed using Docker Desktop on Windows.

Docker Desktop networking includes virtualization and platform-specific behavior. Results should therefore not automatically be generalized to every native Linux Docker deployment.

A future comparison could repeat the experiment on a Linux Docker host.

11.4 Connectivity Test Limitation

The test focused on TCP connectivity to PostgreSQL and Redis.

It did not evaluate:

UDP;

application-layer authentication;

database authorization;

TLS;

IPv6-specific paths;

alternate host networking modes;

privileged containers; or

Docker daemon compromise.

The conclusion should therefore remain limited to the tested TCP paths.

11.5 Baseline Evidence Capture Limitation

Some files later named as baseline captures were created after the configuration had already been segmented. Those files report failed connections and therefore cannot serve as historical baseline evidence.

The original baseline observations remain documented in the test matrix and report, but the evidence archive would be stronger if the baseline commands had been captured to uniquely identified files before the configuration change.

11.6 Future Work

Future work should:

repeat the experiment with an actual application container;

isolate ports removal from the internal setting;

test additional network paths;

validate IPv6 behavior where applicable;

add automated configuration checks;

compare Windows Docker Desktop with native Linux Docker;

capture baseline evidence before every configuration change; and

test application-level operations rather than only TCP reachability.

12. AI Use and Verification

AI assistance was used as a project-support tool for planning, documentation, interpretation, and refinement.

AI-assisted work included:

narrowing the research question;

developing a testable hypothesis;

organizing the methodology;

structuring the evidence index and test matrix;

identifying the need to distinguish baseline observations from later evidence;

drafting report sections; and

reviewing whether conclusions were stronger than the available evidence justified.

The technical claims were independently verified against command output and observed Docker behavior.

In particular, the following claims were checked against the project evidence:

baseline host connectivity to PostgreSQL and Redis succeeded;

baseline application-container connectivity succeeded;

the segmented network reported internal: true;

PostgreSQL and Redis had no published host ports after segmentation;

segmented host TCP tests failed;

segmented direct container-IP tests failed; and

segmented application-container TCP tests succeeded.

A key refinement resulting from verification was the decision not to claim that internal: true alone caused the result. The report instead attributes the observed security outcome to the combined tested configuration of an internal network and removal of published host ports.

AI output was therefore treated as draft analytical assistance rather than as primary technical evidence.

13. Conclusion

This project evaluated whether Docker network segmentation could reduce direct host-to-database connectivity while preserving legitimate application-container connectivity.

The baseline configuration exposed PostgreSQL on TCP 5432 and Redis on TCP 6379 through published Docker host ports. The Windows host successfully connected to both services, and the application-test container also successfully connected to both.

The segmented configuration removed the host port mappings and changed the shared Docker network to internal: true.

After the change:

Windows host -> PostgreSQL: blocked in the tested TCP path;

Windows host -> Redis: blocked in the tested TCP path;

Windows host -> PostgreSQL container IP: blocked in the tested TCP path;

Windows host -> Redis container IP: blocked in the tested TCP path;

application-test -> PostgreSQL: successful; and

application-test -> Redis: successful.

The results support the project hypothesis for the tested configuration.

The main security conclusion is that the segmented configuration reduced unnecessary host-to-database network exposure without preventing the tested application-side communication required by the services.

However, the result should not be interpreted as proof of complete isolation or as proof that internal: true alone produced the security effect. The experiment changed host port exposure and network internalization simultaneously, and it was performed in a Windows Docker Desktop environment using a controlled application-test container.

The recommended security posture is therefore to keep sensitive database services off unnecessary published host ports, use appropriately scoped internal networks, restrict network membership, protect credentials, and combine network segmentation with authentication, authorization, patching, and configuration-drift controls.

14. References

The following sources should be used for the final reference list and verified against the exact versions consulted during final editing:

Docker Documentation. Docker Compose file reference: networks.

Docker Documentation. Docker networking overview.

Docker Documentation. Bridge network driver documentation.

PostgreSQL Documentation. PostgreSQL client/server connection documentation.

Redis Documentation. Redis security and networking documentation.

Cybersecurity, AI & Security+ Final Project Guide supplied for the course.

Project-local Docker Compose configuration and evidence files.

The final submission should use the citation style required by the course and include access dates where required.

Appendix A. Test Commands

A.1 Configuration Validation

docker compose config
docker compose ps
docker network inspect week1_medusa-net

A.2 Published-Port Validation

docker port medusa-postgres
docker port medusa-redis

A.3 Host Connectivity Tests

PostgreSQL:

powershell -Command "Test-NetConnection -ComputerName 127.0.0.1 -Port 5432"

Redis:

powershell -Command "Test-NetConnection -ComputerName 127.0.0.1 -Port 6379"

A.4 Application-Container Connectivity Tests

PostgreSQL:

docker exec medusa-app-test sh -c "nc -zv medusa-postgres 5432"

Redis:

docker exec medusa-app-test sh -c "nc -zv medusa-redis 6379"

A.5 Direct Container-IP Tests

PostgreSQL:

powershell -Command "Test-NetConnection -ComputerName 172.19.0.3 -Port 5432"

Redis:

powershell -Command "Test-NetConnection -ComputerName 172.19.0.2 -Port 6379"

Appendix B. Expected Before-and-After Architecture

B.1 Baseline

                Windows Host
                     |
          +----------+----------+
          |                     |
      TCP 5432              TCP 6379
          |                     |
          v                     v
    PostgreSQL                Redis
    container                 container
          \                     /
           \                   /
            +-----------------+
            | Docker bridge   |
            | medusa-net      |
            | internal=false  |
            +-----------------+
                    ^
                    |
                app-test

B.2 Segmented Configuration

                Windows Host
                     |
              No published DB ports
                     |
                X          X
                |          |
                v          v
          PostgreSQL     Redis
                ^          ^
                 \        /
                  \      /
             +----------------+
             | Docker bridge  |
             | medusa-net     |
             | internal=true  |
             +----------------+
                     ^
                     |
                  app-test

The segmented design preserves the application-side path while removing the tested direct host paths.

Appendix C. Evidence Directory

The project evidence is organized approximately as follows:

evidence/
├── evidence-index.md
├── logs/
│   ├── segmented-compose-config.txt
│   ├── segmented-compose-ps.txt
│   ├── segmented-network-inspect.txt
│   ├── segmented-postgres-ports.txt
│   ├── segmented-redis-ports.txt
│   ├── T-005-host-postgres.txt
│   ├── T-006-host-redis.txt
│   ├── T-007-app-postgres.txt
│   ├── T-008-app-redis.txt
│   ├── T-009-host-postgres-ip.txt
│   └── T-010-host-redis-ip.txt
├── redacted-data/
└── screenshots/

Historical baseline observations T-001 through T-004 are documented in the test matrix and evidence index. Later files that were created with baseline filenames after segmentation are retained for auditability but are not treated as baseline evidence.

Appendix D. Project Artifacts

The project repository contains or is intended to contain:

final-project/
├── README.md
├── Security_Report.md
├── capstone-proposal.md
├── environment.md
├── docker-compose.yml
├── docker-compose.baseline.yml
├── tests/
│   └── test-matrix.md
├── evidence/
│   ├── evidence-index.md
│   ├── screenshots/
│   ├── logs/
│   └── redacted-data/
├── analysis/
├── diagrams/
│   ├── Baseline_architecture.png
│   └── Segmented_architecture.png
├── references/
└── .gitignore

No passwords, tokens, private keys, real personal data, or other secrets should be committed to the repository.