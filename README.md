# ASDM Mainframe Modernizer Sample

> **[ASDM](https://asdm.ai)** (AI-First System Development Methodology) is a methodology that places AI at the core of the software development lifecycle — accelerating development, deployment, and maintenance of intelligent systems.

A sample project demonstrating mainframe-to-cloud modernization using the ASDM Mainframe Modernizer toolset. This repository contains both the legacy IBM CICS Banking Sample Application and its modernized cloud-native counterpart as git submodules.

## Repository Structure

```
asdm-mainframe-modernizer-sample/
├── cics-banking-sample-application-cbsa/   ← Legacy (submodule)
├── modern-cics-banking-sample-application-cbsa/  ← Modern (submodule)
└── README.md
```

| Submodule | Description | Repo |
|-----------|-------------|------|
| `cics-banking-sample-application-cbsa` | Legacy IBM CICS Banking Sample Application — COBOL/CICS/Db2/VSAM on z/OS | [ups216/cics-banking-sample-application-cbsa](https://github.com/ups216/cics-banking-sample-application-cbsa) |
| `modern-cics-banking-sample-application-cbsa` | Modernized cloud-native banking app — Spring Boot/React/PostgreSQL on x86 Linux | [ups216/modern-cics-banking-sample-application-cbsa](https://github.com/ups216/modern-cics-banking-sample-application-cbsa) |

## Modernization Overview

| Aspect | Before (Mainframe) | After (Modern) |
|--------|-------------------|----------------|
| **Runtime** | CICS Transaction Server on z/OS | Spring Boot 3.x on x86 Linux |
| **Business Logic** | 29 COBOL Programs | Java 17 / Spring Services |
| **Data Structures** | 37 COBOL Copybooks | JPA Entities / DTOs |
| **UI** | 9 BMS Mapsets (3270 Terminal) | React + TypeScript + Ant Design SPA |
| **Database** | Db2 + VSAM KSDS | PostgreSQL |
| **Batch** | 102 JCL Scripts | Spring Batch + GitHub Actions |
| **Integration** | z/OS Connect EE | Spring Cloud Gateway |
| **Deployment** | z/OS LPAR | Docker / Kubernetes |

## Getting Started

### Clone with Submodules

```bash
git clone --recurse-submodules git@github.com:ups216/asdm-mainframe-modernizer-sample.git
```

If you already cloned without submodules:

```bash
git submodule init
git submodule update
```

### Modern Application Quick Start

See [modern-cics-banking-sample-application-cbsa/README.md](modern-cics-banking-sample-application-cbsa/README.md) for full details.

```bash
cd modern-cics-banking-sample-application-cbsa

# Build backend
mvn clean package

# Build frontend
cd frontend && npm install && npm run build

# Run with Docker Compose
docker-compose up
```

## Architecture

### Before — Mainframe (z/OS)

```mermaid
graph TB
    subgraph "User Interfaces"
        BMS["BMS 3270 Terminal<br/>COBOL/BMS"]
        CR["Carbon React UI<br/>React + IBM Carbon"]
        CS["Customer Services<br/>Spring Boot + Thymeleaf"]
        PAY["Payment Interface<br/>Spring Boot + Thymeleaf"]
    end

    subgraph "API / Integration Layer"
        JAX["JAX-RS REST API<br/>/banking/*<br/>Liberty JVM Server"]
        ZOS["z/OS Connect EE<br/>10 APIs / 10 Services"]
    end

    subgraph "CICS Transaction Server"
        COBOL["COBOL Programs<br/>29 Programs"]
        CICSAPI["CICS API<br/>LINK / Commarea"]
    end

    subgraph "Data Layer"
        DB2["Db2 V12+<br/>ACCOUNT, CONTROL, PROCTRAN"]
        VSAM["VSAM KSDS<br/>CUSTOMER, ABNDFILE"]
    end

    BMS -->|CICS terminal| CICSAPI
    CR -->|HTTP REST| JAX -->|CICS LINK / JDBC| COBOL
    JAX -->|JDBC| DB2
    JAX -->|JCICS| VSAM
    CS -->|Spring WebClient| ZOS -->|CICS Commarea| CICSAPI
    PAY -->|Spring WebClient| ZOS
    CICSAPI --> COBOL
    COBOL -->|SQL| DB2
    COBOL -->|VSAM I/O| VSAM
```

### After — Cloud-Native (x86 Linux)

```mermaid
graph TB
    subgraph "Client"
        BROWSER["Browser<br/>React SPA"]
    end

    subgraph "Ingress"
        GW["API Gateway<br/>Spring Cloud Gateway<br/>:8080"]
    end

    subgraph "Microservices — Spring Boot 3.x"
        ACCT["account-service<br/>:8081"]
        CUST["customer-service<br/>:8082"]
        TXN["transaction-service<br/>:8083"]
        BATCH["batch-service<br/>:8084"]
    end

    subgraph "Data — PostgreSQL :5432"
        DB1["bank_account<br/>bank_control"]
        DB2["bank_customer"]
        DB3["bank_proctran"]
    end

    subgraph "Infrastructure"
        DOCKER["Docker / Kubernetes"]
        CICD["GitHub Actions CI/CD"]
    end

    BROWSER -->|HTTP /api/v1/*| GW
    GW --> ACCT
    GW --> CUST
    GW --> TXN
    GW --> BATCH

    ACCT --> DB1
    CUST --> DB2
    TXN --> DB3
    BATCH --> DB1
    BATCH --> DB3

    ACCT -.->|Saga/Event| TXN
    CUST -.->|Saga/Event| ACCT

    DOCKER --- ACCT
    DOCKER --- CUST
    DOCKER --- TXN
    DOCKER --- BATCH
    CICD -.-> DOCKER
```

## COBOL-to-Java Mapping

| COBOL Program | Microservice | REST Endpoint |
|---------------|-------------|---------------|
| CREACC | account-service | `POST /api/v1/accounts` |
| INQACC | account-service | `GET /api/v1/accounts/{sortcode}/{number}` |
| UPDACC | account-service | `PUT /api/v1/accounts/{sortcode}/{number}` |
| DELACC | account-service | `DELETE /api/v1/accounts/{sortcode}/{number}` |
| CRECUST | customer-service | `POST /api/v1/customers` |
| INQCUST | customer-service | `GET /api/v1/customers/{sortcode}/{number}` |
| XFRFUN | transaction-service | `POST /api/v1/transactions/transfer` |
| DBCRFUN | transaction-service | `POST /api/v1/transactions/debit-credit` |

## BMS-to-React Mapping

| BMS Map | React Page | Route |
|---------|-----------|-------|
| BNK1MAI (Main Menu) | HomePage | `/` |
| BNK1CAM (Create Account) | CreateAccountPage | `/accounts/create` |
| BNK1CCM (Create Customer) | CreateCustomerPage | `/customers/create` |
| BNK1TFM (Transfer Funds) | TransferPage | `/transactions/transfer` |
| BNK1DAM (Delete Account) | DeleteAccountPage | `/accounts/delete` |
| BNK1DCM (Delete Customer) | DeleteCustomerPage | `/customers/delete` |
| BNK1UAM (Update Account) | UpdateAccountPage | `/accounts/update` |

## License

This project is provided as a sample for demonstration purposes. The legacy CBSA code is based on the [IBM CICS Banking Sample Application](https://github.com/IBM/cics-banking-sample-application-cbsa).
