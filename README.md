# 🔐 DevSecOps POC: Spring Boot Security CI/CD Pipeline

> A comprehensive proof-of-concept demonstrating security-first CI/CD practices through multi-layered vulnerability scanning integrated into a Spring Boot application pipeline.

---

## 📑 Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [Architecture & Pipeline](#architecture--pipeline)
- [Security Tools](#security-tools)
- [Setup Guide](#setup-guide)
- [Running Locally](#running-locally)
- [Understanding Results](#understanding-results)
- [Intentional Vulnerabilities](#intentional-vulnerabilities)
- [Troubleshooting](#troubleshooting)
- [Key Learnings](#key-learnings)

---

## 📌 Overview

This POC showcases how to build a **security-by-design CI/CD pipeline** that automatically detects and prevents vulnerable code, dependencies, containers, and secrets from reaching production.

### What is Vulnerability Scanning?

Vulnerability scanning is the automated detection of security flaws across multiple layers:
- **Source Code**: Logic flaws and insecure patterns
- **Dependencies**: Known vulnerabilities in third-party libraries
- **Container Images**: Vulnerable OS packages and layers
- **Secrets**: Hardcoded credentials and API keys

### Why This Matters

- ✅ **Shift-Left Security**: Catch issues early in development
- ✅ **Compliance**: Meet regulatory requirements (OWASP, PCI-DSS, etc.)
- ✅ **Reduced Risk**: Prevent vulnerable code from reaching production
- ✅ **Cost Savings**: Fix issues early = lower remediation costs

---

## 🚀 Quick Start

### Prerequisites

- **Java 21+** (for local development)
- **Maven 3.9+**
- **Docker** (for container testing)
- **Git** (with git history enabled for secret scanning)
- **GitHub Account** (to view workflow runs)

### 1-Minute Setup

```bash
# Clone the repository
git clone <repo-url>
cd devsecops-service

# Run locally
mvn spring-boot:run

# Application starts at http://localhost:8080
```

---

## ⚙️ Architecture & Pipeline

### Pipeline Flow Diagram

```
┌─────────────────────────────────────────────────────────┐
│  Developer Pushes Code to Repository                   │
└──────────────────┬──────────────────────────────────────┘
                   │
        ┌──────────▼──────────┐
        │  GitHub Actions     │
        │  Workflow Triggered │
        └──────────┬──────────┘
                   │
    ┌──────────────┼──────────────┐
    │              │              │
    ▼              ▼              ▼
┌──────────┐ ┌──────────┐ ┌──────────────┐
│ Gitleaks │ │ CodeQL   │ │ Build Maven  │
│ Scan     │ │ Analysis │ │ & Setup Java │
│(Secrets) │ │ (SAST)   │ │              │
└──────────┘ └──────────┘ └──────────────┘
    │              │              │
    └──────────────┼──────────────┘
                   │
    ┌──────────────┼──────────────┐
    │              │              │
    ▼              ▼              ▼
┌──────────┐ ┌──────────┐ ┌──────────────┐
│ SonarQube│ │  Snyk    │ │  Build       │
│  Scan    │ │ Scan     │ │  Docker      │
│(SAST)    │ │(Deps)    │ │  Image       │
└──────────┘ └──────────┘ └──────────────┘
    │              │              │
    └──────────────┼──────────────┘
                   │
                   ▼
            ┌──────────────────┐
            │  Trivy Scan      │
            │  (Container)     │
            └──────────────────┘
                   │
            ┌──────▼──────┐
            │  CodeQL     │
            │  Reporting  │
            └──────┬──────┘
                   │
        ┌──────────▼──────────┐
        │  All Checks Pass?   │
        └──────┬─────────┬────┘
             Yes │       │ No
               ▼        ▼
          ┌──────┐  ┌──────────────┐
          │Deploy│  │ Fail Pipeline │
          └──────┘  │ & Notify Team │
                    └──────────────┘
```

### Scanning Layers

| Layer | Tool | Focus | Enforcement |
|-------|------|-------|-------------|
| **Secrets** | Gitleaks | Hardcoded credentials | ❌ Fails build |
| **Source Code** | CodeQL + SonarQube | Logic flaws, code quality | ⚠️ Reports (SonarQube) |
| **Dependencies** | Snyk | Vulnerable packages | ❌ Fails build |
| **Container** | Trivy | OS package vulnerabilities | ⚠️ Reports (continues) |

---

## 🛠️ Security Tools

### 1. **Gitleaks** - Secret Detection
```
Purpose: Detect hardcoded secrets (API keys, passwords, tokens)
What it scans: Git history and current code
Enforcement: ❌ BUILD FAILS if secrets detected
How it works:
  - Pattern matching for common secret formats
  - Entropy detection for random-looking strings
  - Scans entire git history (fetch-depth: 0)
```

**Secrets this tool catches:**
- AWS Access Keys
- GitHub Tokens
- API Keys
- Database credentials
- Private keys

---

### 2. **CodeQL** - Static Application Security Testing (SAST)
```
Purpose: Analyze code for security vulnerabilities and logic flaws
What it scans: Source code structure and control flow
Enforcement: ⚠️ REPORTS (can be configured to fail)
How it works:
  - Semantic code analysis
  - Detects unsafe patterns
  - Cross-function vulnerability tracking
```

**Vulnerabilities CodeQL detects:**
- SQL Injection
- Command Injection
- Path Traversal
- Unsafe serialization
- Sensitive data exposure

---

### 3. **SonarQube** - Code Quality & Security
```
Purpose: Comprehensive code quality and security analysis
What it scans: Source code patterns and metrics
Enforcement: ⚠️ REPORTS (warnings don't fail build)
How it works:
  - Rule-based analysis
  - Bug pattern detection
  - Code coverage tracking
  - Maintainability scoring
```

**What SonarQube evaluates:**
- Security vulnerabilities
- Code smells
- Code duplication
- Test coverage
- Complexity metrics

**Note:** In this POC, SonarQube failures are **reported but don't block deployment**—demonstrating why enforcement matters!

---

### 4. **Snyk** - Dependency Vulnerability Scanning
```
Purpose: Identify known vulnerabilities in Maven dependencies
What it scans: pom.xml dependencies
Enforcement: ❌ BUILD FAILS if vulnerabilities found
How it works:
  - Compares against vulnerability database
  - Checks transitive dependencies
  - Provides remediation suggestions
```

**This POC includes:**
- Intentionally outdated Micrometer dependency
- Known vulnerabilities to demonstrate scanning

---

### 5. **Trivy** - Container Image Scanning
```
Purpose: Scan Docker image for vulnerable OS packages
What it scans: Docker image layers
Enforcement: ⚠️ REPORTS (continues with --exit-code 1 overridden)
How it works:
  - Analyzes base image
  - Scans all installed packages
  - Identifies CVEs in dependencies
```

**Scans for:**
- CVEs in base OS (Debian packages)
- Library vulnerabilities
- Configuration issues

---

## 📋 Setup Guide

### 1. GitHub Secrets Configuration

Add these to your GitHub repository settings (`Settings > Secrets and variables > Actions`):

```
SONAR_TOKEN      = Your SonarCloud token
SONAR_ORG        = Your SonarCloud organization
SNYK_TOKEN       = Your Snyk API token (optional for demo)
```

### How to Get These Tokens:

**SonarCloud:**
1. Go to https://sonarcloud.io
2. Sign up / Log in
3. Create organization
4. Generate token at Account > Security
5. Copy token to `SONAR_TOKEN`

**Snyk:**
1. Go to https://snyk.io
2. Sign up / Log in
3. Go to Settings > API Token
4. Copy token to `SNYK_TOKEN`

### 2. Local Development Setup

```bash
# Install Java 21
# (Use your system's package manager or SDKMAN)

# Install Maven
brew install maven  # macOS
# or
sudo apt-get install maven  # Linux

# Verify installations
java -version
mvn -version
```

---

## ▶️ Running Locally

### Run as Spring Boot Application

```bash
mvn spring-boot:run
```

The application starts at `http://localhost:8080`

### Run with Docker

```bash
# Build image
docker build -t springboot-devsecops .

# Run container
docker run -p 8080:8080 springboot-devsecops
```

### Test the Application

#### Health Check
```bash
curl http://localhost:8080/actuator/health
```

#### Response
```json
{"status":"UP"}
```

#### Test XSS Vulnerability (intentional)
```bash
curl "http://localhost:8080/?name=<script>alert(1)</script>"
```

This will echo back the unescaped input, demonstrating the XSS vulnerability.

---

## 📊 Understanding Results

### Viewing Workflow Results

1. Go to **GitHub Repository > Actions**
2. Click the workflow run
3. Review each job:

| Tool | Location | What to Look For |
|------|----------|------------------|
| Gitleaks | Logs > Gitleaks Scan | ❌ Any matches = build fails |
| CodeQL | Security > Code Scanning | Check severity levels |
| SonarQube | Check SonarCloud link in logs | Quality gates status |
| Snyk | Logs > Run Snyk Scan | Vulnerability count & severity |
| Trivy | Logs > Run Trivy Scan | HIGH/CRITICAL vulnerabilities |

### Interpreting Severity Levels

```
CRITICAL  🔴  Requires immediate action
HIGH      🟠  Should be fixed before release  
MEDIUM    🟡  Address in next sprint
LOW       🟢  Consider for future improvements
```

---

## 🔍 Intentional Vulnerabilities

This POC **intentionally includes vulnerabilities** for demonstration:

### 1. XSS Vulnerability
**File:** `Controller.java`
```java
// Vulnerable: Input not escaped
@GetMapping("/")
public String hello(@RequestParam(required = false) String name) {
    if (name == null) name = "World";
    return "<h1>Hello " + name + "!</h1>";  // ⚠️ XSS!
}
```

**How to fix:**
```java
// Use Spring's HTML escaping
return "<h1>Hello " + HtmlUtils.htmlEscape(name) + "!</h1>";
```

### 2. Outdated Micrometer Dependency
**File:** `pom.xml`
```xml
<!-- Previous version had known CVEs -->
<micrometer.version>1.12.0</micrometer.version>  <!-- ⚠️ Vulnerable -->

<!-- Fixed to: -->
<micrometer.version>1.16.6</micrometer.version>  <!-- ✅ Patched -->
```

### 3. Secrets in Code (Demo)
**Previously committed:** Hardcoded API keys (now removed)
- Gitleaks still detects them in git history
- Use `--fetch-depth: 0` to scan entire history

---

## 🐛 Troubleshooting

### Issue: "Sonar token not recognized"
```
❌ ERROR:401 Unauthorized
```
**Solution:**
- Verify SONAR_TOKEN in GitHub Secrets
- Check token hasn't expired in SonarCloud
- Ensure SONAR_ORG matches your organization

### Issue: "Snyk scan fails with SNYK_TOKEN missing"
```
❌ ERROR: No valid SNYK_TOKEN
```
**Solution:**
- Add SNYK_TOKEN to GitHub Secrets (required for full scans)
- Or remove from workflow if just doing POC locally

### Issue: "Docker build fails"
```
❌ ERROR: Cannot find Java 21
```
**Solution:**
```bash
# Check your pom.xml
cat pom.xml | grep "java.version"

# Ensure Maven has right JDK
mvn -version | grep "Java version"

# Force Maven to use Java 21
export JAVA_HOME=/path/to/java21
```

### Issue: "Trivy scan timeout"
```
❌ Scanning takes 5+ minutes
```
**Solution:**
- Trivy caches databases locally
- First run is slow, subsequent runs are faster
- Ensure sufficient disk space (~1GB)

### Issue: "Port 8080 already in use"
```
❌ ERROR: Bind exception on port 8080
```
**Solution:**
```bash
# Find process using port 8080
lsof -i :8080

# Kill the process
kill -9 <PID>

# Or use different port
mvn spring-boot:run -Dserver.port=8081
```

---

## 🎯 Key Learnings

### What This POC Demonstrates

| Tool | Behavior | Lesson |
|------|----------|--------|
| **SonarQube** | Reports vulnerabilities but allows deployment | ⚠️ Reporting ≠ Enforcement |
| **Snyk** | Blocks build on dependency CVEs | ✅ Enforcement prevents bad deployments |
| **Trivy** | Scans containers; can be configured to block | ⚠️ Container security is critical |
| **Gitleaks** | Blocks build immediately on secrets | ✅ Secrets must be enforced from start |

### Best Practices Illustrated

1. **Layered Security**: Don't rely on single tool
2. **Enforcement Matters**: Failing builds prevents bad deployments
3. **Early Detection**: Scanning in CI/CD catches issues before production
4. **Continuous Updates**: Keep dependencies patched
5. **Historical Scanning**: Check entire git history for secrets

### Production Recommendations

- ✅ Make all security checks **mandatory** (not just reporting)
- ✅ Set **strict severity thresholds** for blocking builds
- ✅ **Integrate with incident response**: Alert on policy violations
- ✅ **Regular remediation sprints** for dependency updates
- ✅ **Track metrics**: Chart vulnerability trends over time

---

## 📚 Additional Resources

### Documentation
- [SonarQube](https://docs.sonarqube.org/)
- [Snyk Documentation](https://docs.snyk.io/)
- [Trivy GitHub](https://github.com/aquasecurity/trivy)
- [CodeQL Docs](https://codeql.github.com/docs/)
- [Gitleaks](https://github.com/gitleaks/gitleaks)

### OWASP References
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [OWASP XSS Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [OWASP Dependency Check](https://owasp.org/www-project-dependency-check/)

### DevSecOps Learning
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
- [CIS Controls](https://www.cisecurity.org/cis-controls/)

---

## 🤝 Contributing

This is a POC for learning purposes. To extend it:

1. Add more intentional vulnerabilities
2. Integrate additional scanning tools (OWASP Dependency-Check, etc.)
3. Add policy-as-code (OPA/Kyverno)
4. Set up artifact scanning for deployment
5. Add SLSA provenance tracking

---

## ⚖️ License & Disclaimer

**⚠️ Important:** This application is **intentionally vulnerable** for educational purposes. 

- ❌ Do NOT use this in production
- ❌ Do NOT deploy to public internet
- ✅ Use only in isolated lab environments
- ✅ Excellent for learning DevSecOps practices

