# 🐳 DockerStack — Dockerized Multi-Tier Java Web Application

A production-like containerized deployment of a multi-tier Java web application using **Docker** and **Docker Compose**. The stack includes a reverse proxy, application server, relational database, caching layer, and message broker — all wired together and orchestrated as isolated services.

---

## 🧱 Architecture Overview

```
Client
  │
  ▼
┌─────────────────────────────────────────────────────┐
│  vproweb (Nginx)       :80                          │  ← Reverse Proxy / Entry Point
└───────────────────────────┬─────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────┐
│  vproapp (Tomcat)      :8080                        │  ← Java App Server (.war)
└────────┬──────────────────┬──────────────────┬──────┘
         │                  │                  │
         ▼                  ▼                  ▼
   ┌──────────┐      ┌────────────┐    ┌───────────────┐
   │  vprodb  │      │vprocache01 │    │   vpromq01    │
   │  MySQL   │      │ Memcached  │    │   RabbitMQ    │
   │  :3306   │      │  :11211    │    │    :5672      │
   └──────────┘      └────────────┘    └───────────────┘
```

| Service        | Container      | Image                       | Role                        |
|----------------|----------------|-----------------------------|-----------------------------|
| `vproweb`      | vproweb        | `shivang125/vprofileweb`    | Nginx reverse proxy         |
| `vproapp`      | vproapp        | `shivang125/vprofileapp`    | Tomcat Java application     |
| `vprodb`       | vprodb         | `shivang125/vprofiledb`     | MySQL database              |
| `vprocache01`  | vprocache01    | `memcached:latest`          | In-memory caching           |
| `vpromq01`     | vpromq01       | `rabbitmq`                  | Message broker              |

---

## 📂 Project Structure

```
.
├── Docker-files/
│   ├── app/                # Tomcat + Maven multi-stage build Dockerfile
│   ├── db/                 # MySQL Dockerfile + SQL initialization script
│   └── web/                # Nginx config + Dockerfile
├── docker-compose.yml      # Full service orchestration
└── README.md
```

---

## ⚙️ Tech Stack

- **Docker** & **Docker Compose** — containerization & orchestration
- **Java / Maven** — application build
- **Apache Tomcat** — servlet container (serves `.war`)
- **MySQL** — relational database with persistent volume
- **Memcached** — high-performance in-memory caching
- **RabbitMQ** — async message queuing
- **Nginx** — reverse proxy and load balancer

---

## 🚀 Getting Started

### Prerequisites

- [Docker](https://docs.docker.com/get-docker/) ≥ 20.x
- [Docker Compose](https://docs.docker.com/compose/install/) ≥ 2.x

### 1. Clone the Repository

```bash
git clone https://github.com/<your-username>/vprofile-dockerized-app.git
cd Multi-Container-docker-deployment
```

### 2. Build and Start All Services

```bash
docker-compose up --build
```

> Use `-d` to run in detached (background) mode:
> ```bash
> docker-compose up --build -d
> ```

### 3. Access the Application

| URL                        | Service         |
|----------------------------|-----------------|
| `http://localhost`         | Web App (Nginx) |
| `http://localhost:8080`    | Tomcat Direct   |
| `localhost:3306`           | MySQL           |
| `localhost:11211`          | Memcached       |
| `localhost:5672`           | RabbitMQ AMQP   |

---

## 🔩 Service Configuration Details

### 🗄️ MySQL (`vprodb`)
- Built from `./Docker-files/db`
- Persists data via Docker volume: `vprodbdata`
- Root password: `vprodbpass` *(change before production use)*
- Port: `3306`

### ⚡ Memcached (`vprocache01`)
- Uses official `memcached:latest` image
- Port: `11211`
- Reduces repeated DB queries by caching frequently accessed data

### 📨 RabbitMQ (`vpromq01`)
- Uses official `rabbitmq` image
- Default credentials: `guest / guest` *(change before production use)*
- Port: `5672` (AMQP)

### ☕ Tomcat App (`vproapp`)
- Built from `./Docker-files/app` (Maven + Tomcat multi-stage build)
- Webapp files persisted via volume: `vproappdata`
- Port: `8080`

### 🌐 Nginx (`vproweb`)
- Built from `./Docker-files/web`
- Listens on port `80` and proxies to `vproapp:8080`
- Port: `80`

---

## 📦 Docker Volumes

| Volume        | Mounted In                             | Purpose                    |
|---------------|----------------------------------------|----------------------------|
| `vprodbdata`  | `/var/lib/mysql` (vprodb)              | MySQL data persistence     |
| `vproappdata` | `/usr/local/tomcat/webapps` (vproapp)  | Deployed application files |


---

## 🔐 Security Notes

> ⚠️ The following defaults are **for development only**. Update before any deployment:
> - MySQL root password: `vprodbpass`
> - RabbitMQ credentials: `guest / guest`

Consider using Docker secrets or environment variable files (`.env`) to manage sensitive values.


---

## 🤝 Contributing

Pull requests are welcome! For major changes, please open an issue first to discuss what you'd like to change.

---
