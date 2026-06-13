# E-Commerce Microservices Platform

<div align="center">

![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)
![Java](https://img.shields.io/badge/Java-21%20LTS-orange.svg)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.2.12-green.svg)
![Spring Cloud](https://img.shields.io/badge/Spring%20Cloud-2023.0.2-brightgreen.svg)
![Build Status](https://img.shields.io/badge/build-passing-brightgreen.svg)
![License](https://img.shields.io/badge/license-MIT-blue.svg)

A production-ready e-commerce platform built with microservices architecture and the Saga pattern for distributed transaction management.

[Features](#-features) • [Architecture](#-architecture) • [Quick Start](#-quick-start) • [Documentation](#-documentation) • [Contributing](#-contributing)

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Features](#-features)
- [Architecture](#-architecture)
- [Microservices](#-microservices)
- [Technology Stack](#-technology-stack)
- [Project Structure](#-project-structure)
- [Quick Start](#-quick-start)
- [API Endpoints](#-api-endpoints)
- [Saga Pattern Flow](#-saga-pattern-flow)
- [Development](#-development)
- [Testing](#-testing)
- [Deployment](#-deployment)
- [Documentation](#-documentation)
- [Contributing](#-contributing)
- [License](#-license)

---

## 🎯 Overview

This project implements a **robust, scalable e-commerce system** using modern microservices architecture patterns. It demonstrates enterprise-level design principles including:

- **Microservices Architecture**: Independent, loosely-coupled services
- **Distributed Transactions**: Saga pattern for data consistency
- **Event-Driven Communication**: Apache Kafka for asynchronous messaging
- **Service Discovery**: Netflix Eureka for dynamic service registration
- **API Gateway**: Spring Cloud Gateway for request routing
- **Polyglot Persistence**: PostgreSQL and MongoDB for different use cases

The system handles complex order processing workflows while maintaining data consistency across multiple independent services without distributed locks or global transactions.

---

## ✨ Features

### Core Functionality
- ✅ **Order Management**: Create, track, and manage customer orders
- ✅ **Inventory Management**: Real-time stock management and reservation
- ✅ **Payment Processing**: Secure payment handling with compensation
- ✅ **Shipping & Logistics**: Shipment tracking and delivery management
- ✅ **Customer Notifications**: Multi-channel notification delivery
- ✅ **Service Discovery**: Dynamic service registration and health checks

### Technical Features
- ✅ **Distributed Transactions**: Saga pattern with choreography
- ✅ **Event Sourcing**: Kafka-based event streaming
- ✅ **Resilience**: Circuit breakers and fallback mechanisms
- ✅ **Monitoring**: Actuator endpoints for service health
- ✅ **Containerization**: Docker support for all services
- ✅ **API Gateway**: Centralized request routing and cross-cutting concerns
- ✅ **Java 21 LTS**: Latest long-term support Java version

---

## 🏗️ Architecture

### High-Level Architecture Diagram

The architecture follows the microservices pattern with the following components:

```mermaid
graph TD
    Client[Client Applications] --> ApiGateway[API Gateway]
    ApiGateway --> OrderService[Order Service]
    ApiGateway --> InventoryService[Inventory Service]
    ApiGateway --> PaymentService[Payment Service]
    ApiGateway --> ShippingService[Shipping Service]
    ApiGateway --> NotificationService[Notification Service]
    
    OrderService -- Events --> Kafka[Kafka]
    InventoryService -- Events --> Kafka
    PaymentService -- Events --> Kafka
    ShippingService -- Events --> Kafka
    NotificationService -- Events --> Kafka
    
    Kafka -- Events --> OrderService
    Kafka -- Events --> InventoryService
    Kafka -- Events --> PaymentService
    Kafka -- Events --> ShippingService
    Kafka -- Events --> NotificationService
    
    OrderService --> OrderDB[(PostgreSQL)]
    InventoryService --> InventoryDB[(MongoDB)]
    PaymentService --> PaymentDB[(PostgreSQL)]
    ShippingService --> ShippingDB[(PostgreSQL)]
    NotificationService --> NotificationDB[(MongoDB)]
    NotificationService --> Redis[(Redis)]
    
    ServiceRegistry[Service Registry] --> OrderService
    ServiceRegistry --> InventoryService
    ServiceRegistry --> PaymentService
    ServiceRegistry --> ShippingService
    ServiceRegistry --> NotificationService
```🎛️ Microservices

### 1. **Order Service** (Port: 8081)
**Responsibility**: Order lifecycle management and saga orchestration

**Key Features**:
- Create and manage customer orders
- Initiate saga workflows
- Track order status through saga stages
- Handle order compensation (cancellations)

**Database**: PostgreSQL

---

### 2. **Inventory Service** (Port: 8082)
**Responsibility**: Product stock management and reservation

**Key Features**:
- Manage product inventory levels
- Reserve stock during order processing
- Release reservations on compensation
- Provide inventory availability checks

**Database**: MongoDB

---

### 3. **Payment Service** (Port: 8083)
**Responsibility**: Payment processing and refund management

**Key Features**:
- Process customer payments
- Handle payment refunds
- Track payment status
- Maintain payment history

**Database**: PostgreSQL

---

### 4. **Shipping Service** (Port: 8084)
**Responsibility**: Shipment creation and delivery tracking

**Key Features**:
- Create shipments for orders
---

## 🔄 Saga Pattern Flow

### Successful Order Flow

```
1. Customer places order
   └─> Order Service creates order (status: PENDING)
       └─> Publishes: OrderCreatedEvent

2. Inventory Service receives OrderCreatedEvent
   └─> Checks product availability
   └─> Reserves inventory
       └─> Publishes: InventoryReservedEvent

3. Payment Service receives InventoryReservedEvent
   └─> Processes payment
   └─> Updates payment status
       └─> Publishes: PaymentProcessedEvent

4. Shipping Service receives PaymentProcessedEvent
   └─> Creates shipment
   └─> Generates tracking number
       └─> Publishes: ShipmentCreatedEvent

5. Order Service receives ShipmentCreatedEvent
   └─> Updates order status to COMPLETED

6. Notification Service receives ShipmentCreatedEvent
   └─> Sends confirmation email to customer
```

### Compensation Flow (on Failure)

```
If Payment Fails:
  ├─> Payment Service publishes: PaymentFailedEvent
  ├─> Inventory Service receives event
  │   └─> Releases reserved inventory
  ├─> Order Service receives event
  │   └─> Updates order status to FAILED
  └─> Notification Service
      └─> Sends failure notification to customer
```
## 📦 Technology Stack

### Core Framework
| Technology | Version | Purpose |
|-----------|---------|---------|
| Java | 21 LTS | Programming language |
| Spring Boot | 3.2.12 | Application framework |
| Spring Cloud | 2023.0.2 | Microservices toolkit |

### Data & Caching
| Technology | Purpose |
|-----------|---------|
| PostgreSQL | Relational database (Orders, Payments, Shipping) |
| MongoDB | NoSQL database (Inventory, Notifications) |
| Redis | In-memory cache and session store |

### Message Queue
| Technology | Purpose |
|-----------|---------|
| Apache Kafka | Event streaming and asynchronous communication |

### Build & Deployment
| Technology | Purpose |
|-----------|---------|
| Maven 3.9.13 | Build and dependency management |
| Docker | Containerization |
| Docker Compose | Container orchestration |ata storage
- **Docker & Docker Compose**: Containerization and orchestration
- **Maven**: Build and dependency management

## Saga Pattern Implementation

This project implements the Saga pattern using a choreography-based approach:

### Order Processing Flow:

1. **Order Creation**:
    - Customer places an order
    - Order service creates an order with PENDING status
    - Order service publishes OrderCreatedEvent

2. **Inventory Reservation**:
    - Inventory service consumes OrderCreatedEvent
    - Checks product availability
    - Reserves inventory if available
    - Publishes InventoryReservedEvent or InventoryReservationFailedEvent

3. **Payment Processing**:
    - Payment service consumes InventoryReservedEvent
    - Processes payment
    - Publishes PaymentProcessedEvent or PaymentFailedEvent

4. **Shipping Creation**:
    - Shipping service consumes PaymentProcessedEvent
    - Creates shipping record
    - Publishes ShipmentProcessedEvent or ShipmentFailedEvent

5. **Order Completion**:
    - Order service updates order status to COMPLETED

### Compensation Transactions:

If any step fails, the system executes compensation transactions to maintain consistency:

- **Payment Failure**: Inventory service releases reserved inventory
- **Shipping Failure**: Payment service refunds payment, Inventory service releases inventory
- **Notification Service**: Informs the customer about transaction status (success/failure)

## 📁 Project Structure

```
ecommerce-microservices/
│
├── pom.xml                              # Parent POM - defines versions
├── common-library/                      # Shared utilities module
├── service-registry/                    # Eureka Service Discovery
├── api-gateway/                         # Spring Cloud Gateway
├── order-service/                       # Order Management Service
├── inventory-service/                   # Inventory Management Service
├── payment-service/                     # Payment Processing Service
├── notification-service/                # Notification Service
├── shipping-service/                    # Shipping Management Service
├── scripts/                             # Utility scripts
└── README.md                            # This file
```

---

## 🚀 Quick Start

### Prerequisites

- Java 21 JDK or later
- Maven 3.9+
- Docker & Docker Compose

### Step 1: Clone the Repository

```bash
git clone https://github.com/hacisimsek/ecommerce-microservices.git
cd ecommerce-microservices
```

### Step 2: Build the Project

```bash
mvn clean package -DskipTests
```

### Step 3: Start Infrastructure

```bash
docker-compose up -d
```

### Step 4: Run Services

```bash
# Terminal 1: Service Registry
cd service-registry && mvn spring-boot:run

# Terminal 2: API Gateway
cd api-gateway && mvn spring-boot:run

# Terminal 3+: Other Services
# (Similar commands for other services)
```

### Step 5: Verify Services

```bash
# Check Eureka Dashboard
curl http://localhost:8761

# Check API Gateway Health
curl http://localhost:8080/actuator/health
```

---

## 📡 API Endpoints

### Order Service (Port: 8081)
```http
POST   /api/orders              - Create new order
GET    /api/orders/{id}         - Get order details
PUT    /api/orders/{id}         - Update order
DELETE /api/orders/{id}         - Cancel order
GET    /api/orders              - List all orders
```

### Inventory Service (Port: 8082)
```http
GET    /api/inventory/{id}      - Check stock level
POST   /api/inventory/reserve   - Reserve inventory
POST   /api/inventory/release   - Release reservation
PUT    /api/inventory/{id}      - Update stock
GET    /api/inventory           - List inventory
```

### Payment Service (Port: 8083)
```http
POST   /api/payments            - Process payment
GET    /api/payments/{id}       - Get payment details
POST   /api/payments/{id}/refund - Refund payment
GET    /api/payments            - List payments
```

### Shipping Service (Port: 8084)
```http
POST   /api/shipments           - Create shipment
GET    /api/shipments/{id}      - Get shipment details
PUT    /api/shipments/{id}/status - Update status
GET    /api/shipments           - List shipments
```

### Notification Service (Port: 8085)
```http
POST   /api/notifications       - Send notification
GET    /api/notifications/{id}  - Get notification status
GET    /api/notifications       - List notifications
```

---

## 🔧 Development

### Build Commands

```bash
# Clean and build all modules
mvn clean package

# Build specific module
mvn -pl order-service clean package

# Run specific module
mvn -pl order-service spring-boot:run

# Skip tests during build
mvn clean package -DskipTests
```

### Testing

```bash
# Run all tests
mvn test

# Run tests for specific module
mvn -pl order-service test

# Run integration tests
mvn verify
```

---

## 🧪 Testing

```bash
# Unit tests
mvn test

# Integration tests
mvn verify

# Specific test class
mvn test -Dtest=OrderServiceApplicationTests

# Generate coverage report
mvn clean test jacoco:report
```

---

## 📦 Deployment

### Docker Compose

```bash
# Build and start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down
```

### Production Deployment

- Use managed databases (AWS RDS, Azure Database)
- Use managed Kafka (AWS MSK, Confluent Cloud)
- Implement monitoring (Prometheus, Grafana)
- Setup distributed tracing (Zipkin, Jaeger)
- Use secrets management (Vault, AWS Secrets Manager)

---

## 📚 Documentation

### Architecture & Design
- [Microservices Pattern](https://microservices.io/)
- [Saga Pattern Guide](https://microservices.io/patterns/data/saga.html)
- [Spring Cloud Docs](https://spring.io/projects/spring-cloud)

### Guides
- [Spring Boot Reference](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)
- [Spring Cloud Gateway](https://spring.io/guides/gs/gateway/)
- [Apache Kafka Docs](https://kafka.apache.org/documentation/)

---

## 🤝 Contributing

1. Fork the repository
2. Create feature branch: `git checkout -b feature/amazing-feature`
3. Commit changes: `git commit -m 'Add amazing feature'`
4. Push branch: `git push origin feature/amazing-feature`
5. Open Pull Request

### Code Standards
- Follow Google Java Style Guide
- Write unit tests for features
- Update documentation
- Keep commits atomic

---

## 📄 License

MIT License - see LICENSE file for details

---

## 👨‍💻 Author

**Hacisimsek** - [@hacisimsek](https://github.com/hacisimsek)

---

<div align="center">

### ⭐ If you found this helpful, please give it a star!

[⬆ back to top](#e-commerce-microservices-platform)

</div>
