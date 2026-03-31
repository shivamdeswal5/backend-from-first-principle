# Authentication and Authorization for Backend Engineers

## 1. Core Definitions

| Term | Question it Answers | Definition |
|---|---|---|
| **Authentication** | *"Who are you?"* | The mechanism used to assign an identity to a subject |
| **Authorization** | *"What can you do?"* | The process of determining a user's permissions and capabilities |

---

## 2. The Evolution of Authentication

| Era | Mechanism | Paradigm |
|---|---|---|
| **Pre-Industrial** | Handshakes, vouching | Implicit human trust |
| **Medieval** | Wax seals | *Something you have* (possession) — first bypass attacks via forgery |
| **Industrial Revolution** | Telegraph passphrases | *Something you know* |
| **1960s (Mainframes)** | Passwords (MIT) | Plain text → hashing invented after a password file was accidentally printed |
| **1970s** | Diffie-Hellman, PKI, Kerberos | Asymmetric cryptography, ticket-based auth |
| **1990s** | MFA | Combined: *know + have + are* |
| **Future** | Post-Quantum Cryptography, decentralized identity, behavioral biometrics | — |

### The Three Factors of MFA
- **Something you know** — password
- **Something you have** — OTP / hardware key
- **Something you are** — biometrics

---

## 3. Three Core Components of Modern Authentication

### A. Sessions

**Problem:** HTTP is stateless — it remembers nothing about past requests. Apps like e-commerce sites need stateful memory.

**Solution:**
```
User logs in
    → Server creates a unique Session ID
    → User data stored alongside Session ID in Redis (fast, in-memory)
    → Session ID sent to client
    → Client includes Session ID in all future requests
    → Server looks up Session ID in Redis to identify the user
```

---

### B. JWTs (JSON Web Tokens)

**Problem:** At global scale, storing and synchronizing millions of sessions across distributed servers causes latency and high storage costs.

**Solution:** JWTs are a *stateless* mechanism that offloads storage entirely from the server.

**Structure** — three base64-encoded parts separated by `.`:

| Part | Contents |
|---|---|
| **Header** | Metadata — signing algorithm used |
| **Payload** | Claims — user ID (`sub`), issued-at time (`iat`), roles |
| **Signature** | Verified using a secret key held only by the server — ensures the token hasn't been tampered with |

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjMiLCJyb2xlIjoiYWRtaW4ifQ.s5cSwqD...
      Header                         Payload                        Signature
```

---

### C. Cookies

A mechanism allowing servers to store small pieces of information directly in the user's browser.

**Workflow:**
```
Server sets an HTTP-only cookie containing the Auth Token
    → HTTP-only flag: JavaScript cannot access it (XSS protection)
    → Browser automatically attaches the cookie to every
      subsequent request sent to that specific server
```

---

## 4. Major Types of Authentication

### 1. Stateful Authentication
```
Login → Server stores session in Redis → Sends Session ID via cookie
      → Client sends cookie on next request
      → Server looks up ID in Redis
```

| Pros | Cons |
|---|---|
| Centralized control | Limited scalability |
| Easy token revocation | Higher operational complexity |

> Best for: standard web applications.

---

### 2. Stateless Authentication
```
Login → Server signs a JWT with a secret key → Client stores JWT
      → Client sends JWT in Authorization header on next request
      → Server cryptographically verifies token (no DB lookup needed)
```

| Pros | Cons |
|---|---|
| Highly scalable | Revoking access before expiry is extremely complex |
| Portable across services | Changing the secret key logs out all users |

---

### 3. API Key-Based Authentication

**Purpose:** Machine-to-machine communication (e.g., your backend calling OpenAI's API).

```
User generates a cryptographically random string via UI
    → Server includes this key in automated requests
    → Bypasses human login flows entirely
```

---

### 4. OAuth 2.0 & OpenID Connect (OIDC)

**The Delegation Problem:** Users historically shared passwords so one app could access resources on another (e.g., a travel app scanning Gmail). This made revocation impossible and was a massive security risk.

**OAuth 2.0** — solves *authorization*:
- Client app requests a token with **limited permissions** (e.g., read-only contacts) from an Authorization Server.
- The target app never sees the user's password.

**OpenID Connect (OIDC)** — solves *authentication* (built on top of OAuth 2.0):
- Introduced the **ID Token** (a JWT) that securely shares user profile data — email, name, etc.
- Powers **"Sign in with Google"** and similar flows today.

```
OAuth 2.0  =  "Can this app access your Google Contacts?"  →  Authorization
OIDC       =  "Who is this user?"                          →  Authentication
```

---

## 5. Authorization: Role-Based Access Control (RBAC)

Not all users should have the same capabilities (e.g., standard users vs. admins accessing deleted files).

**How it works:**
- Users are assigned **roles** — `User`, `Admin`, `Moderator`
- Each role maps to strict **resource permissions** — `Read`, `Write`, `Delete`
- During the request cycle, the server deduces the role from the user's token
- Unauthorized action → server rejects with `403 Forbidden`

```
Request arrives
    → Server reads role from token
    → Checks role against required permission for that route
    → ✅ Allowed  or  ❌ 403 Forbidden
```

---

## 6. Critical Security Best Practices

### Use Generic Error Messages
Never expose specific failure reasons to the client.

| ❌ Don't | ✅ Do |
|---|---|
| `"User not found"` | `"Authentication failed"` |
| `"Incorrect password"` | `"Authentication failed"` |

Specific messages let attackers deduce valid usernames and narrow their attack surface.

---

### Prevent Timing Attacks

Attackers can measure server response times to infer valid usernames:

```
Invalid username  →  Server fails instantly (no hash computation)
Valid username    →  Server takes longer (runs password hashing algorithm)
        ↑
  Attacker detects the difference and confirms the username exists
```

**Solution:** Equalize all response times using:
- **Constant-time comparison operations**
- **Simulated fake delay** — e.g., always wait 200ms before responding, regardless of outcome
