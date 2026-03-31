# What is Routing in Backend? How Requests Find Their Way Home

## 1. What is Routing?

Routing is the process of mapping a combination of an **HTTP method** and a **URL path** to a specific server-side **Handler** — a set of instructions or business logic.

- **HTTP Methods** express the *what* (the intent) — e.g., fetching or adding data.
- **Routes** express the *where* — the specific resource or destination to apply that action to.

**Uniqueness:** The server concatenates the HTTP method and the route to form a unique key. For example, `GET /api/books` and `POST /api/books` trigger completely different logic without clashing.

---

## 2. Types of Routes

### Static Routes
Constant strings with no variable parameters. They always point to the same general resource.

```
/api/books
```

### Dynamic Routes
Include variable slots within the URL that the server extracts as data. Denoted by a colon `:` in most frameworks (Node.js, Python, Go).

```
/api/users/:id
```

If a client requests `/api/users/123`, the server extracts `123` as the ID to fetch that specific user's data.

---

## 3. Path Parameters vs. Query Parameters

| Type | Placement | Purpose | Example |
|---|---|---|---|
| **Path Parameters** | Directly inside the route path | Identify a unique resource (semantic meaning) | `/api/users/123` |
| **Query Parameters** | After a `?` at the end of the URL | Send key-value metadata (no request body on `GET`) | `/api/search?query=value` |

**Common query parameter use cases:**
- Pagination — `page=2&limit=20`
- Filtering user-defined values
- Sorting order — `sort=asc` / `sort=desc`

---

## 4. Nested Routing

Nested routing is a standard REST API practice for expressing a **hierarchy between resources**, making routes highly readable and semantic.

```
GET /api/users                      → All users
GET /api/users/123                  → Specific user (ID 123)
GET /api/users/123/posts            → All posts by user 123
GET /api/users/123/posts/456        → Specific post (ID 456) by user 123
```

Each level deeper narrows the scope to a more specific resource.

---

## 5. Route Versioning and Deprecation

**The Problem:** If you change the response format on a live route (e.g., renaming the key `name` to `title`), you break any frontend (iOS app, React app) currently relying on it.

**The Solution — Versioning:** Add version numbers directly to the route path.

```
/api/v1/products    ← old format, still active
/api/v2/products    ← new format
```

**Deprecation:** Both versions run simultaneously, giving frontend engineers a safe migration window to update their code to `v2` before `v1` is officially removed.

---

## 6. Catch-All Routes

A catch-all route acts as a **safety net** for invalid or unmatched requests.

- Placed at the very end of the server's routing logic using a wildcard, typically `/*`.
- If a request passes through all route-matching logic without finding a match, it hits the catch-all.
- Instead of returning a broken or null response, the handler cleanly returns a `404 Route Not Found` message to the client.

```
/*   →   404 Route Not Found
```
