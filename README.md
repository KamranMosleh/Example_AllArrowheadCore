# All Arrowhead Core Docker Setup

This repository provides a self‐contained Docker Compose setup that builds and runs **all Arrowhead Core modules** against a MySQL 8.0 backend. Each core system is configured via external `.properties` files and exposed on its own port for easy local development and testing.

---

## Table of Contents

1. [Prerequisites](#prerequisites)  
2. [Getting Started](#getting-started)  
3. [Folder Structure](#folder-structure)  
4. [Database Initialization](#database-initialization)  
5. [Running the Core Services](#running-the-core-services)  
6. [Configuration](#configuration)  
7. [Ports & Endpoints](#ports--endpoints)  
8. [Enabling SSL/TLS](#enabling-ssltls)  
9. [Adding or Disabling Modules](#adding-or-disabling-modules)  
10. [Contributing](#contributing)  
11. [License](#license)

---

## Prerequisites

- [Docker Engine ≥ 20.10](https://docs.docker.com/get-docker/)  
- [Docker Compose ≥ 1.29](https://docs.docker.com/compose/install/)  
- (Optional) Git, if you want to clone this repo  

---

## Getting Started

1. **Clone this repository**  
   ```bash
   git clone https://github.com/<your-org>/AllArrowheadCore.git
   cd AllArrowheadCore
   ```
2. **(Optional) Fetch latest SQL scripts**  
   If you want to pull the most recent Arrowhead SQL definitions, run:  
   ```bash
   ./initSQL.sh
   ```
3. **Launch all services**  
   ```bash
   docker-compose up --build
   ```
4. **Access the Service Registry**  
   Open your browser at [http://localhost:8443/](http://localhost:8443/).

---

## Folder Structure

```
example_AllArrowheadCore/
│
├─ docker-compose.yml         # Docker Compose definition for MySQL + Core image
├─ initSQL.sh                 # Helper script to fetch and arrange SQL files
│
├─ properties/                # Spring Boot .properties per core module
│   ├─ serviceregistry.properties
│   ├─ authorization.properties
│   ├─ orchestrator.properties
│   ├─ eventhandler.properties
│   ├─ gatekeeper.properties
│   ├─ gateway.properties
│   └─ certificateauthority.properties
│
└─ sql/
   ├─ create_empty_arrowhead_db.sql
   └─ privileges/             # One SQL file per core system to create tables & grants
       ├─ create_arrowhead_tables.sql
       ├─ service_registry_privileges.sql
       ├─ orchestrator_privileges.sql
       └─ …  
```

---

## Database Initialization

- The **MySQL** container mounts `./sql/` into `/docker-entrypoint-initdb.d/`.  
- On first startup, `create_empty_arrowhead_db.sql` is executed, which in turn `SOURCE`s each file under `privileges/`.  
- Each Arrowhead core system receives its own DB user and the proper tables via those scripts.  
- A named volume (`arrowhead_core_mysql`) persists your data across restarts.

---

## Running the Core Services

- The **Arrowhead Core** container is built from your local Arrowhead Java-Spring repo.  
- It **depends on** the MySQL service, ensuring the database is ready before startup.  
- JVM heap is capped at 1 GB for lightweight development (`-Xmx1G`).  
- Only modules with a corresponding `.properties` in `./properties/` will start.

---

## Configuration

Each `*.properties` follows this pattern:

```properties
# Database
spring.datasource.url=jdbc:mysql://arrowhead_core_mysql:3306/arrowhead?serverTimezone=Europe/Stockholm
spring.datasource.username=<system_user>
spring.datasource.password=<strong_password>
spring.jpa.hibernate.ddl-auto=none

# HTTP / SSL
server.port=<module_port>
server.ssl.enabled=false

# Arrowhead-specific settings...
```

- **Datasource URL** uses the Compose service name `arrowhead_core_mysql`.  
- **Credentials** must match those granted in the SQL scripts.  
- **SSL** is disabled by default for “insecure” HTTP access.

---

## Ports & Endpoints

| Module                    | Host Port | Default Endpoint             |
|---------------------------|-----------|------------------------------|
| Service Registry          | 8443      | `http://localhost:8443/`     |
| Authorization             | 8445      | `http://localhost:8445/`     |
| Orchestrator              | 8441      | `http://localhost:8441/`     |
| Event Handler             | 8455      | `http://localhost:8455/`     |
| Gatekeeper                | 8449      | `http://localhost:8449/`     |
| Gateway                   | 8453      | `http://localhost:8453/`     |
| Certificate Authority     | 8448      | `http://localhost:8448/`     |

---

## Enabling SSL/TLS

1. Generate or provide a Java KeyStore (`.jks` or `.p12`).  
2. Mount it into the container:
   ```yaml
   volumes:
     - ./keystore/arrowhead.jks:/opt/arrowhead-core/keystore/arrowhead.jks
   ```
3. In each `*.properties`, set:
   ```properties
   server.ssl.enabled=true
   server.ssl.key-store=classpath:/keystore/arrowhead.jks
   server.ssl.key-store-password=<your_password>
   server.ssl.key-store-type=JKS
   ```
4. Restart with `docker-compose up`.

---

## Adding or Disabling Modules

- **Add** a new core system by dropping its `module.properties` into `./properties/` and adding the matching volume & port mapping in `docker-compose.yml`.  
- **Disable** a module by removing its `.properties` file or commenting out its volume & port mapping.

---

## Contributing

1. Fork the repo and create your feature branch (`git checkout -b feature/foo`).  
2. Commit your changes (`git commit -am 'Add foo'`).  
3. Push to the branch (`git push origin feature/foo`).  
4. Open a Pull Request.

---

## License

This project is licensed under the [MIT License](LICENSE).
