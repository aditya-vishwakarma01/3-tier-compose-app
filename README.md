# 🚀 Project: 3-Tier Application using Docker & Docker Compose

---

# 1️⃣ Architecture Overview

## 🎯 Objective

Deploy a production-style 3-tier application using Docker:

* **Frontend** → React / Static App (Nginx)
* **Backend** → Node 
* **Database** → MySQL

---

## 📐 Architecture Diagram (Draw This in Eraser.io)

```
                User (Browser)
                       |
                       v
                Nginx (Frontend)
                       |
                       v
                Backend API
                       |
                       v
                  MySQL DB
```

Now convert this to container architecture:

```
+------------------------------------------------+
|                 Docker Host                   |
|                                                |
|  [nginx-container]  --->  [backend-container] |
|                                  |             |
|                                  v             |
|                           [mysql-container]    |
|                                                |
+------------------------------------------------+
```

---

# 2️⃣ Network Design Thinking

### 🔹 Use Custom Bridge Network

Why?

* Container-to-container DNS resolution
* Isolation from default bridge
* Clean architecture

```
docker network create app-network
```

In compose → this is auto-created.

---

# 3️⃣ Folder Structure (Very Important for Interviews)

```
3tier-app/
│
├── frontend/
│   └── Dockerfile
│
├── backend/
│   └── Dockerfile
│
├── docker-compose.yml
│
└── .env
```

Interview Tip:
Always separate services cleanly.

---

# 4️⃣ Backend Dockerfile

Example: Node.js API

```dockerfile
FROM node:18-alpine

WORKDIR /app
COPY package*.json .
RUN npm install

COPY . .
EXPOSE 5000

CMD ["node", "server.js"]
```

🧠 Why alpine?
Smaller image size → production optimized.

---

# 5️⃣ Frontend Dockerfile (Multi-stage Build)

```dockerfile
# Build Stage
FROM node:18-alpine AS build
WORKDIR /app
COPY . .
RUN npm install && npm run build

# Production Stage
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
```

🧠 Why multi-stage?

* Removes node_modules
* Final image lightweight
* Security improvement

---

# 6️⃣ Database Service (No Custom Dockerfile Needed)

We use official image:

```
mysql:8
```

---

# 7️⃣ docker-compose.yml (Core of Project)

```yaml
version: '3.8'

services:

  mysql:
    image: mysql:8
    container_name: mysql-db
    restart: always
    env_file:
      - .env
    volumes:
      - db-data:/var/lib/mysql
    networks:
      - app-network

  backend:
    build: ./backend
    container_name: backend-api
    restart: always
    depends_on:
      - mysql
    ports:
      - "5000:5000"
    env_file:
      - .env
    networks:
      - app-network

  frontend:
    build: ./frontend
    container_name: frontend-app
    restart: always
    depends_on:
      - backend
    ports:
      - "80:80"
    networks:
      - app-network

volumes:
  db-data:

networks:
  app-network:
```

---

# 8️⃣ Environment Variables (.env)

```
MYSQL_ROOT_PASSWORD=rootpass
MYSQL_DATABASE=mydb
MYSQL_USER=user
MYSQL_PASSWORD=userpass
```

🧠 Why .env?

* Secrets separation
* Reusable configuration
* Production readiness

---

# 9️⃣ Internal Working (Explain This in Interview)

When you run:

```
docker-compose up -d --build
```

What happens internally?

1. Compose reads YAML
2. Creates custom network
3. Builds frontend image
4. Builds backend image
5. Pulls mysql image
6. Creates volume
7. Starts containers in dependency order
8. DNS resolution works via service names

Backend connects to MySQL using:

```
host = mysql
```

NOT localhost.

This is a very common interview trap.

---

# 🔐 10️⃣ Security Improvements (Add This to Impress Interviewer)

### Production Improvements

* Use non-root user in Dockerfile
* Use .dockerignore
* Remove unnecessary packages
* Use secrets instead of plain .env
* Use healthcheck in compose
* Add resource limits

Example:

```yaml
deploy:
  resources:
    limits:
      cpus: "0.5"
      memory: 512M
```

---

# 📊 11️⃣ Monitoring (Optional Extension)

Add:

* Prometheus
* Grafana
* Logging driver
* docker stats

---

# 🚀 12️⃣ CI/CD Extension (DevOps Level Upgrade)

You can extend this:

* Push code to GitHub
* GitHub Actions builds image
* Push to Docker Hub
* Pull on server
* Deploy using docker-compose

Pipeline flow:

```
Developer → GitHub → CI build → Push image → Server pull → Deploy
```

---

# 🎯 13️⃣ Common Interview Questions From This Project

### Q1: How do containers communicate?

Through Docker bridge network via service name DNS.

### Q2: Why use volumes?

To persist MySQL data beyond container lifecycle.

### Q3: What if MySQL container restarts?

Volume ensures data remains intact.

### Q4: How do you scale backend?

```
docker-compose up --scale backend=3
```

(But for production → use Kubernetes)

### Q5: What is depends_on limitation?

It controls startup order, not readiness.

---

# 🧠 What Makes This “DevOps-Level” Instead of Beginner

| Beginner             | DevOps Thinking    |
| -------------------- | ------------------ |
| Just runs containers | Designs network    |
| Hardcoded secrets    | Uses env files     |
| No persistence       | Uses volumes       |
| Single container     | Multi-tier         |
| No scalability       | Scale-ready        |
| No CI/CD             | Automated pipeline |

---

# 📐 How You Should Draw in Eraser.io

Draw 3 layers:

Layer 1 → Browser
Layer 2 → Nginx Container
Layer 3 → Backend Container
Layer 4 → MySQL Container
Side → Volume attached to MySQL
Bottom → Docker Network connecting all

Label:

* Ports
* Environment variables
* Volume mount
* Network name

---

# 🔥 If You Want Next Level

We can now:

1. Upgrade this to EC2 deployment project
2. Add Nginx Reverse Proxy with SSL
3. Add Terraform + Ansible provisioning
4. Convert this into Kubernetes YAML
5. Turn this into resume-ready project description

---

You’re thinking correctly — building projects to remember concepts is how real DevOps engineers grow.

Tell me:
Do you want to upgrade this to a **production-ready cloud deployment version next?**
