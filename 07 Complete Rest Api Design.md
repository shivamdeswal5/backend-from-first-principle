# Complete REST API Design

## 1. What is REST? (History & Definition)

In 2000, **Roy Fielding** proposed a standardized architectural style to solve the web's growing scalability crisis — he named it **REST (Representational State Transfer)**.

Breaking down the name:

| Word | Meaning |
|---|---|
| **Representational** | Resources can have multiple representations — JSON for API clients, HTML for browsers |
| **State** | The current condition/attributes of a resource (e.g., items in a shopping cart) |
| **Transfer** | Moving these representations between client and server using standard HTTP methods |

---

## 2. The 6 Constraints of REST Architecture

To be truly "RESTful", a system must follow all six constraints proposed by Fielding:

| # | Constraint | Description |
|---|---|---|
| 1 | **Client-Server** | Strict separation — client owns UI/UX, server owns data and business logic |
| 2 | **Uniform Interface** | Standardized communication across all web components |
| 3 | **Layered System** | Hierarchical layers (load balancers, proxies) — each layer only sees the one directly below it |
| 4 | **Cache** | Responses must explicitly label themselves as cacheable or non-cacheable |
| 5 | **Stateless** | Server retains no memory — every request must carry all information needed to process it |
| 6 | **Code on Demand** *(optional)* | Servers can send executable code (e.g., JavaScript) to extend client functionality |

---

## 3. Anatomy of a RESTful Route

```
https://api.example.com/v1/books/123
  |      |               |   |    |
  |      |               |   |    └─ Resource ID
  |      |               |   └─ Resource (plural noun)
  |      |               └─ API version
  |      └─ API subdomain
  └─ Secure scheme (always HTTPS)
```

### Naming Rules

**Always use plural nouns** — even when fetching a single item:

```
OK   /books
OK   /books/123
OK   /organizations/5/projects
BAD  /book
BAD  /getBook
```

**URL formatting:**

```
OK   /books/harry-potter       (lowercase, hyphen-separated)
BAD  /books/Harry Potter        (spaces not allowed)
BAD  /books/harry_potter        (no underscores)
```

**Hierarchy with `/`:**

```
/organizations/123/projects
       |                |
  org with ID 123       └─ projects belonging to that org
```

---

## 4. HTTP Methods and Idempotency

An operation is **idempotent** if performing it multiple times produces the exact same server-side effect as performing it once.

| Method | Idempotent? | Purpose |
|---|---|---|
| `GET` | Yes | Fetch data — calling 1000 times won't alter server state |
| `PUT` | Yes | Completely replace an entire resource |
| `PATCH` | Yes | Partially update specific fields only |
| `DELETE` | Yes | Remove a resource — subsequent calls return 404 but no further side effects |
| `POST` | No | Create a new resource — 10 calls = 10 distinct records with different IDs |

**PATCH vs PUT:**
- `PATCH` — update only a few fields (e.g., change a user's status)
- `PUT` — replace the entire resource representation

---

## 5. Custom Actions (The Exception to CRUD)

Some actions don't fit neatly into standard CRUD. For example, "cloning" a project or "archiving" an organization triggers complex background tasks beyond a simple DB update.

**Rule:** When an action doesn't map to a standard method, make it a `POST` request and append the action as a **verb at the end** of a specific resource route.

```
POST /projects/123/clone
POST /organizations/5/archive
POST /users/99/suspend
```

---

## 6. Designing Robust List APIs (GET)

When returning lists, an API must handle heavy data loads with three features:

### Pagination

Never send thousands of records at once — send data in pages.

```json
{
  "data": [...],
  "total": 843,
  "page": 2,
  "totalPages": 85
}
```

| Field | Purpose |
|---|---|
| `data` | Array of objects for this page |
| `total` | Absolute count of all items in the database |
| `page` | Current page being viewed |
| `totalPages` | Maximum pages available |

### Sorting

```
GET /books?sortBy=name&sortOrder=ascending
```

### Filtering

```
GET /organizations?status=active&name=org
```

---

## 7. Handling HTTP Status Codes Properly

| Code | When to use |
|---|---|
| `200 OK` | Fetching, updating, or performing custom actions |
| `201 Created` | `POST` request successfully created a new DB entity |
| `204 No Content` | Successful `DELETE` — intentionally empty response body |
| `404 Not Found` | A specific resource ID was requested and does not exist |

### The 404 Rule for List APIs

```
GET /users/999         -> User doesn't exist  ->  404 Not Found    (correct)
GET /users?name=Zack   -> No matches found    ->  200 OK with []   (correct)
GET /users?name=Zack   -> No matches found    ->  404 Not Found    (wrong)
```

> Never return `404` for an empty list. An empty result is still a valid, successful response.

---

## 8. Golden Rules & Best Practices

### Extract Nouns from UI

Before writing any code, study the frontend wireframes (Figma). Identify the "nouns" users interact with — these become your core API resources.

```
Figma shows: projects, tasks, users, comments
          -> /projects, /tasks, /users, /comments
```

### Implement Sane Defaults

Your server should never crash because a client forgot an optional field.

| Missing field | Default |
|---|---|
| Pagination `limit` | `10` |
| Sort order | `created_at` descending |
| New org `status` | `active` |

### Total Consistency

| Do | Avoid |
|---|---|
| Always `camelCase` in JSON | Mixing `camelCase` and `snake_case` |
| Use `description` everywhere | Abbreviating to `desc` in some endpoints |
| Same error format across all routes | Different error shapes per endpoint |

> Inconsistencies force other developers to guess — guessing causes bugs.

### Interactive Documentation

Always generate interactive API docs using **Swagger / OpenAPI**. It serves as:
- Documentation for frontend developers
- A live testing playground for your team
