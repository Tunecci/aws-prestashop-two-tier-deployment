# AWS PrestaShop Two-Tier Architecture

The deployment separates the PrestaShop application and its database across two Amazon EC2 instances.

```mermaid
flowchart TD
    A[Internet / Web Browser] -->|HTTP - Port 80| B[Application EC2]
    B --> C[Apache HTTP Server]
    C --> D[PHP-FPM / PHP]
    D --> E[PrestaShop 9.2.1]
    E -->|Private VPC Connection - TCP 3306| F[Database EC2]
    F --> G[MariaDB 10.11]
    G --> H[(PrestaShop Database)]

    style B fill:#dbeafe,stroke:#2563eb
    style F fill:#dcfce7,stroke:#16a34a
```

## Network Design

- The application server handles public web requests.
- Apache and PHP process the PrestaShop application.
- The database server runs separately from the application server.
- PrestaShop connects to MariaDB through the private VPC network.
- Database access is restricted to the application server's Security Group.
- SSH access is restricted to the administrator's IP address.

## Infrastructure

| Tier | EC2 Instance | Main Services |
|---|---|---|
| Application | PrestaShop-App-Server | Apache, PHP, PrestaShop |
| Database | PrestaShop-Database-Server | MariaDB |

**AWS Region:** eu-north-1 (Stockholm)