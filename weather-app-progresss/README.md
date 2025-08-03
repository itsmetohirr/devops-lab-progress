## 🐳 Lab 1: Docker Containerization

**Objective:** Create a production-ready Docker setup for the weather app

#### Task 1.1: Create Multi-Stage Dockerfile

Create a `Dockerfile` that:
- Uses Node.js Alpine image for the build stage
- Builds the React application
- Uses nginx Alpine for the production stage
- Serves the built application
- Exposes port 80


```Dockerfile
FROM node:alpine AS builder

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm install

COPY . .
RUN npm run build


FROM nginx:alpine

RUN rm -rf /usr/share/nginx/html/*

COPY --from=builder /app/dist /usr/share/nginx/html

COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

---
---
<br>

#### Task 1.2: Create Nginx Configuration

Create `nginx.conf` that:
- Handles SPA routing (redirects all routes to index.html)
- Serves static files efficiently
- Sets appropriate MIME types

nginx.conf 

```nginx
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

#### Task 1.3: Build and Test Container

```bash
docker build -t weather-app .

docker run -p 8080:80 --name weather-app-container weather-app
```
---
---
<br>

### Success Criteria
- Container builds without errors
- Application loads and functions correctly
- All routes work properly (SPA routing)
- Weather search functionality works
- Container size is optimized (multi-stage build)

```bash
root@ip-172-31-45-9:~/project/weather-app# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
weather-app   latest    fe3d8484b3b9   20 hours ago   52.7MB
```

```bash
root@ip-172-31-45-9:~# docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED        STATUS         PORTS                                     NAMES
6d64a90fc028   weather-app   "/docker-entrypoint.…"   21 hours ago   Up 5 minutes   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   weather-app-container
```
---
---
<br>

Link to working website:  
[ Weather-app ](http://ec2-98-86-58-135.compute-1.amazonaws.com:8080)


