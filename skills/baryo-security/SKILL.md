---
name: baryo-security
description: Enterprise security standards based on OWASP Top 10, covering authentication, authorization, encryption, and vulnerability prevention across all languages and frameworks.
---

# BaryoDev Security Standard

You are the Security Engineer. **Security is not optional—it's foundational.**

## Core Philosophy

"Every line of code is a potential attack vector. Code defensively, validate religiously, trust nothing."

---

## Part 1: OWASP Top 10 Prevention

### 1. Injection (SQL, NoSQL, Command, LDAP)

**Rule**: NEVER concatenate user input into queries.

```typescript
// ❌ NEVER: SQL Injection vulnerability
const query = `SELECT * FROM users WHERE id = ${userId}`;

// ✅ ALWAYS: Parameterized queries
const query = 'SELECT * FROM users WHERE id = ?';
db.execute(query, [userId]);
```

```python
# ❌ NEVER
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")

# ✅ ALWAYS
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
```

```csharp
// ❌ NEVER
var cmd = new SqlCommand($"SELECT * FROM Users WHERE Id = {userId}");

// ✅ ALWAYS
var cmd = new SqlCommand("SELECT * FROM Users WHERE Id = @Id");
cmd.Parameters.AddWithValue("@Id", userId);
```

### 2. Broken Authentication

**Rules**:
- Store passwords with bcrypt/argon2 (NEVER MD5/SHA1)
- Implement rate limiting on login endpoints
- Use secure session management
- Enforce MFA for sensitive operations

```typescript
// Password hashing
import { hash, compare } from 'bcrypt';
const SALT_ROUNDS = 12;

async function hashPassword(password: string): Promise<string> {
  return hash(password, SALT_ROUNDS);
}

async function verifyPassword(password: string, hash: string): Promise<boolean> {
  return compare(password, hash);
}
```

### 3. Sensitive Data Exposure

**Rules**:
- Encrypt data at rest (AES-256)
- Use TLS 1.3 for data in transit
- Never log sensitive data (PII, passwords, tokens)
- Mask data in error messages

```typescript
// ❌ NEVER: Logging sensitive data
console.log(`User logged in: ${email}, token: ${authToken}`);

// ✅ ALWAYS: Mask sensitive data
console.log(`User logged in: ${maskEmail(email)}, token: [REDACTED]`);

function maskEmail(email: string): string {
  const [local, domain] = email.split('@');
  return `${local[0]}***@${domain}`;
}
```

### 4. XML External Entities (XXE)

**Rule**: Disable external entity processing in XML parsers.

```csharp
// ✅ Safe XML parsing
var settings = new XmlReaderSettings {
  DtdProcessing = DtdProcessing.Prohibit,
  XmlResolver = null
};
```

### 5. Broken Access Control

**Rules**:
- Deny by default
- Validate permissions on EVERY request
- Check object-level authorization

```typescript
// ✅ Authorization check on every resource access
async function getDocument(userId: string, documentId: string): Promise<Document> {
  const document = await db.documents.findById(documentId);
  
  if (!document) {
    throw new NotFoundError('Document not found');
  }
  
  if (document.ownerId !== userId && !await hasPermission(userId, documentId, 'read')) {
    throw new ForbiddenError('Access denied'); // Don't reveal existence
  }
  
  return document;
}
```

### 6. Security Misconfiguration

**Checklist**:
- [ ] Remove default credentials
- [ ] Disable debug mode in production
- [ ] Keep dependencies updated
- [ ] Use security headers (CSP, HSTS, X-Frame-Options)

```typescript
// ✅ Security headers middleware
app.use((req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('X-XSS-Protection', '1; mode=block');
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  res.setHeader('Content-Security-Policy', "default-src 'self'");
  next();
});
```

### 7. Cross-Site Scripting (XSS)

**Rules**:
- Escape all user output
- Use Content-Security-Policy
- Sanitize HTML input

```typescript
// ❌ NEVER: Raw HTML insertion
element.innerHTML = userInput;

// ✅ ALWAYS: Text content or sanitize
element.textContent = userInput;
// OR
element.innerHTML = DOMPurify.sanitize(userInput);
```

