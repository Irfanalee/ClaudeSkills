---
name: cybersec-scanner
description: Security vulnerability scanner for code repositories. Use when asked to scan repos for security issues, find vulnerabilities, audit code security, check for secrets/credentials, identify OWASP vulnerabilities, review dependencies for CVEs, or perform any security analysis on codebases. Triggers on phrases like "scan for security", "find vulnerabilities", "security audit", "check for secrets", "CVE scan", or "security review".
---

# CyberSec Repository Scanner

Scan repositories for security vulnerabilities, exposed secrets, dependency issues, and code security anti-patterns.

## Scan Workflow

1. **Clone/access the repository**
2. **Run all applicable scans** (secrets, dependencies, SAST, config)
3. **Aggregate findings** by severity (Critical → High → Medium → Low)
4. **Generate report** with findings and remediation guidance

## Scan Categories

### 1. Secrets Detection

Scan for exposed credentials, API keys, tokens, and sensitive data.

**Patterns to detect:**
```
# API Keys & Tokens
AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY
GITHUB_TOKEN, gh[pousr]_[A-Za-z0-9_]{36,}
OPENAI_API_KEY, sk-[A-Za-z0-9]{48}
Stripe keys: sk_live_*, pk_live_*
Generic: api[_-]?key, secret[_-]?key, auth[_-]?token

# Credentials
password\s*=\s*["'][^"']+["']
private_key, -----BEGIN (RSA|EC|OPENSSH) PRIVATE KEY-----
jdbc:[a-z]+://[^:]+:[^@]+@

# Cloud & Infrastructure
Azure connection strings, GCP service account JSON
Database URLs with embedded credentials
```

**Scan command:**
```bash
# Using grep for quick scan
grep -rn --include="*.{js,ts,py,java,go,rb,php,env,yml,yaml,json,xml,config}" \
  -E "(api[_-]?key|secret|password|token|credential|private[_-]?key)\s*[:=]" . \
  --exclude-dir={node_modules,.git,vendor,dist,build}

# Check for .env files committed
find . -name "*.env*" -not -path "*/node_modules/*" -not -path "*/.git/*"

# Check git history for secrets (if git available)
git log -p --all -S "password" -- "*.py" "*.js" "*.env" 2>/dev/null | head -100
```

**Files to always check:**
- `.env`, `.env.*`, `*.env`
- `config.*`, `settings.*`, `secrets.*`
- `docker-compose.yml`, `Dockerfile`
- CI/CD files: `.github/workflows/*`, `.gitlab-ci.yml`, `Jenkinsfile`

### 2. Dependency Vulnerability Scan

Check dependencies for known CVEs.

**Python:**
```bash
# Install safety if needed
pip install safety pip-audit --break-system-packages

# Scan requirements
pip-audit -r requirements.txt 2>/dev/null || safety check -r requirements.txt

# Check for outdated packages
pip list --outdated 2>/dev/null
```

**JavaScript/Node:**
```bash
# npm audit (if package-lock.json exists)
npm audit --json 2>/dev/null

# Check for outdated
npm outdated 2>/dev/null
```

**Go:**
```bash
# govulncheck if available, otherwise check go.sum manually
go list -m all 2>/dev/null | head -50
```

**Ruby:**
```bash
# bundler-audit
bundle audit check 2>/dev/null
```

### 3. Static Analysis (SAST)

Identify code-level vulnerabilities.

**Injection vulnerabilities:**
```bash
# SQL Injection patterns
grep -rn --include="*.{py,js,ts,java,php,rb}" \
  -E "(execute|query|raw|cursor)\s*\([^)]*[\"']\s*\+|f[\"'].*SELECT|\.format\(.*SELECT" .

# Command Injection
grep -rn --include="*.{py,js,ts,java,php,rb}" \
  -E "(exec|system|popen|subprocess|child_process|eval)\s*\(" .

# XSS patterns (unescaped output)
grep -rn --include="*.{js,ts,jsx,tsx,html,php}" \
  -E "(innerHTML|dangerouslySetInnerHTML|v-html|ng-bind-html)" .
```

**Authentication/Authorization issues:**
```bash
# Hardcoded JWT secrets
grep -rn -E "jwt\.(sign|verify)\s*\([^)]+[\"'][A-Za-z0-9+/=]{20,}[\"']" .

# Missing auth checks (look for unprotected routes)
grep -rn -E "@(app|router)\.(get|post|put|delete|patch)\s*\([\"']/(?!auth|login|public)" .
```

**Cryptography issues:**
```bash
# Weak algorithms
grep -rn -E "(MD5|SHA1|DES|RC4|ECB)\s*\(|hashlib\.(md5|sha1)" .

# Hardcoded IVs or weak random
grep -rn -E "iv\s*=\s*[\"'][^\"']+[\"']|Math\.random\(\)" .
```

### 4. Configuration Security

**Web security headers (check server configs):**
```bash
# Missing security headers in responses
grep -rn --include="*.{js,ts,py,java,go}" \
  -E "(X-Frame-Options|Content-Security-Policy|X-Content-Type-Options)" .
```

**CORS misconfigurations:**
```bash
grep -rn -E "(Access-Control-Allow-Origin|cors)\s*[:=]\s*[\"']\*[\"']" .
```

**Debug/development settings in production:**
```bash
grep -rn -E "(DEBUG\s*=\s*True|NODE_ENV.*development|devMode.*true)" .
```

