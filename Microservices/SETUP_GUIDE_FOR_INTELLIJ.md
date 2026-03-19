# RevConnect - IntelliJ Setup Guide

## Prerequisites
1. **Java 17+** installed
2. **Maven** installed (or use IntelliJ's bundled Maven)
3. **Docker Desktop** installed and **running**

---

## Step 1: Start Docker MySQL Containers

The project uses **5 separate MySQL containers** (NOT local MySQL on port 3306).

Open a terminal in the project root (`d:\Microservices`) and run:

```bash
docker-compose up -d
```

This starts 5 MySQL containers:
| Container         | Port  | Database        |
|-------------------|-------|-----------------|
| mysql-user        | 3307  | user_db         |
| mysql-post        | 3308  | post_db         |
| mysql-interaction | 3309  | interaction_db  |
| mysql-connection  | 3310  | connection_db   |
| mysql-notification| 3311  | notification_db |

**Verify containers are running:**
```bash
docker ps
```
You should see all 5 `mysql-*` containers with status "Up".

---

## Step 2: Fix "Communication Link Failure / Unable to open JDBC connector"

This error means Docker MySQL containers are **not running** or **not reachable**.

### Common Fixes:

1. **Docker Desktop must be running** — Open Docker Desktop first
2. **Start containers:** `docker-compose up -d` in the project root
3. **Wait 10-15 seconds** after containers start before launching services
4. **Check ports are not blocked** — Run:
   ```bash
   netstat -ano | findstr "3307 3308 3309 3310 3311"
   ```
   You should see LISTENING on all 5 ports.

5. **If using local MySQL instead of Docker**, update each service's `application.yml`:
   ```yaml
   spring:
     datasource:
       url: jdbc:mysql://localhost:3306/user_db  # Change port to 3306
       username: root
       password: YOUR_LOCAL_PASSWORD  # Your local MySQL password
   ```
   And create the databases manually:
   ```sql
   CREATE DATABASE IF NOT EXISTS user_db;
   CREATE DATABASE IF NOT EXISTS post_db;
   CREATE DATABASE IF NOT EXISTS interaction_db;
   CREATE DATABASE IF NOT EXISTS connection_db;
   CREATE DATABASE IF NOT EXISTS notification_db;
   ```

---

## Step 3: Import Project in IntelliJ

1. **File → Open** → Select `d:\Microservices` folder
2. IntelliJ will detect the Maven modules automatically
3. Wait for Maven to download all dependencies (first time takes a few minutes)

---

## Step 4: Run Services (in this order)

Start each service as a Spring Boot application in IntelliJ:

1. **eureka-server** (port 8761) — Start FIRST, wait until it's fully up
2. **api-gateway** (port 8080)
3. **user-service** (port 8081)
4. **post-service** (port 8082)
5. **feed-service** (port 8083)
6. **interaction-service** (port 8084)
7. **connection-service** (port 8085)
8. **notification-service** (port 8086)

### How to run in IntelliJ:
- Navigate to each service's `*Application.java` main class
- Right-click → **Run**
- Or use the **Services** tab (View → Tool Windows → Services) to manage all

---

## Step 5: Run Frontend

```bash
cd frontend
npm install    # First time only
ng serve
```

Frontend runs at: http://localhost:4200

---

## Troubleshooting

| Error | Solution |
|-------|----------|
| Communication link failure | Docker not running. Start Docker Desktop + `docker-compose up -d` |
| Port already in use | Another instance is running. Kill it or change the port in `application.yml` |
| Connection refused on 3307-3311 | MySQL containers not started. Run `docker-compose up -d` |
| Eureka connection refused | Start eureka-server first and wait 15 seconds |
| 401 Unauthorized | JWT token expired. Re-login from frontend |

---

## Quick Start (All Commands)

```bash
# 1. Start Docker containers
docker-compose up -d

# 2. Wait for MySQL to initialize
timeout 15

# 3. Build all services
mvn clean package -DskipTests

# 4. Start services (run each in separate terminal)
java -jar eureka-server/target/eureka-server-1.0.0.jar
java -jar api-gateway/target/api-gateway-1.0.0.jar
java -jar user-service/target/user-service-1.0.0.jar
java -jar post-service/target/post-service-1.0.0.jar
java -jar feed-service/target/feed-service-1.0.0.jar
java -jar interaction-service/target/interaction-service-1.0.0.jar
java -jar connection-service/target/connection-service-1.0.0.jar
java -jar notification-service/target/notification-service-1.0.0.jar

# 5. Start frontend
cd frontend && ng serve
```
