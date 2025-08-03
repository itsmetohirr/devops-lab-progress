## 🐳 Lab 1: Docker Containerization

**Objective:** Create a production-ready Docker setup for the weather app

#### Task 1.1: Create Multi-Stage Dockerfile

Create `nginx.conf` that:
- Handles SPA routing (redirects all routes to index.html)
- Serves static files efficiently
- Sets appropriate MIME types



```
nginx.conf
server {
    listen 80;
    server_name localhost;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location ~* \.(js|css|html|png|jpg|jpeg|svg|woff2?)$ {
        try_files $uri =404;
        access_log off;
        expires 1y;
    }

    error_page 404 /index.html;
}
```
