# Simulated-Production-Incident-RCA


Objective

Simulate a production-like failure in a local environment, debug it using logs, identify the root cause, apply a fix, and verify resolution.

This setup demonstrates:

Incident reproduction
Log-based debugging
Root Cause Analysis (RCA)
Resolution and validation
DevOps operational mindset

---

Architecture

```
Client (curl)
   |
   v
NGINX (Reverse Proxy)
   |
   v
Backend Service (NGINX)
```

---

Prerequisites

Docker
Docker Compose
macOS / Linux / Windows
Basic knowledge of containers and logs

Verify installation:

```bash
docker --version
docker-compose --version
```

---

## 📁 Project Structure

```
.
├── docker-compose.yml
├── nginx.conf
└── README.md
```

---
PART 1: Buggy Setup (Simulated Incident)

### Incident Type

Intermittent **504 Bad Gateway** errors caused by an **unstable backend service**.

---

Buggy `docker-compose.yml`

```yaml
version: '3.8'

services:
  backend:
    image: nginx:alpine
    container_name: backend
    command: >
      sh -c "sleep 5 && exit 1"
    restart: always

  nginx:
    image: nginx:alpine
    container_name: nginx
    ports:
      - "8080:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - backend
```


NGINX Configuration (`nginx.conf`)

```nginx
events {}

http {
  upstream backend {
    server backend:80;
  }

  server {
    listen 80;

    location / {
      proxy_pass http://backend;
      proxy_connect_timeout 2s;
      proxy_read_timeout 2s;
    }
  }
}
```

---

### 3️⃣ Start the Buggy Application

```bash
docker-compose up
```

---

### 4️⃣ Reproduce the Issue

Run multiple times:

```bash
curl http://localhost:8080
```

Observed behavior:

* Sometimes success
* Sometimes:

```
504 Bad Gateway
```

---

ART 2: Debugging the Incident

Check NGINX Logs

```bash
docker logs nginx
```

Expected error:

```
connect() failed (110: Timedout)
```

---

Check Backend Logs

```bash
docker logs backend
```

You will observe:

Backend exits
Container restarts repeatedly

---

Verify Container Stability

```bash
docker ps
```

Backend status:

```
Restarting
```

---

Root Cause Analysis (RCA)

What Happened

Users experienced intermittent 504 errors.

Root Cause

The backend container crashed repeatedly after startup, making the upstream service unavailable.

Impact

Partial outage and unreliable service behavior.

---

PART 3: Resolution (Fix)

 Fixed `docker-compose.yml`

```yaml
version: '3.8'

services:
  backend:
    image: nginx:alpine
    container_name: backend
    restart: always

  nginx:
    image: nginx:alpine
    container_name: nginx
    ports:
      - "8080:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - backend
```

### What Changed

* Removed crashing command
* Backend runs a stable HTTP service
* No more restarts

---

Restart the Application

```bash
docker-compose down
docker-compose up
```

---

PART 4: Verification (Proof of Resolution)

Verify Containers

```bash
docker ps
```

Expected:

```
backend   Up
nginx    Up
```

---

Verify Backend Directly

```bash
docker exec -it backend curl localhost
```

Expected:

```html
Welcome to nginx!
```

---

Verify Through NGINX

```bash
curl http://localhost:8080
```

Expected (every time):

```html
Welcome to nginx!
```

---

