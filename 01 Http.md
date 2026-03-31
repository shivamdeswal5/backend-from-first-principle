# Understanding HTTP for Backend Engineers

## 1. Core Principles of HTTP

HTTP (Hypertext Transfer Protocol) is an application-layer protocol (Layer 7 in the OSI model) used by clients and servers to communicate. It is built on two fundamental ideas:

**Statelessness:** The server retains no memory of past interactions. Every request is entirely self-contained and must include all the necessary information (like authentication tokens or cookies) for the server to process it.

- **Benefits:** Simplifies server architecture and improves scalability, because a single server doesn't need to track user sessions, and a server crash won't destroy a client's state.

**Client-Server Model:** Communication is always initiated by the client (e.g., a web browser) to request resources or actions, and the server waits for these requests to process and respond.

---

## 2. Transport Protocol & HTTP Versions

HTTP relies on a reliable, connection-based transport protocol, almost universally **TCP (Transmission Control Protocol)**. Over the years, HTTP has evolved to improve how these TCP connections are handled:

| Version | Key Feature |
|---|---|
| **HTTP 1.0** | Opened a new TCP connection for every single request/response — highly inefficient |
| **HTTP 1.1** | Introduced persistent connections (`keep-alive`) as the default, allowing multiple requests over one reused connection |
| **HTTP 2.0** | Multiplexing, binary framing, header compression, and server push — all on one connection |
| **HTTP 3.0** | Replaced TCP with QUIC (built over UDP) for faster connections, better packet loss handling, and no head-of-line blocking |

---

## 3. Anatomy of HTTP Messages

Client-server communication happens via structured text messages.

**Request Message (Client → Server)**
```
[Request Method]  [Resource URL]  [HTTP Version]
Host: [domain]
[Headers]

[Optional Request Body]
```

**Response Message (Server → Client)**
```
[HTTP Version]  [Status Code]  [Status Value]
[Headers]

[Response Body]
```

---

## 4. HTTP Headers

Headers are key-value pairs that act as metadata for the package being transmitted. They make the system highly extensible and act as a "remote control" to dictate server behavior.

| Type | Purpose | Examples |
|---|---|---|
| **Request Headers** | Provide client context | `User-Agent`, `Authorization` |
| **General Headers** | Apply to both requests and responses | `Date`, `Connection`, `Cache-Control` |
| **Representation Headers** | Describe the message body | `Content-Type`, `Content-Length`, `Content-Encoding` |
| **Security Headers** | Protect against attacks | `Strict-Transport-Security`, `Content-Security-Policy`, `Set-Cookie` |

---

## 5. HTTP Methods and Idempotency

Methods define the semantic **intent** of the client's request.

| Method | Purpose |
|---|---|
| `GET` | Fetches data without modifying anything |
| `POST` | Submits new data to the server (includes a request body) |
| `PATCH` | Partially updates an existing resource |
| `PUT` | Completely replaces an existing resource |
| `DELETE` | Removes a resource |
| `OPTIONS` | Inquires about server capabilities (used heavily in CORS) |

### Idempotency

**Idempotent methods** can be executed multiple times and yield the exact same result on the server state:
- `GET`, `PUT`, `DELETE`

**Non-idempotent methods** produce different results when called multiple times:
- `POST` — submitting twice creates two separate resources

---

## 6. Cross-Origin Resource Sharing (CORS)

Browsers enforce a **Same-Origin Policy**, blocking web apps from making requests to different domains. CORS is a security mechanism to bypass this safely.

### Simple Requests
Usually `GET` or `POST` with standard headers/content types.

1. Browser automatically adds an `Origin` header.
2. If the server allows the request, it replies with `Access-Control-Allow-Origin` containing the client's domain (or `*`).
3. If the header is missing, the browser blocks the response.

### Pre-flight Requests
Triggered when a request uses a non-simple method (`PUT`/`DELETE`), requires authorization headers, or uses `application/json` content type.

