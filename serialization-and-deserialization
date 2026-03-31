# Serialization and Deserialization for Backend Engineers

## 1. The Core Problem: Language Barriers in Networking

A typical web application consists of a client (e.g., a JavaScript frontend) and a server (e.g., a Rust backend). These languages handle data very differently:

- **JavaScript** — dynamic, not compiled, loosely typed
- **Rust** — compiled, extremely strict data types

When a JavaScript client sends an internal data object over the internet to a Rust server, the Rust server cannot natively understand or parse JavaScript data structures. To communicate effectively, they need a **universal language**.

---

## 2. What is Serialization and Deserialization?

Both the client and server agree on a single, common standard format for transmitting data.

| Term | Definition |
|---|---|
| **Serialization** | Converting native data (e.g., a JS object or Rust struct) into a common standard format *before* sending over the network |
| **Deserialization** | The reverse — parsing the received standard format back into the machine's own native data type to perform business logic |

**Goal:** Make data transmission completely **language-agnostic** and **domain-agnostic** — any machine can talk to any other machine regardless of underlying technology.

---

## 3. Types of Serialization Standards

### Text-Based Formats
Human-readable. Easier to debug and inspect.

- JSON, YAML, XML

### Binary Formats
Compiled into raw binary for highly efficient, compact transmission.

- Protobuf, Avro

---

## 4. Deep Dive into JSON (The Industry Standard)

For traditional HTTP REST API communication, **JSON (JavaScript Object Notation)** is the most popular serialization standard — used roughly **80% of the time**.

**Use cases beyond HTTP:**
- Application logging
- Configuration files

**Key characteristics:**
- Not limited to JavaScript despite the name
- Entirely human-readable

### Strict Syntax Rules

- Must start with `{` and end with `}`
- All **keys** must be strings wrapped in double quotes — `"name":`
- **Values** are limited to: strings, numbers, booleans, arrays, or nested JSON objects

```json
{
  "id": 123,
  "name": "Alice",
  "active": true,
  "scores": [98, 87, 95],
  "address": {
    "city": "Mumbai"
  }
}
```

---

## 5. The Backend Engineer's Mental Model (OSI Layers)

When data travels over the internet, it passes through many complex network layers (the OSI model). As a backend engineer, you only need to focus on the **Application Layer**.

```
Client (JSON)
     ↓
Data Frames → IP Packets → Bits (0s and 1s) → Electrical/Optical Signals
                                                          ↓
                                              [travels across the internet]
                                                          ↓
Bits (0s and 1s) → IP Packets → Data Frames → Server (JSON)
```

Before your server ever touches incoming data, the network has already reassembled those bits back into the exact JSON the client sent. **You only deal with JSON — the intermediate layers are invisible to you.**

---

## 6. The Complete End-to-End Workflow

```
1. Client Prepares Data
   └─ Frontend JS app gathers user input

2. Serialization (Client)
   └─ Converts internal data → JSON string
   └─ Attaches to HTTP request body

3. Transmission
   └─ Data broken into bits → travels across the internet → reaches server

4. Deserialization (Server)
   └─ Receives JSON string → parses into native format (e.g., Rust struct)

5. Processing
   └─ Server executes business logic (e.g., saves to database)

6. Serialization (Server)
   └─ Packages response → converts back to JSON → sends HTTP response

7. Deserialization (Client)
   └─ Receives JSON → converts back to JS object → updates the UI
```
