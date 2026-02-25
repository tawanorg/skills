# Security Checklist

OWASP-aligned security guidance for AI coding agents.

## Input Validation

### Rules

- Validate on the server, not just the client
- Whitelist allowed values, don't blacklist bad ones
- Validate type, length, format, and range
- Reject invalid input; don't try to sanitize

### Examples

```typescript
// BAD: No validation
function processAmount(amount: string) {
  return parseFloat(amount);
}

// GOOD: Explicit validation
function processAmount(input: unknown): number {
  if (typeof input !== 'string' && typeof input !== 'number') {
    throw new ValidationError('Amount must be a string or number');
  }

  const amount = Number(input);

  if (Number.isNaN(amount)) {
    throw new ValidationError('Amount must be a valid number');
  }

  if (amount < 0 || amount > 1_000_000) {
    throw new ValidationError('Amount must be between 0 and 1,000,000');
  }

  return amount;
}
```

## SQL Injection Prevention

### Always Use Parameterized Queries

```typescript
// BAD: String concatenation
const query = `SELECT * FROM users WHERE id = '${userId}'`;

// GOOD: Parameterized query
const query = 'SELECT * FROM users WHERE id = $1';
const result = await db.query(query, [userId]);

// GOOD: ORM with parameter binding
const user = await User.findOne({ where: { id: userId } });
```

### Dynamic Queries

```typescript
// BAD: Dynamic column name
const query = `SELECT * FROM users ORDER BY ${sortColumn}`;

// GOOD: Whitelist allowed columns
const ALLOWED_SORT_COLUMNS = ['name', 'created_at', 'email'] as const;

function buildQuery(sortColumn: string) {
  if (!ALLOWED_SORT_COLUMNS.includes(sortColumn as any)) {
    throw new ValidationError('Invalid sort column');
  }
  return `SELECT * FROM users ORDER BY ${sortColumn}`;
}
```

## Secrets Management

### Rules

1. Never commit secrets to version control
2. Use environment variables or secret managers
3. Rotate secrets regularly
4. Use different secrets per environment

### Patterns

```typescript
// BAD: Hardcoded secret
const apiKey = 'sk-1234567890abcdef';

// GOOD: Environment variable
const apiKey = process.env.API_KEY;
if (!apiKey) {
  throw new Error('API_KEY environment variable is required');
}

// GOOD: Secret manager
const apiKey = await secretManager.getSecret('api-key');
```

### Git Prevention

```gitignore
# .gitignore
.env
.env.*
*.pem
*.key
secrets/
```

## Authentication & Authorization

### Password Handling

```typescript
// GOOD: Use bcrypt or argon2
import { hash, verify } from 'argon2';

async function hashPassword(password: string): Promise<string> {
  return hash(password);
}

async function verifyPassword(password: string, hash: string): Promise<boolean> {
  return verify(hash, password);
}
```

### Authorization Checks

```typescript
// BAD: No authorization check
async function deleteUser(userId: string) {
  await db.users.delete(userId);
}

// GOOD: Verify permissions
async function deleteUser(userId: string, requestingUser: User) {
  if (requestingUser.id !== userId && !requestingUser.isAdmin) {
    throw new ForbiddenError('Cannot delete other users');
  }
  await db.users.delete(userId);
}
```

## XSS Prevention

### Output Encoding

```typescript
// BAD: Direct HTML insertion
element.innerHTML = userInput;

// GOOD: Text content
element.textContent = userInput;

// GOOD: Sanitization library
import DOMPurify from 'dompurify';
element.innerHTML = DOMPurify.sanitize(userInput);
```

### Content Security Policy

```typescript
// Set CSP headers
res.setHeader(
  'Content-Security-Policy',
  "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'"
);
```

## Path Traversal Prevention

```typescript
import path from 'path';

// BAD: Direct path usage
function readFile(filename: string) {
  return fs.readFileSync(`./uploads/${filename}`);
}

// GOOD: Validate path
function readFile(filename: string) {
  const uploadsDir = path.resolve('./uploads');
  const filePath = path.resolve(uploadsDir, filename);

  // Ensure file is within uploads directory
  if (!filePath.startsWith(uploadsDir)) {
    throw new SecurityError('Invalid file path');
  }

  return fs.readFileSync(filePath);
}
```

## Quick Security Checklist

### Before Every PR

- [ ] No secrets in code
- [ ] All user input validated
- [ ] SQL uses parameterized queries
- [ ] File paths are validated
- [ ] Sensitive data not logged
- [ ] Authorization checks in place
- [ ] HTTPS for external requests

### Periodic Review

- [ ] Dependencies updated
- [ ] Secrets rotated
- [ ] Access logs reviewed
- [ ] Error handling doesn't leak info
