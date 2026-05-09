---
title: "What is a Request?"
date: 2025-11-14T22:01:10+03:07
draft: false
tags: ["http"]
---

A Request is seen in almost every framework for web and APIs. I first saw this in Django, then in FastAPI, and I thought: this is everywhere, and it is not language- or framework-specific. So how does this work? How is this universal? What exactly is it? These questions led me to look deeper into this, and I found out that it is just a string of bytes with a specific structure, like a letter.

It is the structure that every framework follows, and the structure broken down into its individual components is collectively called a request object. That is it. This is what makes it universal and framework-agnostic.

In reality, there is no object. The same applies for the response object. Each request and response has a specific format.

---

## Example Request

```sh
POST /api/login HTTP/1.1\r\n
Host: example.com\r\n
Content-Type: application/json\r\n
Content-Length: 48\r\n
\r\n
{
  "username": "eye",
  "password": "tough"
}
\r\n\r\n
```

---

This is what the server actually receives. It is not JSON or a Python object — just a sequence of bytes. The request has a specific structure, and each of the sections are separated by strings, too: CRLF.

---

## Request Breakdown

### 1. Request Line

```sh
POST /api/login HTTP/1.1\r\n
```

- `POST`       → HTTP method
- `/api/login` → Route / endpoint
- `HTTP/1.1`   → HTTP version
- `\r\n`       → End of line

### 2. Headers

```sh
Host: example.com\r\n
Content-Type: application/json\r\n
Content-Length: 48\r\n
Authorization: Bearer abc123\r\n
```

Each header line ends with `\r\n`.

### 3. Header Terminator

```sh
\r\n
```

This empty line separates headers from the request body.

### 4. Request Body

```json
{
  "username": "john",
  "password": "secret"
}
```

### 5. Complete End Marker

The sequence `\r\n\r\n` means:

- First `\r\n`  → end of last header
- Second `\r\n` → blank line before body

This tells the server: **"headers are finished."**

---

The server continuously listens for these requests by opening sockets. At the socket level, nothing arrives as a "complete request." You do something like:

```python
conn, addr = server.accept()
data = conn.recv(1024)
```

`recv()` gives you **whatever bytes are available at that moment**. It is not guaranteed to be:
- complete
- aligned
- even readable

---

The client does NOT send everything "at once." Data flows like this:

```sh
Client ───────► [chunk1] ───────► [chunk2] ───────► Server
```

So the server may receive:

```sh
b"GET / HTTP/1.1\r\nHost: exa"
b"mple.com\r\nUser-Agent: curl\r\n\r\n"
```

The server accumulates and reconstructs the full request.

---

## Parsing a Request from Raw Bytes

Frameworks hide this, but internally they do something like this:

### 1. Read Raw Bytes

```python
buffer = conn.recv(1024)
```

At this point, the server only has raw bytes.

---

### 2. Detect End of Headers

```python
if b"\r\n\r\n" in buffer:
    headers_part, body = buffer.split(b"\r\n\r\n", 1)
```

`\r\n\r\n` marks the end of the headers.

---

### 3. Parse Header Lines

```python
lines = headers_part.split(b"\r\n")
```

Now the request is broken into individual lines.

---

### 4. Extract Meaning

```python
request_line = lines[0].decode()

method, path, version = request_line.split()
```

Now you finally have something like:

```sh
GET / HTTP/1.1
```

This is where a **request object is constructed**.

---

## So What Is the "Request Object"?

It is just a **parsed representation of raw bytes**:

```python
{
    "method": "GET",
    "path": "/",
    "headers": {...},
    "body": ...
}
```

Frameworks like FastAPI or Django build this for you. But underneath it all, it came from a raw byte stream.

---

## The Illusion of a Single Request

From the developer's perspective:

```python
def handler(request):
    ...
```

Looks like one clean object.

But in reality:

- data arrived in chunks
- was buffered
- parsed
- transformed

The "object" is an abstraction, not reality.

---

## Same Idea Applies to Responses

Servers do not send responses "in one go" either.

They use the same socket:

```python
conn.send(b"HTTP/1.1 200 OK\r\n")
conn.send(b"Content-Type: text/plain\r\n\r\n")
conn.send(b"Hello World")
```

This may be sent in multiple chunks.

---

## Why It Still Looks Instant

Because:

- TCP guarantees ordered delivery
- the OS buffers data
- the client waits until enough data arrives

So the user sees: **one complete response**

Even though it was streamed.

---

## Connecting to Your Own Server

If you have built a minimal server using sockets, you have already done this:

- read raw bytes
- handled partial reads
- looked for `\r\n\r\n`
- parsed manually

That is literally what production servers do — just highly optimized.

---

## Final Thought

There is no magic request object.

Just:

```
bytes → buffer → parse → object
```

---