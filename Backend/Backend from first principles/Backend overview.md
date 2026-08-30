# Table-of-Contents

<!-- toc -->

- [**What is a Backend?** (0:00 - 0:56)](#what-is-a-backend-000---056)
- [**How Backends Work** (0:56 - 7:30)](#how-backends-work-056---730)
- [**Why Do We Need Backends?** (7:30 - 10:20)](#why-do-we-need-backends-730---1020)
- [**Frontend Fundamentals & Limitations** (10:20 - 12:40)](#frontend-fundamentals--limitations-1020---1240)
- [**Why Backend Logic Cannot Be in the Frontend** (12:40 - 18:45)](#why-backend-logic-cannot-be-in-the-frontend-1240---1845)

<!-- tocstop -->

---
This lecture provides an introduction to the fundamentals of backend systems, how they interact with the internet, and why they are necessary compared to frontends.

### **What is a Backend?** (0:00 - 0:56)
* A backend is a computer/server listening for network requests (HTTP, WebSocket, gRPC) via open ports (80 or 443).
* Its primary purpose is to **serve content** (files, JSON) and **process data** sent by clients.

### **How Backends Work** (0:56 - 7:30)
* **Request Path:** Requests travel from the browser → DNS Server → Firewall/Security Group → Reverse Proxy (Nginx) → Application Server.
* **DNS:** Maps domain names to specific IP addresses of hosted instances (e.g., AWS EC2).
* **Security:** AWS Security Groups must explicitly allow ports 80 (HTTP) and 443 (HTTPS) for traffic to pass.
* **Reverse Proxy:** Tools like *Nginx* act as a gateway, managing redirects, SSL certificates (*Certbot*), and forwarding traffic to local application ports (e.g., Localhost 3001).
* **Process Management:** Tools like *PM2* are used to keep backend and frontend processes running continuously.

### **Why Do We Need Backends?** (7:30 - 10:20)
* **Centralized Data:** Backends act as a single source of truth for all users.
* **Persistence:** They handle saving/retrieving data from databases to ensure states (like "likes" or profiles) remain updated for everyone.
* **Notification Logic:** Backends orchestrate communication between different users (e.g., sending a notification when a user likes a post).

### **Frontend Fundamentals & Limitations** (10:20 - 12:40)
* **Execution:** Frontend code (JS/CSS/HTML) is fetched by the browser and executed directly on the **client machine**.
* **Rendering:** The browser fetches the HTML structure first, then downloads resources and "paints" the page (CSS) and adds interactivity (JS).

### **Why Backend Logic Cannot Be in the Frontend** (12:40 - 18:45)
* **Security:** Browser runtimes are sandboxed to isolate the user's OS and file system from potentially malicious code.
* **CORS Restrictions:** Browser policies (CORS) block scripts from calling external APIs unless specifically authorized, limiting flexibility.
* **Database Connections:** Browsers cannot maintain secure, persistent database connections (connection pooling) and would overwhelm databases if every client connected directly.
* **Computing Power:** Backend servers can be scaled (CPU/RAM) to handle heavy business logic, whereas client devices vary greatly in performance, which could lead to lag or crashes.
