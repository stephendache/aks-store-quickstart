# AKS Store Quickstart

This is a Kubernetes manifest file (`aks-store-quickstart.yml`) that deploys a complete microservices-based e-commerce store application on Azure Kubernetes Service (AKS).

## Overview

The application consists of four main services that work together to provide a functional online store:

1. **RabbitMQ** - Message queue for inter-service communication
2. **Order Service** - Handles order processing
3. **Product Service** - Manages product catalog
4. **Store Front** - Vue.js-based user interface

## Technology Stack

This deployment uses the following technologies and frameworks:

- **Message Queue**: RabbitMQ 3.10 with management plugin
- **Backend Framework**: Fastify (used by Order Service)
- **Frontend Framework**: Vue.js (used by Store Front)
- **Runtime**: Node.js (for Order and Product services)
- **Container Platform**: Docker (all services containerized)
- **Orchestration**: Kubernetes / Azure Kubernetes Service (AKS)
- **Service Mesh**: Service discovery via Kubernetes DNS
- **Monitoring**: Prometheus metrics (RabbitMQ with Prometheus plugin)

### Application Types

- **Microservices Architecture**: Loosely coupled services communicating via message queues
- **Backend APIs**: RESTful APIs for order and product management
- **Frontend SPA**: Single Page Application built with Vue.js
- **Message-Driven**: Asynchronous communication between services using RabbitMQ
- **Cloud-Native**: Containerized, scalable Kubernetes deployment

## Architecture

### Components

#### 1. RabbitMQ Message Broker
- **Container Image**: `mcr.microsoft.com/mirror/docker/library/rabbitmq:3.10-management-alpine`
- **Ports**:
  - `5672` - AMQP protocol (message queue)
  - `15672` - Management UI (HTTP)
- **Default Credentials**: username / password
- **Enabled Plugins**: Management, Prometheus metrics, AMQP 1.0 support
- **Resources**:
  - CPU Request: 10m, Limit: 250m
  - Memory Request: 128Mi, Limit: 256Mi

#### 2. Order Service
- **Container Image**: `ghcr.io/azure-samples/aks-store-demo/order-service:latest`
- **Port**: 3000 (HTTP)
- **Dependencies**: RabbitMQ
- **Features**:
  - Connects to RabbitMQ for queue management
  - Startup wait for RabbitMQ using netcat health check
  - Runs on Fastify framework
- **Resources**:
  - CPU Request: 1m, Limit: 75m
  - Memory Request: 50Mi, Limit: 128Mi
- **Init Container**: Waits for RabbitMQ to be ready before starting

#### 3. Product Service
- **Container Image**: `ghcr.io/azure-samples/aks-store-demo/product-service:latest`
- **Port**: 3002 (HTTP)
- **Purpose**: Serves product information and catalog
- **Resources**:
  - CPU Request: 100m, Limit: 500m
  - Memory Request: 128Mi, Limit: 256Mi

#### 4. Store Front (Web UI)
- **Container Image**: `ghcr.io/azure-samples/aks-store-demo/store-front:latest`
- **Port**: 8080 (HTTP, exposed as 80)
- **Framework**: Vue.js
- **Service Type**: LoadBalancer (publicly accessible)
- **Configuration**:
  - Connects to Order Service at `http://order-service:3000/`
  - Connects to Product Service at `http://product-service:3002/`
- **Resources**:
  - CPU Request: 1m, Limit: 1000m
  - Memory Request: 200Mi, Limit: 512Mi

## Kubernetes Resources

### Deployments
- **rabbitmq** (1 replica)
- **order-service** (1 replica)
- **product-service** (1 replica)
- **store-front** (1 replica)

### Services
- **rabbitmq** - ClusterIP service for internal messaging
- **order-service** - ClusterIP service for internal order processing
- **product-service** - ClusterIP service for internal product queries
- **store-front** - LoadBalancer service for external access

### ConfigMap
- **rabbitmq-enabled-plugins** - Configures which RabbitMQ plugins are enabled

## Deployment

### Prerequisites
- Azure Kubernetes Service (AKS) cluster
- `kubectl` configured to access your cluster

### Deploy the Application
```bash
kubectl apply -f aks-store-quickstart.yml
```

### Verify Deployment
```bash
# Check all deployments
kubectl get deployments

# Check all services
kubectl get services

# Check pods
kubectl get pods
```

### Access the Store Front
1. Get the external IP of the store-front service:
```bash
kubectl get service store-front
```

2. Open your browser and navigate to the external IP address

## Environment Variables

### Order Service
- `ORDER_QUEUE_HOSTNAME` - RabbitMQ hostname
- `ORDER_QUEUE_PORT` - RabbitMQ AMQP port
- `ORDER_QUEUE_USERNAME` - RabbitMQ username
- `ORDER_QUEUE_PASSWORD` - RabbitMQ password
- `ORDER_QUEUE_NAME` - Queue name for orders
- `FASTIFY_ADDRESS` - Fastify server bind address

### Store Front
- `VUE_APP_ORDER_SERVICE_URL` - Order service endpoint
- `VUE_APP_PRODUCT_SERVICE_URL` - Product service endpoint

## Resource Management

All services have resource requests and limits configured to ensure proper scheduling and prevent resource exhaustion:

| Service | CPU Request | CPU Limit | Memory Request | Memory Limit |
|---------|------------|-----------|----------------|--------------|
| RabbitMQ | 10m | 250m | 128Mi | 256Mi |
| Order Service | 1m | 75m | 50Mi | 128Mi |
| Product Service | 100m | 500m | 128Mi | 256Mi |
| Store Front | 1m | 1000m | 200Mi | 512Mi |

## Notes

- All pods are configured to run on Linux nodes
- RabbitMQ management UI is available at `rabbitmq:15672`
- Default RabbitMQ credentials should be changed for production
- The order service includes an init container that waits for RabbitMQ to be ready
- Store Front service is exposed as a LoadBalancer for public access