1. Browser fires an `OPTIONS` request asking if the route supports the intended method and headers.
2. Server replies with `204 No Content`, listing allowed origins, methods, headers, and a `max-age` to cache the configuration.
3. If successful, the browser sends the actual original request.

---

## 7. Standardized Status Codes

Status codes are three-digit numbers that act as a universal language to indicate the outcome of a request.

### 1xx — Informational
| Code | Meaning |
|---|---|
| `100 Continue` | Headers received; client can proceed (used for large uploads) |

### 2xx — Success
| Code | Meaning |
|---|---|
| `200 OK` | Successful operation |
| `201 Created` | Resource created (usually follows `POST`) |
| `204 No Content` | Successful, but no body to return (used in `OPTIONS` or `DELETE`) |

### 3xx — Redirection
| Code | Meaning |
|---|---|
| `301 Moved Permanently` | Resource has a new permanent URL |
| `302 Found` | Temporary redirect to a new route |
| `304 Not Modified` | Client should use its locally cached version |

### 4xx — Client Errors
| Code | Meaning |
|---|---|
| `400 Bad Request` | Invalid data format sent by client |
| `401 Unauthorized` | Missing or invalid authentication token |
| `403 Forbidden` | Authenticated, but lacks necessary permissions |
| `404 Not Found` | Incorrect URL or deleted resource |
| `405 Method Not Allowed` | Wrong method used for a route |
| `409 Conflict` | Business logic violation (e.g., duplicate username) |
| `429 Too Many Requests` | Client has hit rate limits |

### 5xx — Server Errors
| Code | Meaning |
|---|---|
| `500 Internal Server Error` | An unhandled exception crashed the server |
| `501 Not Implemented` | Feature not yet supported |
| `502 Bad Gateway` | Proxy/load balancer failed to reach upstream server |
| `503 Service Unavailable` | Server down or under maintenance |
| `504 Gateway Timeout` | Upstream server timed out |

---

## 8. HTTP Caching

Caching reuses previously downloaded responses to save bandwidth and load times.

**Initial Request:**
The server responds with the payload plus three headers:
- `Cache-Control` — sets the max cache duration
- `ETag` — a unique hash of the payload
- `Last-Modified` — timestamp of last change

**Subsequent Requests:**
The client sends conditional headers:
- `If-None-Match` — carries the cached `ETag`
- `If-Modified-Since` — carries the cached timestamp

**Server Response:**
- If data is **unchanged** → `304 Not Modified` (empty body; browser uses cache)
- If data **has changed** → `200 OK` with new payload and a new `ETag`

---

## 9. Content Negotiation and Compression

Clients and servers can negotiate the best format to exchange data.

**Client sends preferences via:**
- `Accept` — e.g., `application/json` vs `application/xml`
- `Accept-Language` — e.g., `en` vs `es`
- `Accept-Encoding` — e.g., `gzip`

**Compression:** By negotiating an encoding like `gzip`, a server can drastically compress text responses — for example, shrinking a 26MB JSON payload down to 3.8MB — saving significant network bandwidth.

---

## 10. Handling Large Data Transfers

**Large Client Uploads (Images/Video)**

Standard JSON is unsuitable for binary data. Instead, clients use a `multipart/form-data` request, which breaks the file into chunks separated by a unique string delimiter defined in the `boundary` header.

**Large Server Downloads**

To prevent timeouts, the server streams the file in chunks using:
- `Content-Type: text/event-stream`
- `Connection: keep-alive`

The browser continually appends these chunks until the transfer completes.

---

## 11. Security — SSL/TLS & HTTPS

- **TLS (Transport Layer Security):** The modern, secure replacement for the outdated SSL protocol.
- Encrypts data in transit to prevent **eavesdropping** and **tampering**.
- Uses certificates to verify the server's identity.
- **HTTPS:** Standard HTTP wrapped inside a secure TLS connection.
