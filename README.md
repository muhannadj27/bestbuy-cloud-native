# Best Buy Cloud-Native Application

## CST8915 — Full Stack Cloud Development (Final Project)

**Student:** Mohannad Jaber  
**Course:** CST8915 — Winter 2026  
**Professor:** Ramy Mohamed

## Demo Video

[YouTube Demo Video](https://youtu.be/EdQwInCxByk))

## Application Overview

This is a cloud-native microservices application built for Best Buy as a demo e-commerce platform. The application allows customers to browse products and place orders, while employees can manage products and view orders through an admin dashboard. The system uses an event-driven architecture with RabbitMQ for message queuing and MongoDB for persistent data storage.

## Architecture Diagram

![Architecture Diagram](architecture-diagram.png)

```
                         ┌──────────────┐
                    ┌───▸│ order-service │───────┐
                    │    │   (Node.js)   │       │
┌───────────┐       │    └──────────────┘       ▼
│ store-front│───────┤                     ┌───────────┐
│  (Vue.js)  │       │    ┌──────────────┐ │  RabbitMQ │
└───────────┘       ├───▸│product-service│ │  (Queue)  │
                    │    │    (Rust)     │ └─────┬─────┘
                    │    └──────────────┘       │
                    │                           ▼
┌───────────┐       │    ┌────────────────┐  ┌──────────┐
│store-admin │───────┘───▸│makeline-service│─▸│ MongoDB  │
│  (Vue.js)  │            │     (Go)       │  │(Database)│
└───────────┘            └────────────────┘  └──────────┘
```

### Data Flow

1. **Customers** browse products via the **Store Front** (Vue.js)
2. Orders are sent to **Order Service** (Node.js), which pushes them to **RabbitMQ**
3. **Makeline Service** (Go) consumes orders from RabbitMQ and stores them in **MongoDB**
4. **Store Admin** (Vue.js) allows employees to manage products via **Product Service** (Rust) and view orders via **Makeline Service**

## Microservices

| Service | Description | Technology | GitHub Repo | Docker Hub Image |
|---------|-------------|------------|-------------|-----------------|
| Store Front | Customer web app | Vue.js | [bestbuy-store-front](https://github.com/muhannadj27/bestbuy-store-front) | [muhannadj27/bestbuy-store-front](https://hub.docker.com/r/muhannadj27/bestbuy-store-front) |
| Store Admin | Employee web app | Vue.js | [bestbuy-store-admin](https://github.com/muhannadj27/bestbuy-store-admin) | [muhannadj27/bestbuy-store-admin](https://hub.docker.com/r/muhannadj27/bestbuy-store-admin) |
| Order Service | Order processing API | Node.js | [bestbuy-order-service](https://github.com/muhannadj27/bestbuy-order-service) | [muhannadj27/bestbuy-order-service](https://hub.docker.com/r/muhannadj27/bestbuy-order-service) |
| Product Service | Product management API | Rust | [bestbuy-product-service](https://github.com/muhannadj27/bestbuy-product-service) | [muhannadj27/bestbuy-product-service](https://hub.docker.com/r/muhannadj27/bestbuy-product-service) |
| Makeline Service | Order processing worker | Go | [bestbuy-makeline-service](https://github.com/muhannadj27/bestbuy-makeline-service) | [muhannadj27/bestbuy-makeline-service](https://hub.docker.com/r/muhannadj27/bestbuy-makeline-service) |
| MongoDB | Database | MongoDB 4.2 | — | [mongo:4.2](https://hub.docker.com/_/mongo) |
| RabbitMQ | Message Queue | RabbitMQ 3 | — | [rabbitmq:3-management](https://hub.docker.com/_/rabbitmq) |

## Deployment Instructions

### Prerequisites

- Azure CLI installed
- kubectl installed
- Docker Desktop installed
- Active Azure subscription

### 1. Login to Azure

```bash
az login
```

### 2. Create Resource Group

```bash
az group create --name bestbuyRG --location westus3
```

### 3. Create AKS Cluster

```bash
az aks create --resource-group bestbuyRG --name bestbuyAKS --node-count 2 --generate-ssh-keys --location westus3 --node-vm-size Standard_B2s_v2
```

### 4. Connect to the Cluster

```bash
az aks get-credentials --resource-group bestbuyRG --name bestbuyAKS
kubectl get nodes
```

### 5. Deploy the Application

```bash
cd "Deployment Files"
kubectl apply -f bestbuy-all-in-one.yaml
```

### 6. Verify Deployment

```bash
kubectl get pods
kubectl get services
```

### 7. Access the Application

| App | URL |
|-----|-----|
| Store Front | `http://<STORE_FRONT_EXTERNAL_IP>` |
| Store Admin | `http://<STORE_ADMIN_EXTERNAL_IP>` |

## CI/CD Pipeline

Each microservice has a GitHub Actions workflow that automatically builds and pushes a new Docker image to Docker Hub on every push to the `main` branch.

### Pipeline Steps

1. **Checkout** — pulls the latest code
2. **Login to Docker Hub** — authenticates using repository secrets
3. **Build and Push** — builds the Docker image and pushes it to Docker Hub

### Setting Up CI/CD

For each service repository, add these GitHub secrets:
- `DOCKER_USERNAME` — your Docker Hub username
- `DOCKER_PASSWORD` — your Docker Hub access token

The workflow file is located at `.github/workflows/ci-cd.yml` in each service repository.

## Cleanup

```bash
az group delete --name bestbuyRG --yes --no-wait
```