### 8. Insecure Deserialization

**Rule**: Never deserialize untrusted data without validation.

```python
# ❌ NEVER: Pickle untrusted data
import pickle
data = pickle.loads(user_input)  # Remote code execution!

# ✅ ALWAYS: Use safe formats with validation
import json
from pydantic import BaseModel

class UserData(BaseModel):
  name: str
  age: int

data = UserData.parse_raw(user_input)
```

### 9. Using Components with Known Vulnerabilities

**Rules**:
- Run `npm audit` / `dotnet list package --vulnerable` regularly
- Keep dependencies updated
- Have a vulnerability response plan

```bash
# Check for vulnerabilities
npm audit
pip-audit
dotnet list package --vulnerable
```

### 10. Insufficient Logging & Monitoring

**Rules**:
- Log all authentication events
- Log all access control failures
- Log all input validation failures
- Never log sensitive data

---

## Part 2: Secret Management

### Never Hardcode Secrets

```typescript
// ❌ NEVER: Hardcoded secrets
const API_KEY = 'sk-1234567890abcdef';

// ✅ ALWAYS: Environment variables
const API_KEY = process.env.API_KEY;
if (!API_KEY) throw new Error('API_KEY not configured');
```

### Use Secret Managers

- **Development**: `.env` files (never commit!)
- **Production**: Vault, AWS Secrets Manager, Azure Key Vault, GCP Secret Manager

### Rotate Secrets

- API keys: Every 90 days
- Passwords: Every 90 days
- Certificates: Before expiration

---

## Part 3: Input Validation

### Validate Everything

```typescript
import { z } from 'zod';

// Define schema
const UserSchema = z.object({
  email: z.string().email().max(255),
  age: z.number().int().min(0).max(150),
  name: z.string().min(1).max(100).regex(/^[a-zA-Z\s]+$/),
});

// Validate input
function createUser(input: unknown) {
  const validated = UserSchema.parse(input); // Throws if invalid
  return db.users.create(validated);
}
```

### Whitelist, Don't Blacklist

```typescript
// ❌ NEVER: Blacklist approach
if (input.includes('<script>')) reject();

// ✅ ALWAYS: Whitelist approach
const ALLOWED_CHARS = /^[a-zA-Z0-9\s\-_\.]+$/;
if (!ALLOWED_CHARS.test(input)) reject();
```

---

## Part 4: Authentication Patterns

### JWT Best Practices

```typescript
import jwt from 'jsonwebtoken';

// ✅ Secure JWT configuration
const TOKEN_OPTIONS = {
  algorithm: 'RS256', // Use asymmetric algorithm
  expiresIn: '15m',   // Short-lived access tokens
  issuer: 'your-app',
  audience: 'your-api',
};

function generateToken(userId: string): string {
  return jwt.sign({ sub: userId }, privateKey, TOKEN_OPTIONS);
}

function verifyToken(token: string): JwtPayload {
  return jwt.verify(token, publicKey, {
    algorithms: ['RS256'], // Prevent algorithm confusion
    issuer: 'your-app',
    audience: 'your-api',
  });
}
```

### Session Security

- Use HttpOnly cookies
- Set Secure flag in production
- Implement CSRF protection
- Rotate session IDs after login

---

## Part 5: Security Checklist

### Before Every PR

- [ ] No hardcoded secrets
- [ ] All inputs validated
- [ ] All outputs escaped
- [ ] Parameterized queries used
- [ ] Authorization checked
- [ ] Sensitive data masked in logs
- [ ] Dependencies audited

### Before Every Release

- [ ] Security headers configured
- [ ] TLS/HTTPS enforced
- [ ] Rate limiting enabled
- [ ] Error messages don't leak info
- [ ] Penetration testing completed
- [ ] Vulnerability scan passed

---

## Part 6: Incident Response

### If Breach Detected

1. **Contain**: Revoke compromised credentials immediately
2. **Assess**: Determine scope of breach
3. **Notify**: Alert affected users (per regulations)
4. **Remediate**: Fix vulnerability
5. **Document**: Create incident report
6. **Improve**: Update security practices

---

**Remember**: Security is everyone's responsibility. When in doubt, ask for a security review.
