# Security and Authentication

A working reference for the security concerns that show up in almost every web system: proving who a caller is, deciding what they may do, moving bytes safely, and storing secrets without handing them to an attacker.

## Authentication vs Authorization

These are different questions and are often handled by different components.

| | Authentication (AuthN) | Authorization (AuthZ) |
|---|---|---|
| Question | "Who are you?" | "Are you allowed to do this?" |
| Proves | Identity (user, service) | Permission on a resource |
| Inputs | Password, token, certificate, MFA | Roles, attributes, policy, ownership |
| Typical failure | Account takeover | Privilege escalation |
| When | Once per session/request to establish identity | On every protected action |

A request can be authenticated but not authorized (logged in, but cannot delete someone else's post). Authorization must be checked **server-side on every endpoint** — never trust the client to hide a button.

## Sessions and Cookies

### Server-side sessions

1. User submits credentials over HTTPS.
2. Server verifies them and creates a **session record** (e.g. `{ sessionId, userId, createdAt, expiresAt }`) in a store (Redis, DB).
3. Server returns an opaque, high-entropy `sessionId` in a `Set-Cookie` header.
4. Browser sends the cookie automatically on each subsequent request; the server looks up the session to recover identity.

The session ID is just a random pointer — the actual state lives server-side, so revocation is trivial (delete the record). This is the main advantage over self-contained tokens.

### Cookie attributes that matter

```
Set-Cookie: sid=9f3a...; HttpOnly; Secure; SameSite=Lax; Path=/; Max-Age=3600
```

| Attribute | Effect | Why |
|---|---|---|
| `HttpOnly` | JS cannot read the cookie (`document.cookie`) | Limits XSS-based token theft |
| `Secure` | Sent only over HTTPS | Prevents interception on plaintext links |
| `SameSite=Lax/Strict` | Restricts cross-site sending | Mitigates CSRF |
| `SameSite=None` | Allows cross-site; requires `Secure` | For legitimate third-party contexts |
| `Path` / `Domain` | Scope of the cookie | Least exposure |
| `Max-Age` / `Expires` | Lifetime | Bounded session window |

### Session fixation

Attack: the attacker plants a known session ID in the victim's browser (e.g. via a link), the victim logs in, and the session — now authenticated — is one the attacker already knows.

Defense: **always issue a brand-new session ID at the moment privilege changes** (login, role elevation). Never accept a session ID supplied by a query string or trust a pre-login ID after authentication.

## Tokens

### JWT structure

A JSON Web Token is three Base64URL segments joined by dots:

```
header.payload.signature
eyJhbGciOiJIUzI1NiJ9 . eyJzdWIiOiIxMjMiLCJleHAiOjE3...} . 4Q2f...
```

- **Header** — `{ "alg": "RS256", "typ": "JWT" }`
- **Payload (claims)** — `sub` (subject), `iss` (issuer), `aud` (audience), `exp` (expiry), `iat` (issued-at), `nbf` (not-before), plus custom claims.
- **Signature** — over `base64(header) + "." + base64(payload)` using the algorithm in the header.

The payload is **encoded, not encrypted** — anyone can read it. The signature only guarantees integrity and authenticity, not confidentiality.

### Signing: HS256 vs RS256

| | HS256 (HMAC-SHA256) | RS256 (RSA-SHA256) |
|---|---|---|
| Keys | One shared secret | Private key signs, public key verifies |
| Verification | Anyone with the secret (= can also forge) | Anyone with the public key (cannot forge) |
| Best for | Single trusted service | Multiple verifiers / federated systems |

Security note: pin the expected algorithm on the verifier. The classic `alg: none` and "RS256→HS256 confusion" attacks come from trusting the token's own header to choose the verification method.

### Expiry, refresh, and revocation

- Keep **access tokens short-lived** (minutes) and pair them with a **refresh token** (long-lived, stored securely, single-use/rotating). The refresh token mints new access tokens without re-prompting for credentials.
- **Revocation is the hard problem.** A signed JWT is valid until it expires; there is no server-side record to delete. Options: short TTLs, a token denylist/`jti` blacklist checked on each request (reintroduces server state), or token versioning tied to the user.
- **Do not put sensitive data in a JWT** (passwords, PII, secrets). It is readable by anyone who holds it, and it cannot be unsent once issued.

### Opaque tokens vs JWT

| | Opaque token | JWT |
|---|---|---|
| Form | Random string, meaningless to client | Self-describing claims |
| Validation | Lookup in token store (introspection) | Verify signature locally |
| Revocation | Easy (delete record) | Hard (wait for expiry) |
| Scaling cost | DB/cache hit per call | Stateless, no lookup |

Choose JWT for stateless, high-throughput verification; choose opaque tokens when instant revocation and minimal client exposure matter.

## Single Sign-On (SSO)

SSO lets one identity provider (IdP) authenticate a user once, and many relying applications trust that assertion. Conceptually: the app redirects an unauthenticated user to the IdP; the IdP authenticates them (possibly reusing an existing IdP session); the IdP returns a signed assertion/token; the app validates it and creates its own local session.

| | SAML 2.0 | OpenID Connect (OIDC) |
|---|---|---|
| Format | XML assertions | JSON / JWT |
| Transport | Browser POST/redirect bindings | Built on OAuth 2.0 + HTTPS |
| Era / fit | Enterprise, legacy B2B | Modern web, mobile, SPAs |
| Complexity | Heavier (XML signing) | Lighter, developer-friendly |

## OAuth 2.0

OAuth 2.0 is a **delegated authorization** framework: it lets an app act on a resource owner's behalf **without** seeing their password.

### Roles

- **Resource owner** — the user who owns the data.
- **Client** — the application requesting access.
- **Authorization server** — authenticates the user and issues tokens.
- **Resource server** — the API that holds the protected resources and accepts access tokens.

### Authorization Code flow with PKCE

The recommended flow for web and mobile apps. PKCE (Proof Key for Code Exchange) binds the code to the client that started the flow, defeating code interception.

```
User-Agent        Client App            Auth Server           Resource Server
   |                  |                       |                      |
   | click "login"    |                       |                      |
   |----------------->| generate code_verifier|                      |
   |                  | + code_challenge      |                      |
   |  redirect w/ challenge, scope, state     |                      |
   |<-----------------|---------------------->|                      |
   | authenticate + consent                   |                      |
   |----------------------------------------->|                      |
   |  redirect back with authorization code   |                      |
   |<-----------------------------------------|                      |
   |  deliver code    |                       |                      |
   |----------------->| code + code_verifier  |                      |
   |                  |---------------------->| verify challenge     |
   |                  |  access + refresh tok |                      |
   |                  |<----------------------|                      |
   |                  |  call API with access token ---------------->|
   |                  |<-------------------------- protected data ---|
```

`state` is an anti-CSRF nonce echoed back to detect forged callbacks. The authorization code is short-lived and single-use.

### Other grants

- **Client credentials** — machine-to-machine, no user present. The client authenticates with its own ID/secret and gets a token for its own resources.
- **Implicit flow (deprecated)** — returned the access token directly in the redirect URL fragment. It leaked tokens into browser history/referrers and had no client authentication. PKCE + authorization code replaces it everywhere, including SPAs.

### OAuth vs OpenID Connect

OAuth 2.0 answers **authorization** ("this app may call the calendar API"). OIDC layers **authentication** on top by adding an **ID token** (a JWT describing the user) and a `/userinfo` endpoint. Using raw OAuth access tokens as proof of *who the user is* is a common and dangerous mistake — use OIDC's ID token for identity.

## Access Control Models

| Model | Decision based on | Example | Strength |
|---|---|---|---|
| **RBAC** | User's role(s) | `editor` can publish | Simple, auditable |
| **ABAC** | Attributes of user/resource/env | `dept==owner.dept AND time<18:00` | Fine-grained, dynamic |
| **ReBAC** | Relationships in a graph | "user is a member of the doc's parent folder" | Models sharing/hierarchies (Google Zanzibar style) |

**Principle of least privilege**: grant the minimum permissions needed, for the minimum time. Default-deny; add scopes deliberately; review and expire grants.

## Transport Security

### TLS handshake (high level)

1. **ClientHello** — client offers TLS versions and cipher suites.
2. **ServerHello + certificate** — server picks parameters and sends its X.509 certificate chain.
3. **Authentication** — client validates the cert against trusted CAs (signature, hostname, expiry, revocation).
4. **Key exchange** — both sides derive a shared symmetric session key (modern TLS 1.3 uses ephemeral Diffie-Hellman for forward secrecy).
5. **Encrypted application data** — all further traffic uses fast symmetric encryption.

HTTPS is simply HTTP over this TLS channel. A **certificate** binds a public key to a domain and is signed by a Certificate Authority the client already trusts.

### mTLS for service-to-service

In mutual TLS, **both** sides present certificates, so the server authenticates the client too. This is the backbone of zero-trust service meshes — services prove identity cryptographically instead of relying on network location.

## Password Handling

- **Never store plaintext, and never "encrypt" passwords** — encryption is reversible. Use a one-way **password hashing** function.
- Use a slow, salted, memory-hard hash: **argon2id** (preferred), **scrypt**, or **bcrypt**. Avoid raw SHA-256/MD5 for passwords — they are too fast and trivially brute-forced.
- **Salt** is a unique random value per password (these algorithms generate and embed it). It defeats rainbow tables and ensures identical passwords hash differently.
- **Pepper** is an optional secret added to all passwords, stored separately from the database (e.g. in a vault/HSM), so a DB dump alone is insufficient.
- **Rate-limit and throttle login** attempts (per account and per IP), add exponential backoff and lockout/CAPTCHA to slow credential stuffing.
- Offer **MFA** (TOTP, WebAuthn/passkeys, push). Prefer phishing-resistant factors (FIDO2) over SMS.

```python
# argon2id verification sketch
from argon2 import PasswordHasher
ph = PasswordHasher()                  # tuned cost params
stored = ph.hash(password)             # salt embedded in output
ph.verify(stored, submitted_password)  # raises on mismatch
```

## API Security Checklist

- **Validate input** server-side: type, length, format, range; reject by default (allowlist).
- **Encode output** for its context (HTML, attribute, JS, URL) to stop injection downstream.
- **AuthN and AuthZ on every endpoint** — including object-level checks (does this user own this record?).
- **TLS everywhere**, HSTS on, no plaintext fallback.
- **Rate limit and quota** per client/key to blunt abuse and DoS.
- **Manage secrets** outside code; inject at runtime; rotate.
- **CORS done right**: explicit allowlist of origins, do not reflect arbitrary `Origin`, avoid `Access-Control-Allow-Origin: *` together with credentials.
- Return generic auth errors; do not leak stack traces or whether a username exists.

## OWASP Top 10 (current themes)

| Risk | One-line defense |
|---|---|
| Broken access control | Enforce server-side authz + object-level ownership checks; default-deny |
| Cryptographic failures | TLS in transit, strong hashing/encryption at rest, no homemade crypto |
| Injection (SQL/NoSQL/cmd) | Parameterized queries; never concatenate untrusted input |
| Insecure design | Threat-model early; secure-by-default patterns |
| Security misconfiguration | Harden defaults, disable debug, patch, minimal surface |
| Vulnerable/outdated components | SCA scanning, pin and update dependencies |
| Identification & auth failures | MFA, strong session/token handling, no fixation |
| Software & data integrity failures | Verify signatures, lock CI/CD, no untrusted deserialization |
| Logging & monitoring failures | Log security events, alert, no sensitive data in logs |
| Server-Side Request Forgery (SSRF) | Validate/allowlist outbound URLs, block internal ranges/metadata IP |

## Common Attacks and Defenses

- **SQL injection** — attacker smuggles SQL via input. Defense: **parameterized queries / prepared statements** (and ORMs that use them), least-privilege DB accounts. Never build queries by string concatenation.
- **Cross-Site Scripting (XSS)** — injected script runs in the victim's browser. Defense: **contextual output encoding**, a strict **Content-Security-Policy**, `HttpOnly` cookies, and frameworks that auto-escape.
- **Cross-Site Request Forgery (CSRF)** — victim's browser is tricked into a state-changing request using its cookies. Defense: **anti-CSRF tokens** (synchronizer/double-submit) and `SameSite` cookies; require re-auth for sensitive actions.
- **SSRF** — server is coaxed into fetching attacker-chosen URLs (e.g. cloud metadata at `169.254.169.254`). Defense: allowlist destinations, resolve and block private/link-local ranges, drop redirects, no raw user URLs to internal services.
- **Secrets in code** — keys committed to git. Defense: secret scanning in CI/pre-commit, and if leaked, **rotate immediately** (deletion does not undo exposure).

## Secrets Management

- Store secrets in a dedicated **vault** (HashiCorp Vault, AWS Secrets Manager, GCP Secret Manager, KMS), not in source or images.
- **Inject at runtime** via environment or a sidecar/agent; keep them out of logs, error messages, and client bundles.
- **Rotate** regularly and on suspicion of compromise; prefer short-lived dynamic credentials over static ones.
- **Never commit secrets.** Use `.gitignore`, pre-commit scanners, and treat any committed secret as already burned — revoke and replace it.
