# Table-of-Contents

<!-- toc -->

- [What is a Backend?](#what-is-a-backend)
  * [Definition](#definition)
  * [How a Request Travels — End to End](#how-a-request-travels--end-to-end)
    + [1. DNS Resolution](#1-dns-resolution)
    + [2. Cloud Firewall (AWS Security Groups)](#2-cloud-firewall-aws-security-groups)
    + [3. Reverse Proxy (Nginx)](#3-reverse-proxy-nginx)
    + [4. Application Server](#4-application-server)
  * [Why Do We Need Backends?](#why-do-we-need-backends)
  * [Why Can't We Do This in the Frontend?](#why-cant-we-do-this-in-the-frontend)
    + [How the Frontend Works](#how-the-frontend-works)
    + [Reason 1: Security Sandboxing](#reason-1-security-sandboxing)
    + [Reason 2: CORS Policy](#reason-2-cors-policy)
    + [Reason 3: No Native Database Drivers](#reason-3-no-native-database-drivers)
    + [Reason 4: Computing Power](#reason-4-computing-power)
  * [Key Distinction: Frontend vs Backend Runtime](#key-distinction-frontend-vs-backend-runtime)

<!-- tocstop -->

---

# What is a Backend?

**Source:** Sriniously — Backend from First Principles (Video 3)

---

## Definition

- A backend is a computer listening for incoming requests (HTTP, WebSocket, gRPC, etc.) through an open port (e.g., 80, 443) accessible over the internet
- Clients or frontends connect to it to send or receive data
- Called a "server" because it *serves* content — static files (HTML, JS, images) or dynamic data (JSON)
- It also *accepts* data sent by clients

---

## How a Request Travels — End to End

**Flow:** Browser → DNS → Firewall → EC2 Instance → Reverse Proxy (Nginx) → App Server

### 1. DNS Resolution
- The domain name (e.g., `backend-demo.domain.xyz`) is resolved by a DNS server
- DNS A records map a subdomain to a specific IP address
- CNAME records map a subdomain to another domain name
- The IP returned is the public IP of the cloud server (e.g., AWS EC2 instance)

### 2. Cloud Firewall (AWS Security Groups)
- Before the request reaches the server, it passes through a firewall
- AWS Security Groups define which ports are open to the internet
- Port 22 — SSH (admin access)
- Port 80 — HTTP traffic
- Port 443 — HTTPS traffic
- If these ports are not explicitly allowed, AWS blocks the request

### 3. Reverse Proxy (Nginx)
- Sits in front of the actual application server
- Manages redirects and configs from a centralized place
- Port 80 → redirects to Port 443 (HTTPS, managed by Certbot/SSL)
- Based on the `server_name` (domain), forwards traffic to the correct local port (e.g., `localhost:3001`)
- Allows multiple services to run on the same machine on different ports

### 4. Application Server
- The actual Node/Python/any backend process running on `localhost:3001`
- Managed by a process manager (e.g., PM2) to keep the process alive
- Receives the request, processes it, returns a response

---

## Why Do We Need Backends?

**Example — Instagram Like button:**
1. User taps Like
2. App sends a request to the server
3. Server identifies the user (auth)
4. Server persists the action to a database
5. Server identifies the post owner
6. Server triggers a notification to the post owner

- A backend is a **centralized** computer that holds state for all users
- The core purpose of a backend, stripped to one word: **data**
  - Fetch data
  - Receive data
  - Persist data
  - Trigger actions based on data

---

## Why Can't We Do This in the Frontend?

### How the Frontend Works
- Browser fetches HTML, then JS/CSS/fonts in subsequent requests
- All JavaScript is sent to and **executed by the browser** (client-side runtime)
- Unlike a backend where the server processes the request and returns a result — the frontend sends code and the browser runs it

### Reason 1: Security Sandboxing
- Browser environments are isolated from the OS, file system, and processes
- Code from a remote server runs in the browser — if unrestricted, it could access your files and send them to the server
- Browsers enforce this isolation intentionally

### Reason 2: CORS Policy
- Browsers restrict JavaScript from calling APIs on a different domain
- A frontend can only call resources from its own domain by default
- Backends need to call many external APIs freely — CORS restrictions make this impossible from the browser

### Reason 3: No Native Database Drivers
- Backends use native DB drivers (e.g., `pg` for PostgreSQL, `pymongo` for MongoDB)
- These drivers handle socket connections, binary data, and **persistent connection pools**
- Connection pooling is critical — backends receive thousands of requests per second; creating/destroying DB connections per request would overwhelm the DB
- Browsers cannot maintain persistent connections to databases and are not designed for connection pooling

### Reason 4: Computing Power
- Frontend runs on any device — a phone, a low-RAM laptop, anything
- Heavy business logic on the client would lag or break on weaker devices
- A centralized backend server can have its memory and CPU scaled independently based on load

---

## Key Distinction: Frontend vs Backend Runtime

| | Frontend | Backend |
|---|---|---|
| Runtime | Browser | Server OS |
| Code execution | Client's machine | Server's machine |
| File system access | No | Yes |
| Native DB drivers | No | Yes |
| External API calls | Restricted (CORS) | Unrestricted |
| Compute scaling | Limited by user device | Independently scalable |
| Security isolation | Sandboxed | Full OS access |
