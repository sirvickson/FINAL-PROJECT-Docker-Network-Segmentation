# Weekend Recon: Node Base Image Security Scan

## Project Overview

This project demonstrates a proactive **DevSecOps** security assessment of the official **Node.js 18 Alpine** Docker image before deploying a Medusa headless e-commerce backend.

The objective is to identify known vulnerabilities in the base image using **Trivy** and document the findings before the application reaches production.

---

## Objectives

- Pull the official `node:18-alpine` Docker image.
- Perform a vulnerability scan using Trivy.
- Analyze the scan results.
- Prioritize the most critical vulnerabilities.
- Document mitigation recommendations.
- Practice integrating security into the Software Development Life Cycle (SDLC).

---

## Technologies Used

- Docker
- Trivy
- Alpine Linux
- Node.js 18
- Git
- GitHub
- Markdown

---

## Project Structure

```
.
├── README.md
├── security_report.md
├── weekend_recon.png
└── trivy-node18-alpine.json
```

---

## Pull the Target Image

```bash
docker pull node:18-alpine
```

---

## Run the Vulnerability Scan

```bash
docker run --rm \
-v /var/run/docker.sock:/var/run/docker.sock \
aquasec/trivy image node:18-alpine
```

To export the scan results as JSON:

```bash
docker run --rm \
-v /var/run/docker.sock:/var/run/docker.sock \
-v $(pwd):/output \
aquasec/trivy image \
--format json \
-o /output/trivy-node18-alpine.json \
node:18-alpine
```

---

## Scan Summary

**Target Image**

- node:18-alpine

**Operating System**

- Alpine Linux 3.21.3

**Node Version**

- 18.20.8

**Scanner**

- Trivy v0.72.0

The scan detected several vulnerabilities affecting packages installed within the container image. The most significant findings were related to the OpenSSL (`libcrypto3`) package.

---

# Top Vulnerabilities

## 1. CVE-2026-31789 (CRITICAL)

**Package**

```
libcrypto3 (OpenSSL)
```

**Installed Version**

```
3.3.3-r0
```

**Fixed Version**

```
3.3.7-r0
```

### Description

This vulnerability affects OpenSSL and may allow a specially crafted X.509 certificate to trigger a memory corruption issue.

### Potential Impact

An attacker could potentially:

- Crash the application (Denial of Service)
- Execute malicious code under certain conditions

### Recommendation

Upgrade the OpenSSL package by using an updated Node Alpine image or upgrading Alpine packages to version **3.3.7-r0** or later.

---

## 2. CVE-2025-15467 (HIGH)

**Package**

```
libcrypto3 (OpenSSL)
```

**Installed Version

```
3.3.3-r0
```

**Fixed Version**

```
3.3.6-r0
```

### Description

This vulnerability affects the processing of encrypted CMS (Cryptographic Message Syntax) messages.

### Potential Impact

A maliciously crafted encrypted message could:

- Crash the application
- Potentially lead to remote code execution

### Recommendation

Upgrade OpenSSL to **3.3.6-r0** or later before deploying the application.

---

# Why These Vulnerabilities Matter

The Medusa backend will be Internet-facing and will rely heavily on secure HTTPS communication. Since both vulnerabilities affect OpenSSL, they pose a greater security risk than the lower-severity BusyBox vulnerabilities identified in the scan.

Updating the vulnerable packages before deployment significantly reduces the application's attack surface.

---

# Mitigation Steps

- Pull the latest Node Alpine base image.
- Upgrade Alpine packages.
- Rebuild the Docker image.
- Re-run Trivy to verify remediation.
- Continue vulnerability scanning during the CI/CD pipeline.

---

# AI Prompt Used

> I am preparing to deploy an e-commerce backend on the node:18-alpine Docker image. Analyze these Trivy scan results. Identify the top 2 vulnerabilities (CVEs) I need to be aware of, and explain in simple terms how an attacker might exploit them.

---

# AI Analysis Summary

The scan identified two high-priority OpenSSL vulnerabilities:

- **CVE-2026-31789 (Critical)** – Heap buffer overflow that may result in denial of service or potential remote code execution.
- **CVE-2025-15467 (High)** – Stack buffer overflow during CMS message parsing that could allow denial of service or remote code execution.

Both vulnerabilities were found in the `libcrypto3` package and have available fixes.

---

# Verification Checklist

- [x] Pulled `node:18-alpine`
- [x] Executed Trivy vulnerability scan
- [x] Exported scan results to JSON
- [x] Reviewed identified CVEs
- [x] Prioritized highest-risk vulnerabilities
- [x] Documented findings
- [x] Captured scan screenshot (`weekend_recon.png`)

---

# Git Commands

```bash
git status
git add .
git commit -m "Weekend Recon: Node Base Image Scanned"
git push origin main