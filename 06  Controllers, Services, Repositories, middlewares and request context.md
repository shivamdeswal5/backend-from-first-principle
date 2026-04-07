# Controllers, Services, Repositories, Middlewares, and Request Context

## 1. The Backend Architecture Pattern

Separating server code into distinct layers is not strictly mandatory, but it is a vital design pattern. The separation of concerns makes the codebase:

- **Scalable** — layers can grow independently
- **Maintainable** — easier to locate and fix issues
- **Debuggable** — problems are isolated to a specific layer
- **Extensible** — new features slot in without touching unrelated code

```
Incoming HTTP Request
        │
        ▼
┌───────────────────┐
│   Middleware(s)   │  ← Security, Auth, Rate Limiting, Logging
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│    Controller     │  ← Extract, Deserialize, Validate, Transform, Respond
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│     Service       │  ← Business logic, orchestration (no HTTP knowledge)
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│    Repository     │  ← Database queries only (Postgres, Redis, etc.)
└───────────────────┘
```

---

## 2. The Controller (Handler) Layer

The Controller is the **entry point** for a route after the routing algorithm finds a match. Its job is managing the flow of data from the client to the server and back.

```
Request arrives
      │
      ▼
1. Data Extraction
   └─ Pull query params, path params, JSON body from the request object

2. Binding (Deserialization)
   └─ Convert JSON string → native type (Go struct, Python dict, JS object)
   └─ Fails? → return 400 Bad Request immediately

3. Validation
   └─ Check format, required fields, prevent malicious payloads

4. Transformation
   └─ Inject defaults for missing optional fields
      e.g., no "sort" param provided → default to sort by date

5. Delegation
   └─ Pass clean, validated data to the Service Layer

6. Send Response
   └─ Receive result from Service
   └─ Choose appropriate status code (200, 201, 400, 500...)
   └─ Send final response to client
```

> **One rule:** The Controller owns HTTP. Everything HTTP-related starts and ends here.

---

## 3. The Service Layer

The Service Layer is where **actual business logic and core processing** happen.

**Key principle — Complete HTTP Isolation:**
- No request/response objects
- No status codes
- No validation logic

The Service layer is a pure function: **data in → process → result out**.

**What a single service method can orchestrate:**
- Multiple repository calls
- Merging different data sets
- Sending emails
- Making external API calls

```
// Controller calls:
result = UserService.createUser(cleanData)

// Service does:
existingUser = UserRepo.findByEmail(email)
if existingUser → throw conflict error
hashedPassword = hash(password)
newUser = UserRepo.insert({ email, hashedPassword })
EmailService.sendWelcomeEmail(email)
return newUser
```

---

## 4. The Repository (Database) Layer

The Repository layer has **one job only** — database operations.

- Takes data from the Service layer
- Constructs the specific DB query (insert, filter, sort)
- Returns raw results back to the Service layer

**Single Responsibility Principle in practice:**

| ❌ Wrong | ✅ Right |
|---|---|
| One method with `if/else` returning either one book or all books | Two separate methods: `findById()` and `findAll()` |
| Complex conditional logic inside a repo method | One method, one query, one return type |

---

## 5. Middlewares

Middlewares are functions that execute **between** the moment the server receives a request and the moment it hits the Controller.

**Why use them?** To avoid duplicating logic across hundreds of endpoints. Common operations (auth, logging, security) are centralized in one place.

### The `next()` Function

Every middleware receives three things: `request`, `response`, and `next`.

```
Middleware receives request
        │
        ├── calls next()  →  passes execution to the next middleware/controller
        │
        └── doesn't call next()  →  returns response immediately, terminates request
```

> **Execution order matters** — middlewares run sequentially, so their arrangement is critical.

### Common Middlewares (in typical order)

| Middleware | Position | What it does |
|---|---|---|
| **CORS & Security Headers** | Very first | Checks request origin — blocks unauthorized domains instantly |
| **Rate Limiting** | Early | Checks IP — returns `429 Too Many Requests` if spamming |
| **Authentication** | Early | Verifies token — `401 Unauthorized` if invalid; extracts user data if valid |
| **Logging** | Throughout | Records URL, method, params to terminal for debugging |
| **Global Error Handler** | Very last | Catches unstructured errors from controllers/services — formats them into clean responses |

---

## 6. Request Context

Request context is a concept used in web development. It refers to all the information related to a single user request while the server is handling it.

Think of it like a “container” that holds everything about that request until it’s finished.

Request Context is a **scoped key-value store** strictly limited to a single HTTP request's lifetime.

**Why it exists:** A request passes through many isolated function boundaries (middlewares → controller). Context lets them share state without tightly coupling the code.

### Common Use Cases

**1. Passing Authentication Data**
```
Auth Middleware
  └─ Validates token
  └─ Extracts user_id and role → saves to Context

Controller
  └─ Reads user_id from Context (not from client JSON payload)
```
> ⚠️ Never trust a `user_id` sent by the client in the request body — malicious users can spoof it. Always read identity from Context.

**2. Request Tracing**
```
Early Middleware
  └─ Generates a unique UUID → saves to Context

All subsequent logs, microservice calls, DB queries
  └─ Attach the same UUID everywhere

Result: engineers can trace the exact path of any bug across the entire system
```

**3. Cancellations & Timeouts**
- Context stores abort signals and timeout deadlines
- Prevents external service calls from hanging indefinitely
