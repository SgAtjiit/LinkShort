# LinkShort - URL Shortener

![Java 21](https://img.shields.io/badge/Java-21-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![Spring Boot 3.4.3](https://img.shields.io/badge/Spring_Boot-3.4.3-6DB33F?style=for-the-badge&logo=springboot&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-Sharded_DB-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-Caching_%26_ID-DC382D?style=for-the-badge&logo=redis&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Containers-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Swagger / OpenAPI 3](https://img.shields.io/badge/OpenAPI-3.0-85EA2D?style=for-the-badge&logo=swagger&logoColor=black)

**LinkShort** is a production-grade, horizontally scalable URL Shortener built with **Java 21** and **Spring Boot 3.4.3**. Designed around core distributed system design principles, it demonstrates low-latency redirection, atomic globally-unique ID generation, sliding-window rate limiting, and horizontal database sharding across multiple MySQL nodes.

---

## ✨ Key Features

- ⚡ **Ultra-Low Latency Redirection**: Read-heavy optimization utilizing Redis Cache-Aside pattern.
- 🆔 **Distributed ID Generation**: Collision-free numeric counter converted to compact Base62 encoded strings.
- 🗄️ **Horizontal Database Sharding**: Custom application-level hashing router partitioning records across multiple MySQL instances.
- 🛡️ **Sliding Window Rate Limiting**: Redis-backed IP throttling (`429 Too Many Requests`) to prevent API abuse.
- 🧯 **Graceful Degradation & Resilience**: Automatic fallback to MySQL datastores if Redis experiences downtime.
- 📖 **Interactive API Documentation**: Embedded Swagger UI / OpenAPI 3 specification.

---

## 🛠️ Tech Stack

| Component | Technology | Description |
| :--- | :--- | :--- |
| **Language** | Java 21 | Modern LTS Java featuring high performance and virtual thread support |
| **Framework** | Spring Boot 3.4.3 | Core framework (Spring MVC, Spring Validation, Spring Data Redis) |
| **Database** | MySQL 8.x | Sharded relational storage layer using Spring `JdbcTemplate` |
| **Caching & State** | Redis | In-memory datastore for caching, rate-limiting, and ID generation |
| **Documentation** | SpringDoc OpenAPI 2.8.5 | Swagger UI endpoint playground |
| **Build Tool** | Apache Maven | Dependency management and build lifecycle |

---

## 🏗️ High-Level Architecture

```
                                  +-------------------+
                                  |    HTTP Client    |
                                  +---------+---------+
                                            |
                                            v
                                  +-------------------+
                                  |   LinkShort App   |
                                  |  (Spring Boot)    |
                                  +----+-----+----+---+
                                       |     |    |
           +---------------------------+     |    +--------------------------+
           |                                 |                               |
           v                                 v                               v
+--------------------+            +--------------------+           +--------------------+
|   Redis Cache      |            |  Shard Router      |           | Rate Limiter / ID  |
| (Cache-Aside Pattern)           | (Hash-based Routing|           |    (Redis INCR)    |
+--------------------+            +---------+----------+           +--------------------+
                                            |
                         +------------------+------------------+
                         |                                     |
                         v                                     v
             +-----------------------+             +-----------------------+
             |    MySQL Shard 0      |             |    MySQL Shard 1      |
             | (Port 3306 / Node A)  |             | (Port 3307 / Node B)  |
             +-----------------------+             +-----------------------+
```

---

## 🧠 System Design Highlights

### 1️⃣ Distributed Collision-Free ID Generation
- Avoids random string collisions and long UUIDs.
- Uses Redis `INCR` to guarantee atomic incrementing integer IDs across distributed application instances.
- Converts the numeric ID into a compact Base62 short code (`0-9`, `a-z`, `A-Z`).

```
Redis INCR  --->  Numeric ID (e.g. 125892)  --->  Base62 Encoding  --->  Short Code ("dVe")
```

### 2️⃣ Read-Optimized Caching Strategy
- Implements the **Cache-Aside Pattern** for short URL lookup operations.
- **Cache Hit**: Instantly redirects client without querying relational databases.
- **Cache Miss**: Queries the designated MySQL shard, populates Redis, and completes redirection.

### 3️⃣ Rate Limiting & Abuse Prevention
- Protects write endpoints (`POST /api/v1/shorten`).
- Uses a sliding window mechanism stored in Redis per client IP address.
- Configured with a fail-open strategy so database availability is prioritized if Redis becomes temporarily unreachable.

### 4️⃣ Application-Level Database Sharding
- Bypasses traditional single-database bottlenecks by horizontally partitioning data.
- **Shard Key**: `shortCode`
- **Shard Algorithm**: `shardId = abs(hash(shortCode)) % totalShards`
- Executes queries via Spring `JdbcTemplate` to control single-row routing without cross-shard join penalties.

---

## 📁 Directory & Folder Structure

```
LinkShort/
├── pom.xml
├── README.md
├── HELP.md
├── mvnw
├── mvnw.cmd
└── src/
    ├── main/
    │   ├── java/
    │   │   └── com/
    │   │       └── project2026/
    │   │           └── url_shortener/
    │   │               ├── UrlShortenerApplication.java
    │   │               ├── config/
    │   │               │   ├── ClientIpResolver.java
    │   │               │   ├── RedisConfig.java
    │   │               │   └── ShardDataSourceConfig.java
    │   │               ├── controller/
    │   │               │   ├── ShardTestController.java
    │   │               │   └── UrlController.java
    │   │               ├── generator/
    │   │               │   ├── Base62Generator.java
    │   │               │   └── RedisIdGenerator.java
    │   │               ├── model/
    │   │               │   └── UrlMapping.java
    │   │               ├── repository/
    │   │               │   ├── ShardedUrlRepository.java
    │   │               │   └── UrlRepository.java
    │   │               └── service/
    │   │                   ├── CacheService.java
    │   │                   ├── RateLimiterService.java
    │   │                   ├── RedisCacheService.java
    │   │                   ├── ShardRouter.java
    │   │                   └── UrlService.java
    │   └── resources/
    │       ├── application.properties
    │       └── application.yml
    └── test/
        └── java/
            └── com/
                └── project2026/
                    └── url_shortener/
                        └── UrlShortenerApplicationTests.java
```

---

## 🚀 Getting Started

### Prerequisites

Ensure you have the following installed on your machine:
- **Java 21 Development Kit (JDK 21+)**
- **Apache Maven 3.8+** (or use included `mvnw` wrapper)
- **Docker & Docker Compose** (for running MySQL shards & Redis)

### 1️⃣ Database & Cache Provisioning

Start your MySQL shards and Redis container:
```bash
docker run -d --name mysql-shard-0 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=sujal -e MYSQL_DATABASE=url_shortener mysql:8.0
docker run -d --name mysql-shard-1 -p 3307:3306 -e MYSQL_ROOT_PASSWORD=sujal -e MYSQL_DATABASE=url_shortener mysql:8.0
docker run -d --name redis-cache -p 6379:6379 redis:alpine
```

### 2️⃣ Build the Project

Build the application using Maven:
```bash
./mvnw clean compile
```

### 3️⃣ Run the Application

Launch the Spring Boot server:
```bash
./mvnw spring-boot:run
```

The application will start on `http://localhost:8080`.

---

## 🔌 API Documentation & Reference

Access the interactive Swagger UI at:  
👉 **`http://localhost:8080/swagger-ui/index.html`**

| Method | Endpoint | Description | Sample Request Body / Parameter |
| :--- | :--- | :--- | :--- |
| `POST` | `/shorten` | Shorten a long URL | `{"longUrl": "https://example.com/very-long-path"}` |
| `GET` | `/{shortCode}` | Redirect to original long URL | `GET /dVe` |
| `GET` | `/test-shard` | Test database shard routing | Query parameter: `?shortCode=dVe` |

---

## 📄 License

This repository is available under open software licensing for demonstration and educational purposes.
