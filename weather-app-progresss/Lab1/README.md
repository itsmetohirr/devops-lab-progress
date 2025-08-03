## 🐳 Lab 1: Docker Containerization

**Objective:** Create a production-ready Docker setup for the weather app

#### Task 1.1: Create Multi-Stage Dockerfile

Create a `Dockerfile` that:
- [x] Uses Node.js Alpine image for the build stage
- [x] Builds the React application
- [x] Uses nginx Alpine for the production stage
- [x] Serves the built application
- [x] Exposes port 80


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
- [x] Handles SPA routing (redirects all routes to index.html)
- [x] Serves static files efficiently
- [x] Sets appropriate MIME types

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
- [x] Container builds without errors
- [x] Application loads and functions correctly
- [x] All routes work properly (SPA routing)
- [x] Weather search functionality works
- [x] Container size is optimized (multi-stage build)

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

---
<br>
<br>

---

## 🔧 Lab 2: Code Quality and Testing Setup

#### Task 2.1: Configure ESLint for TypeScript
- [x] Install necessary ESLint packages for TypeScript
- [x] Configure ESLint to check .ts and .tsx files
- [x] Set up rules to report unused disable directives
- [x] Configure maximum warnings threshold
---
<br>

- [x] Install necessary ESLint packages for TypeScript
```bash
npm install --save-dev eslint @eslint/js typescript typescript-eslint
```

- [x] Configure ESLint to check .ts and .tsx files
- [x] Set up rules to report unused disable directives
```ts
import tseslint from "typescript-eslint";
import { defineConfig } from "eslint/config";

export default defineConfig({
  reportUnusedDisableDirectives: true,
  overrides: [
    {
      files: ["**/*.{ts,tsx}"],
      ...tseslint.configs.recommended,
    },
  ],
});
```
- [x] Configure maximum warnings threshold

```json
  "scripts": {
    "lint": "eslint . --max-warnings=10"
    ...
  }
  ```
  ---
  ---
  <br>

#### Task 2.2: Add npm Scripts

- [x] Add these scripts to `package.json`:
- `lint`: Run ESLint on TypeScript files
- `lint:fix`: Run ESLint with auto-fix
- `test`: Run tests using Vitest
- `test:coverage`: Run tests with coverage reporting


```json
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "preview": "vite preview",
    "lint": "eslint . --max-warnings=10",
    "lint:fix": "eslint . --ext .ts,.tsx --fix",
    "test": "vitest run",
    "test:coverage": "vitest run --coverage"
  }
  ```
---
---
<br>

#### Task 2.3: Set Up Testing Framework
- Install Vitest and React Testing Library
- Configure testing environment
- Set up Jest DOM matchers

---
---
<br>

#### Task 2.4: Create Basic Tests
- Write component rendering tests
- Write API service tests
- Write basic integration tests
- Organize tests in appropriate directories

---
---
<br>

### Success Criteria
- `npm run lint` passes with 0 errors
- `npm run test` passes all tests
- `npm run build` completes successfully
- Test coverage meets reasonable thresholds
