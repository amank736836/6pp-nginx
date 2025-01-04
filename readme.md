
## Learning NGINX - README.md

### Overview

This document explains different NGINX configurations for various use cases, such as serving static files, setting up proxy servers, and load balancing. It also includes examples for handling different routes and domain-based routing.

---

### 1. Basic Static File Serving

**Configuration Example**:
```nginx
server {
    listen 80;

    location / {
        root /var/www/html/firstApp;
        try_files /index.html =400;
    }

    location /nice {
        root /var/www/html/firstApp;
        index index.html;
    }

    location /about {
        root /var/www/html/firstApp;
        index index.html;
    }

    location /contact {
        alias /var/www/html/firstApp/sample;
        index index.html;
    }

    location /sample {
        root /var/www/html/firstApp;
        try_files /sample.html =400;
    }
}
```

**Use Case**:
- Serve static files for routes like `/`, `/nice`, `/about`, etc.
- Use `alias` for paths like `/contact` to serve files from a different directory.
- Handle missing files with `try_files`.

---

### 2. Simple Proxy Setup

**Configuration Example**:
```nginx
server {
    listen 80;

    location / {
        proxy_pass http://localhost:4173;
    }

    location /api {
        proxy_pass http://localhost:4000;
    }
}
```

**Use Case**:
- Route `/` requests to a frontend application (`http://localhost:4173`).
- Route `/api` requests to a backend application (`http://localhost:4000`).

---

### 3. Domain-Based Proxy Routing

**Configuration Example**:
```nginx
server {
    listen 80;
    server_name backend-vs.mooo.com virtuostore.mooo.com;

    location / {
        if ($host = virtuostore.mooo.com) {
            proxy_pass http://localhost:4173;
        }

        if ($host = backend-vs.mooo.com) {
            proxy_pass http://localhost:4000;
        }
    }
}
```

**Use Case**:
- Route requests based on the `Host` header to separate backend and frontend services.

---

### 4. Load Balancing with Upstream

**Configuration Example**:
```nginx
upstream backend {
    server localhost:4000 weight=1;
    server localhost:4001 weight=3;
    server localhost:4002 weight=2;
}

server {
    listen 80;
    server_name backendvs.mooo.com;

    location / {
        proxy_pass http://backend;
    }
}

server {
    listen 80;
    server_name virtuostore.mooo.com;

    location / {
        proxy_pass http://localhost:4173;
    }
}
```

**Use Case**:
- Distribute traffic across multiple backend instances with weight-based load balancing.
- Separate domains for backend and frontend services.

---

### 5. Combined Domain and Load Balancing Configuration

**Configuration Example**:
```nginx
upstream backend {
    server localhost:4000 weight=1;
    server localhost:4001 weight=3;
    server localhost:4002 weight=2;
}

server {
    listen 80;
    server_name backendvs.mooo.com virtuostore.mooo.com;

    location / {
        if ($http_host = virtuostore.mooo.com) {
            proxy_pass http://localhost:4173;
        }

        if ($http_host = backendvs.mooo.com) {
            proxy_pass http://backend;
        }

        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**Use Case**:
- Combine domain-based routing and load balancing in a single configuration.
- Ensure headers are set correctly for proxied requests.

---


### Notes and Best Practices

1. **Separation of Concerns**:
   - Use separate `server` blocks for different domains.
   - Group related settings to improve readability.

2. **Testing Configuration**:
   - Always test configurations with:
     ```bash
     sudo nginx -t
     ```

3. **Reloading NGINX**:
   - Apply changes with:
     ```bash
     sudo systemctl reload nginx
     ```

4. **Avoid `if` in `location` Blocks**:
   - Use `server_name` for domain-based routing instead of `if` conditions.

---

Let me know if you want further assistance or a refined structure for this guide!