**Insecure defaults:**
```bash
# SSL verification disabled
grep -rn -E "(verify\s*=\s*False|rejectUnauthorized.*false|InsecureSkipVerify.*true)" .
```

### 5. Infrastructure as Code (IaC) Security

**Terraform:**
```bash
# Overly permissive IAM
grep -rn --include="*.tf" -E '(Action|Resource)\s*=\s*"\*"' .

# Public S3 buckets
grep -rn --include="*.tf" -E "acl\s*=\s*\"public" .
```

**Docker:**
```bash
# Running as root
grep -rn --include="Dockerfile*" -E "^USER root|^USER 0" .

# Latest tag usage
grep -rn --include="Dockerfile*" -E "FROM\s+\S+:latest" .

# Secrets in build args
grep -rn --include="Dockerfile*" -E "ARG\s+(password|secret|key|token)" .
```

**Kubernetes:**
```bash
# Privileged containers
grep -rn --include="*.{yml,yaml}" -E "privileged:\s*true" .

# hostPath volumes
grep -rn --include="*.{yml,yaml}" -E "hostPath:" .
```

## Severity Classification

| Severity | Description | Examples |
|----------|-------------|----------|
| **Critical** | Immediate exploitation risk | Exposed secrets, RCE vulnerabilities, public S3 with sensitive data |
| **High** | Significant security risk | SQL injection, auth bypass, critical CVEs (CVSS ≥ 9.0) |
| **Medium** | Moderate risk, harder to exploit | XSS, CSRF, high CVEs (CVSS 7.0-8.9), weak crypto |
| **Low** | Minor issues, defense in depth | Missing headers, info disclosure, medium CVEs |

## Report Format

Generate findings in this structure:

```markdown
# Security Scan Report
**Repository:** [name]
**Scan Date:** [date]
**Total Findings:** X (Critical: N, High: N, Medium: N, Low: N)

## Critical Findings
### [FINDING-001] Exposed AWS Credentials
- **File:** config/settings.py:45
- **Type:** Secret Exposure
- **Description:** AWS access key hardcoded in source
- **Evidence:** `AWS_ACCESS_KEY_ID = "AKIA..."`
- **Remediation:** Remove credential, rotate key, use environment variables or secrets manager

## High Findings
...

## Dependency Vulnerabilities
| Package | Version | CVE | Severity | Fixed In |
|---------|---------|-----|----------|----------|
| lodash  | 4.17.15 | CVE-2021-23337 | High | 4.17.21 |

## Recommendations
1. **Immediate:** Rotate all exposed credentials
2. **Short-term:** Update vulnerable dependencies
3. **Long-term:** Implement secrets management solution
```

## Quick Scan Script

For rapid assessment, run this comprehensive scan:

```bash
#!/bin/bash
REPO_PATH="${1:-.}"
echo "=== Security Scan: $REPO_PATH ==="

echo -e "\n[1/5] Secrets Detection..."
grep -rln --include="*.{py,js,ts,java,go,rb,php,env,yml,yaml,json,config}" \
  -E "(password|secret|api[_-]?key|token|credential)\s*[:=]\s*[\"'][^\"']{8,}" \
  "$REPO_PATH" 2>/dev/null | grep -v node_modules | head -20

echo -e "\n[2/5] Sensitive Files..."
find "$REPO_PATH" -type f \( -name "*.env*" -o -name "*.pem" -o -name "*.key" -o -name "*secret*" \) \
  -not -path "*/node_modules/*" -not -path "*/.git/*" 2>/dev/null

echo -e "\n[3/5] Injection Patterns..."
grep -rln --include="*.{py,js,ts,php}" \
  -E "(exec|eval|system|subprocess|child_process)\s*\(" \
  "$REPO_PATH" 2>/dev/null | grep -v node_modules | head -10

echo -e "\n[4/5] Hardcoded IPs/URLs..."
grep -rn --include="*.{py,js,ts,java,go,yml,yaml,json}" \
  -E "https?://[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+" \
  "$REPO_PATH" 2>/dev/null | grep -v node_modules | head -10

echo -e "\n[5/5] Docker Security..."
if [ -f "$REPO_PATH/Dockerfile" ]; then
  grep -n -E "(FROM.*:latest|USER root|ARG.*(password|secret|key))" "$REPO_PATH/Dockerfile"
fi

echo -e "\n=== Scan Complete ==="
```

## Tool Installation (if needed)

```bash
# Python security tools
pip install safety pip-audit bandit --break-system-packages

# Run bandit for Python SAST
bandit -r . -f json 2>/dev/null | head -100

# Semgrep for multi-language SAST (if available)
# semgrep --config=auto .
```

## Common Remediation Patterns

**Secrets:** Use environment variables, secrets managers (AWS Secrets Manager, HashiCorp Vault), or encrypted config files. Never commit secrets.

**Dependencies:** Update to patched versions. Use lockfiles. Set up Dependabot/Renovate for automated updates.

**Injection:** Use parameterized queries, input validation, output encoding. Never concatenate user input into commands/queries.

**Crypto:** Use modern algorithms (AES-256-GCM, SHA-256+, bcrypt/argon2 for passwords). Never roll custom crypto.

**Config:** Separate dev/prod configs. Disable debug in production. Use principle of least privilege.
