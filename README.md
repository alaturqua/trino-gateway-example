# Trino Gateway

## Overview
The Trino Gateway is a high-availability gateway for Trino clusters. It provides routing, query history, and resource management for multiple Trino backends.

## Architecture

```mermaid
graph LR
    subgraph "Query Layer"
        TG[Trino Gateway<br/>:8080]
        T1[Trino 1<br/>:8081]
        T2[Trino 2<br/>:8082]
    end

    subgraph "Metadata Layer"
        HMS1[Hive Metastore<br/>:9083]
        PG[(PostgreSQL<br/>:5432)]
    end

    subgraph "Storage Layer"
        MINIO[MinIO<br/>:9000/9001]
        ICE[/Iceberg Tables/]
        DELTA[/Delta Tables/]
        HIVE[/Hive Tables/]
    end

    %% Connections
    TG -->|Load Balance| T1
    TG -->|Load Balance| T2
    
    T1 -.->|Metadata| HMS1
    T2 -.->|Metadata| HMS1
    
    HMS1 -->|Store Metadata| PG
    
    T1 -->|Read/Write Data| MINIO
    T2 -->|Read/Write Data| MINIO
    
    MINIO -->|Store| ICE
    MINIO -->|Store| DELTA
    MINIO -->|Store| HIVE

    classDef gateway fill:#f96,stroke:#333,color:#000
    classDef query fill:#58f,stroke:#333,color:#000
    classDef meta fill:#5f5,stroke:#333,color:#000
    classDef storage fill:#fa1,stroke:#333,color:#000
    
    class TG gateway
    class T1,T2, query
    class HMS1,PG meta
    class MINIO,ICE,DELTA,HIVE storage
```

## Prerequisites
- Docker
- Docker Compose

## Running the Trino Gateway
1. **Build the Docker images:**
   ```sh
   docker-compose build
   ```

2. **Start the services:**
   ```sh
   docker-compose up -d
   ```

3. **Verify the services:**
   - Trino Gateway: [http://localhost:8080](http://localhost:8080)
   - Trino-1: [http://localhost:8080](http://localhost:8080)
   - Trino-2: [http://localhost:8082](http://localhost:8082)
   - PostgreSQL: [http://localhost:5432](http://localhost:5432)
   - Hive Metastore: [thrift://localhost:9083](thrift://localhost:9083)

## Configuration

### Gateway Configuration
The gateway configuration is located in `gateway-ha-config.yml`. It includes settings for server configuration, data store, authentication, and authorization.

### Certificates
The public and private keys for authentication are located in the `certs` directory.

### Logging
Logging configuration is defined in `log.properties`.

## Database Initialization
The PostgreSQL database is initialized with the scripts located in the `trino-gateway` directory:
- `init-user-db.sh`
- `create-multiple-postgresql-databases.sh`

## Health Checks
Health checks are configured for all services in the `docker-compose.yml` file to ensure they are running correctly.

## Stopping the Services
To stop the services, run:
```sh
docker-compose down
```
