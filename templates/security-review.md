---
name: Security Review Checklist
use_case: Identify security vulnerabilities in code before it ships
variables:
  - CODE: the code to review
  - CONTEXT: what this code does (e.g., "handles user authentication", "processes payment webhooks", "renders user-supplied HTML")
  - LANGUAGE_AND_FRAMEWORK: e.g., "Node.js + Express", "Python + Django", "Go + net/http"
  - DATA_SENSITIVITY: what sensitive data this code touches (e.g., "PII", "payment cards", "none", "session tokens")
---

# Security Review Checklist Prompt

## Prompt

```
Perform a security-focused code review. You are looking for vulnerabilities that could be exploited by a malicious actor. Be specific — cite exact lines, explain the attack vector, and provide the fix.

**Context:** {{CONTEXT}}
**Stack:** {{LANGUAGE_AND_FRAMEWORK}}
**Sensitive data involved:** {{DATA_SENSITIVITY}}

**Code:**
{{CODE}}

Review against these categories (skip categories that are clearly not applicable and say why):

### Injection (SQLi, XSS, command injection, LDAP injection)
Does any user-controlled input reach a query, shell command, or HTML output without proper escaping/parameterization?

### Authentication and Authorization
Is identity verified correctly? Are authorization checks present AND in the right place (server-side, not just client-side)?

### Sensitive Data Exposure
Are secrets, keys, or PII logged, returned in API responses, stored insecurely, or transmitted over plain HTTP?

### Input Validation
Is input validated for type, length, range, and format? Are there paths where malformed input causes unexpected behavior?

### Cryptography
Are secure, modern algorithms used? Are keys managed properly? Is there any use of MD5/SHA1 for security purposes?

### Security Misconfiguration
CORS too permissive? Debug mode on? Default credentials? Overly verbose error messages?

### Dependency Risk
Any known-vulnerable libraries used in this code?

### Business Logic Flaws
Any assumptions an attacker could violate? (e.g., "assumes user_id in the JWT matches the resource being accessed")

For each finding: severity (Critical/High/Medium/Low), line number, attack scenario in one sentence, and the fix.
```

## Worked Example

**Context:** User registration endpoint that creates accounts and sends verification emails

**What a good response looks like:** Finds that the email field is interpolated directly into an HTML template (XSS, High); the username uniqueness check has a TOCTOU race condition (Medium); the verification token is 6 digits (brute-forceable, High); password is logged at debug level (Medium). Each finding has the exact line and a one-line fix.

## Tuning Notes

- For threat modeling before writing code: replace CODE with a design description and ask the model to identify threats in each category. Faster than reviewing code that doesn't exist yet.
- Add: "Assume an attacker has read access to this codebase" — this shifts the model toward realistic attack scenarios.
- For compliance contexts (PCI-DSS, HIPAA, SOC2): add the relevant standard to CONTEXT and ask the model to flag any explicit standard violations.
- False positives happen. For each finding, ask: "Is there a mitigating control I haven't shown you that would address this?" before filing a ticket.
