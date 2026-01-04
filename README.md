<div align="center">

# Sentry

![Go](https://img.shields.io/badge/Go-1.21+-00ADD8?style=flat&logo=go&logoColor=white)
![Dagger](https://img.shields.io/badge/Dagger-0.19+-131313?style=flat&logo=dagger&logoColor=white)
![Trivy](https://img.shields.io/badge/Trivy-Scanner-1904DA?style=flat)
![Grype](https://img.shields.io/badge/Grype-Scanner-4A154B?style=flat)
![Snyk](https://img.shields.io/badge/Snyk-Scanner-4C4A73?style=flat)
![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)

**Container Security Auditing for Dagger Pipelines**

Sentry is a Dagger module that audits containers for security compliance. It performs automated security checks, integrates with multiple vulnerability scanners, and generates compliance-ready reports for CI/CD pipelines.

This README is the single source of documentation for this module. It is written for both non-technical and technical readers.

</div>

---

## Contents

- [Overview](#overview)
- [Features](#features)
- [Glossary](#glossary)
- [Quick Start (No Technical Knowledge Required)](#quick-start-no-technical-knowledge-required)
- [Quick Start (Developers)](#quick-start-developers)
- [Architecture](#architecture)
- [Scanner Guide](#scanner-guide)
- [Security Checks](#security-checks)
- [Report Formats](#report-formats)
- [CI/CD Integration](#cicd-integration)
- [API Reference](#api-reference)
- [Scoring Algorithm](#scoring-algorithm)
- [Configuration Examples](#configuration-examples)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

Sentry helps development teams audit container images for security vulnerabilities and misconfigurations before deployment. It integrates into Dagger pipelines to provide:

- **Automated vulnerability scanning** using industry-standard scanners
- **Security best practice checks** for common misconfigurations
- **Compliance-ready reports** in Markdown and JSON formats
- **CI/CD gates** with pass/fail exit codes
- **Security scoring** for tracking improvements over time

---

## Features

| Category | Features |
|----------|----------|
| **Vulnerability Scanning** | Trivy, Grype, Snyk, Wiz, Black Duck integration; custom scanner support; severity thresholds |
| **Security Checks** | Non-root user verification; secret detection in environment variables; health check capability |
| **Reporting** | Markdown reports with executive summary; JSON export for automation; security scoring (0-100) |
| **CI/CD** | Pass/fail exit codes; GitHub Actions integration; GitLab CI integration; any CI with Dagger support |
| **Extensibility** | Custom scanner containers; configurable output formats; pluggable architecture |

---

## Glossary

| Term | Definition |
|------|------------|
| **Container** | A packaged application image to be audited (e.g., `nginx:latest`) |
| **Scanner** | A vulnerability detection tool (Trivy, Grype, Snyk, Wiz, Black Duck) |
| **Vulnerability** | A security weakness identified by CVE ID (e.g., CVE-2024-1234) |
| **Severity** | Risk level: CRITICAL, HIGH, MEDIUM, LOW, INFO |
| **Security Check** | A configuration verification (e.g., non-root user) |
| **Audit** | The complete security assessment process |
| **Score** | A 0-100 rating based on findings |
| **Threshold** | The minimum severity level that causes an audit to fail |

---

## Quick Start (No Technical Knowledge Required)

### Prerequisites

1. **Install Docker Desktop**
   - Mac/Windows: https://www.docker.com/products/docker-desktop
   - Linux: https://docs.docker.com/engine/install/

2. **Install Dagger CLI**
   ```bash
   curl -fsSL https://dl.dagger.io/dagger/install.sh | sh
   ```

### Run Your First Audit

1. **Open a terminal**

2. **Run a security audit on any container:**
   ```bash
   dagger call -m github.com/sylvester-francis/Sentry \
     scan --container=nginx:latest \
     report
   ```

3. **View the results** - You'll see a security report with:
   - Overall pass/fail status
   - Security score (0-100)
   - List of security checks and their results
   - Vulnerability summary
   - Recommendations for improvement

### Understanding the Output

```
┌─────────────────────────────────────────────────────────────┐
│  STATUS: PASSED                                             │
├─────────────────────────────────────────────────────────────┤
│  Score: 85/100                                              │
│  Checks: 2 passed, 1 failed                                 │
│  Vulnerabilities: 0 total (0 critical)                      │
└─────────────────────────────────────────────────────────────┘
```

- **STATUS**: PASSED means safe to deploy; FAILED means issues need attention
- **Score**: Higher is better; 100 is perfect
- **Checks**: Security configuration checks
- **Vulnerabilities**: CVEs found by the scanner

---

## Quick Start (Developers)

### Option A: Use from Daggerverse (Recommended)

No installation needed. Call directly:

```bash
# Basic audit with Trivy (default scanner)
dagger call -m github.com/sylvester-francis/Sentry \
  scan --container=myapp:latest \
  report

# Get JSON output
dagger call -m github.com/sylvester-francis/Sentry \
  scan --container=myapp:latest \
  json

# CI exit code (0=pass, 1=fail)
dagger call -m github.com/sylvester-francis/Sentry \
  scan --container=myapp:latest \
  exit-code
```

### Option B: Clone and Develop Locally

```bash
# Clone the repository
git clone https://github.com/sylvester-francis/Sentry.git
cd Sentry

# Verify module
dagger develop
dagger functions

# Run tests
dagger call test

# Run audit
dagger call scan --container=alpine:latest report
```

### Option C: Use in Your Dagger Pipeline

```go
// In your Dagger module
func (m *MyModule) SecurityAudit(ctx context.Context, container *dagger.Container) (string, error) {
    return dag.Sentry().
        Scan(container).
        WithGrype().  // Use Grype instead of Trivy
        FailOn("CRITICAL").  // Only fail on critical vulns
        Report(ctx)
}
```

---

## Architecture

### System Overview

```mermaid
flowchart TB
    subgraph Sentry["Sentry Module"]
        subgraph Input["Input Components"]
            SC["Security Checks"]
            VS["Vulnerability Scanner"]
            RG["Report Generator"]
        end
        
        SC --> AE
        VS --> AE
        RG --> AE
        
        AE["Audit Engine\n(Scoring & Aggregation)"]
        
        AE --> MD["Markdown Report"]
        AE --> JSON["JSON Export"]
        AE --> EC["Exit Code (CI/CD)"]
    end
```

### Scanner Integration Flow

```mermaid
flowchart LR
    A["Container Image"] --> B["Export as Tarball"]
    B --> C["Scanner Container"]
    C --> D["Parse Output"]
    
    D --> E["Trivy\n(default)"]
    D --> F["Grype\n(Anchore)"]
    D --> G["Snyk\n(token required)"]
    D --> H["Wiz\n(credentials)"]
    D --> I["Black Duck\n(enterprise)"]
```

### Audit Flow

```mermaid
sequenceDiagram
    participant User
    participant Sentry
    participant Scanner
    participant Checks
    
    User->>Sentry: scan --container=image
    Sentry->>Checks: Run security checks
    Checks-->>Sentry: Check results
    Sentry->>Scanner: Export & scan image
    Scanner-->>Sentry: Vulnerabilities
    Sentry->>Sentry: Calculate score
    Sentry-->>User: Report / JSON / Exit code
```

### File Structure

```
Sentry/
├── dagger.json              # Dagger module configuration
├── README.md                # This documentation
├── LICENSE                  # MIT License
└── .dagger/
    ├── main.go              # Module entry point and builder API
    ├── types.go             # Type definitions (Severity, CheckStatus, etc.)
    ├── checks.go            # Security check implementations
    ├── scanners.go          # Scanner configurations and parsers
    ├── trivy.go             # Trivy-specific integration
    ├── report.go            # Report generation and scoring
    └── testing.go           # Unit tests (43 test cases)
```

---

## Scanner Guide

### Supported Scanners

| Scanner | Image | Auth Required | Best For |
|---------|-------|---------------|----------|
| **Trivy** | `aquasec/trivy:latest` | None | General purpose, fast |
| **Grype** | `anchore/grype:latest` | None | Deep analysis, SBOM |
| **Snyk** | `snyk/snyk:docker` | SNYK_TOKEN | Enterprise, developer-friendly |
| **Wiz** | `wizsecurity/wiz-cli:latest` | WIZ_CLIENT_ID/SECRET | Cloud-native, CSPM |
| **Black Duck** | `blackducksoftware/detect:latest` | BLACKDUCK_API_TOKEN | License compliance |

### Scanner Examples

#### Trivy (Default)

```bash
# Trivy is used by default - no configuration needed
dagger call scan --container=nginx:latest report
```

#### Grype

```bash
# Grype provides detailed SBOM-based scanning
dagger call scan --container=nginx:latest with-grype report
```

#### Snyk

```bash
# Snyk requires authentication
export SNYK_TOKEN=your-token
dagger call scan --container=nginx:latest \
  with-snyk --token=env:SNYK_TOKEN \
  report
```

#### Wiz

```bash
# Wiz requires client credentials
dagger call scan --container=nginx:latest \
  with-wiz --client-id=env:WIZ_CLIENT_ID --client-secret=env:WIZ_CLIENT_SECRET \
  report
```

#### Black Duck

```bash
# Black Duck requires URL and token
dagger call scan --container=nginx:latest \
  with-black-duck --url="https://your-blackduck.com" --token=env:BLACKDUCK_API_TOKEN \
  report
```

#### Custom Scanner

```bash
# Use any scanner that outputs JSON
dagger call scan --container=nginx:latest \
  with-custom-scanner \
    --image="my-scanner:latest" \
    --args='["scan", "--format", "json", "/image.tar"]' \
    --output-format="trivy" \
  report
```

#### Disable Scanning

```bash
# Run only security checks, no vulnerability scanning
dagger call scan --container=nginx:latest without-scanner report
```

---

## Security Checks

### Non-Root User Check

**Severity**: HIGH

Verifies that the container is configured to run as a non-root user. Running as root inside containers is a security risk because:

- Container escape vulnerabilities have higher impact
- File system permissions are bypassed
- Network capabilities may be elevated

**Pass conditions**:
- Container USER directive is set to non-root
- Container runs as UID > 0

**Remediation**:
```dockerfile
# Add to your Dockerfile
USER nobody
# or
USER 1000:1000
```

### Secret Detection

**Severity**: HIGH

Scans environment variables for exposed credentials. Detected patterns:

| Secret Type | Pattern Example |
|-------------|-----------------|
| AWS Access Key | `AKIA...` |
| AWS Secret Key | 40-character base64 |
| GitHub Token | `ghp_`, `gho_`, `ghs_`, `ghr_` |
| JWT Token | `eyJ...` (Base64 JSON) |
| Private Key | `-----BEGIN RSA PRIVATE KEY-----` |
| Database URL | `postgresql://user:pass@host` |
| Slack Token | `xoxb-`, `xoxp-` |
| Generic API Key | `api_key`, `apikey`, etc. |

**Remediation**:
- Use Dagger secrets instead of environment variables
- Use secret management tools (Vault, AWS Secrets Manager)
- Never hardcode credentials in images

### Health Check Capability

**Severity**: INFO

Verifies that `curl` or `wget` is available for implementing container health checks.

**Pass conditions**:
- `curl` binary exists in PATH
- OR `wget` binary exists in PATH

**Remediation**:
```dockerfile
# Add curl for health checks
RUN apt-get update && apt-get install -y curl
HEALTHCHECK CMD curl -f http://localhost/ || exit 1
```

---

## Report Formats

### Markdown Report

The default report format with executive summary:

```
# Security Audit Report

## Executive Summary

┌─────────────────────────────────────────────────────────────┐
│  STATUS: FAILED                                             │
├─────────────────────────────────────────────────────────────┤
│  Score: 85/100                                              │
│  Timestamp: 2026-01-04T15:00:00Z                            │
│  Image: container                                           │
├─────────────────────────────────────────────────────────────┤
│  Checks: 2 passed, 1 failed                                 │
│  Vulnerabilities: 3 total (0 critical)                      │
└─────────────────────────────────────────────────────────────┘

## Security Checks

> **Non-Root User**
> 
> - Status: `FAIL`
> - Severity: HIGH
> - Details: Container is running as root user

## Vulnerability Summary

Critical:      0
High:          1  █████
Medium:        2  ██████████
Low:           0
─────────────────────
Total:         3

## Recommendations

1. **Non-Root User**: Container is running as root user
2. **High Vulnerabilities**: Review and patch 1 high-severity vulnerabilities
```

### JSON Report

Machine-readable format for automation:

```json
{
  "timestamp": "2026-01-04T15:00:00Z",
  "imageRef": "nginx:latest",
  "scannerUsed": "trivy",
  "passed": false,
  "score": 85,
  "checks": [
    {
      "name": "Non-Root User",
      "status": "FAIL",
      "severity": "HIGH",
      "details": "Container is running as root user"
    }
  ],
  "vulnerabilities": [
    {
      "packageName": "openssl",
      "cveId": "CVE-2024-1234",
      "severity": "HIGH",
      "installedVersion": "1.1.1",
      "fixedVersion": "1.1.2"
    }
  ],
  "vulnSummary": {
    "critical": 0,
    "high": 1,
    "medium": 2,
    "low": 0,
    "total": 3
  }
}
```

---

## CI/CD Integration

### GitHub Actions

```yaml
name: Security Audit

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Dagger
        uses: dagger/dagger-for-github@v5

      - name: Build Container
        run: docker build -t myapp:${{ github.sha }} .

      - name: Security Audit
        run: |
          dagger call -m github.com/sylvester-francis/Sentry \
            scan --container=myapp:${{ github.sha }} \
            exit-code

      - name: Generate Report
        if: always()
        run: |
          dagger call -m github.com/sylvester-francis/Sentry \
            scan --container=myapp:${{ github.sha }} \
            report > security-report.md

      - name: Upload Report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: security-report
          path: security-report.md
```

### GitLab CI

```yaml
stages:
  - build
  - security

build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

security_audit:
  stage: security
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - curl -fsSL https://dl.dagger.io/dagger/install.sh | sh
  script:
    - dagger call -m github.com/sylvester-francis/Sentry \
        scan --container=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA \
        exit-code
  artifacts:
    when: always
    paths:
      - security-report.md
    reports:
      dotenv: security-status.env
```

### Jenkins

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'docker build -t myapp:${BUILD_NUMBER} .'
            }
        }
        stage('Security Audit') {
            steps {
                sh '''
                    curl -fsSL https://dl.dagger.io/dagger/install.sh | sh
                    dagger call -m github.com/sylvester-francis/Sentry \
                        scan --container=myapp:${BUILD_NUMBER} \
                        exit-code
                '''
            }
        }
    }
    post {
        always {
            sh '''
                dagger call -m github.com/sylvester-francis/Sentry \
                    scan --container=myapp:${BUILD_NUMBER} \
                    report > security-report.md
            '''
            archiveArtifacts artifacts: 'security-report.md'
        }
    }
}
```

---

## API Reference

### Entry Points

| Function | Description |
|----------|-------------|
| `scan --container=<image>` | Initialize security audit for a container |
| `test` | Run internal unit tests (43 test cases) |

### Scanner Selection Methods

| Method | Arguments | Description |
|--------|-----------|-------------|
| `with-trivy` | None | Use Trivy scanner (default) |
| `with-grype` | None | Use Grype (Anchore) scanner |
| `with-snyk` | `--token` | Use Snyk scanner |
| `with-wiz` | `--client-id`, `--client-secret` | Use Wiz scanner |
| `with-black-duck` | `--url`, `--token` | Use Black Duck scanner |
| `with-custom-scanner` | `--image`, `--args`, `--output-format` | Use custom scanner |
| `without-scanner` | None | Disable vulnerability scanning |

### Configuration Methods

| Method | Arguments | Default | Description |
|--------|-----------|---------|-------------|
| `fail-on` | `--severity` | HIGH | Set minimum severity to fail audit |
| `with-secret-check` | `--enable` | true | Enable/disable secret detection |
| `with-non-root-check` | `--enable` | true | Enable/disable non-root check |
| `with-health-check` | `--enable` | true | Enable/disable health check |

### Output Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `audit` | AuditResult | Full audit result object |
| `report` | string | Markdown formatted report |
| `json` | string | JSON formatted report |
| `passed` | bool | true if audit passed |
| `exit-code` | int | 0 (pass) or 1 (fail) |

---

## Scoring Algorithm

The security score starts at 100 and deducts points based on findings:

### Check Failures

| Check Result | Severity | Deduction |
|--------------|----------|-----------|
| FAIL | CRITICAL | -25 points |
| FAIL | HIGH | -15 points |
| FAIL | MEDIUM | -10 points |
| FAIL | LOW | -5 points |
| WARN | Any | -3 points |

### Vulnerabilities

| Severity | Deduction per CVE |
|----------|-------------------|
| CRITICAL | -10 points |
| HIGH | -5 points |
| MEDIUM | -2 points |
| LOW | -1 point |

### Score Interpretation

| Score | Rating | Recommendation |
|-------|--------|----------------|
| 90-100 | Excellent | Safe to deploy |
| 70-89 | Good | Review findings before deploy |
| 50-69 | Fair | Address high-severity issues |
| 0-49 | Poor | Do not deploy |

---

## Configuration Examples

### Production (Strict)

```bash
# Fail on any HIGH or CRITICAL vulnerability
dagger call scan --container=myapp:latest \
  fail-on --severity=HIGH \
  report
```

### Development (Lenient)

```bash
# Only fail on CRITICAL vulnerabilities
dagger call scan --container=myapp:latest \
  fail-on --severity=CRITICAL \
  report
```

### Minimal Checks Only

```bash
# No vulnerability scanning, just configuration checks
dagger call scan --container=myapp:latest \
  without-scanner \
  report
```

### Full Audit with Grype

```bash
# Use Grype with all checks enabled
dagger call scan --container=myapp:latest \
  with-grype \
  fail-on --severity=HIGH \
  report
```

### Disable Specific Checks

```bash
# Skip non-root check for known root containers
dagger call scan --container=myapp:latest \
  with-non-root-check --enable=false \
  report
```

---

## Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| `Scanner returned empty results` | Minimal image or scanner timeout | Normal for images like `scratch` |
| `Container is running as root user` | No USER directive in Dockerfile | Add `USER` instruction |
| `No curl/wget found` | Minimal base image | Add health check tools or ignore |
| `401 Unauthorized` (Snyk/Wiz) | Invalid or expired token | Refresh authentication token |
| `Dagger engine not running` | Docker not started | Start Docker Desktop |

### Scanner-Specific Issues

**Trivy**:
- First run downloads vulnerability database (~30 seconds)
- Use `--no-progress` flag for CI environments

**Grype**:
- Requires database sync on first run
- May have different CVE coverage than Trivy

**Snyk**:
- Token must have container scanning permissions
- Free tier has limited scans per month

### Debug Mode

```bash
# Verbose output for debugging
dagger call -v scan --container=myapp:latest report
```

---

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Run tests (`dagger call test`)
5. Commit your changes (`git commit -m 'Add amazing feature'`)
6. Push to the branch (`git push origin feature/amazing-feature`)
7. Open a Pull Request

### Development Setup

```bash
git clone https://github.com/sylvester-francis/Sentry.git
cd Sentry
dagger develop
dagger call test  # Run 43 unit tests
```

---

## License

MIT License. See [LICENSE](LICENSE) for details.

---

<div align="center">

**Built with [Dagger](https://dagger.io)**

[Documentation](https://daggerverse.dev/mod/github.com/sylvester-francis/Sentry) · [Report Bug](https://github.com/sylvester-francis/Sentry/issues) · [Request Feature](https://github.com/sylvester-francis/Sentry/issues)

</div>