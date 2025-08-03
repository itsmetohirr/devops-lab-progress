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

---
<br>
<br>

---

## ☁️ Lab 3: AWS S3 Static Website Hosting

#### Task 3.3: Create Terraform Configuration

1. **AWS Provider Configuration:**
   - Set region to "us-east-1"

2. **S3 Bucket Resources:**
   - S3 bucket with unique name (use random_string)
   - Website configuration with index.html
   - Public access block settings
   - Bucket policy for public read access
   - Appropriate tags

3. **Outputs:**
   - Website endpoint URL
   - Bucket name

**Don't forget to disable public acces block**

<br>
<br>
./terraform/main.tf

```terraform
provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "my_s3" {
  bucket = "tohirdevops"
}

resource "aws_s3_bucket_website_configuration" "web_conf" {
  bucket = aws_s3_bucket.my_s3.id

  index_document {
    suffix = "index.html"
  }

  error_document {
    key = "error.html"
  }
}

resource "aws_s3_bucket_public_access_block" "public_access" {
  bucket = aws_s3_bucket.my_s3.id

  block_public_acls       = false
  block_public_policy     = false
  ignore_public_acls      = false
  restrict_public_buckets = false
}

resource "aws_s3_bucket_policy" "public_read" {
  bucket = aws_s3_bucket.my_s3.id
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid       = "PublicReadGetObject"
        Effect    = "Allow"
        Principal = "*"
        Action    = "s3:GetObject"
        Resource  = "${aws_s3_bucket.my_s3.arn}/*"
      }
    ]
  })
}

output "website_url" {
  description = "The S3 static website endpoint"
  value       = aws_s3_bucket_website_configuration.web_conf.website_endpoint
}

output "bucket_name" {
  description = "The name of the S3 bucket"
  value       = aws_s3_bucket.my_s3.bucket
}

```
<br>

To see the bucket name:

```bash
terraform output bucket_name
```

To see the website_url:

```bash
terraform output website_url
```

### Success Criteria
- [x] S3 bucket created with static website hosting
- [x] Production build uploaded successfully
- [x] Application accessible via S3 website URL
- [x] All weather functionality works
- [x] Terraform state managed properly
<br>

Bucket website endpoint: <br>
http://tohirdevopsvention.s3-website-us-east-1.amazonaws.com/

---
---
<br>
<br>


