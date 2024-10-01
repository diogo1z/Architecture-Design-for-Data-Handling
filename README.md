# Microservice Design: API for Data Handling

## Project Overview

This microservice provides two primary functionalities:
- `POST /data`: Accepts and validates JSON data before storing it in a database.
- `GET /data`: Retrieves stored data with fast read performance.

The microservice is built with TypeScript (Node.js) and follows a scalable, performant architecture designed to handle millions of data records efficiently.

## Technology Stack
- **Language**: TypeScript (Node.js)
- **Base Framework**: NestJS
- **Database**: PostgreSQL
- **Caching**: Redis for enhanced read performance
- **Containerization**: Docker
- **Orchestration**: Kubernetes for scalability and deployment
- **CI/CD Pipeline**: GitHub Actions

---

## Code Structure

The code is organized into multiple layers to ensure clear separation of concerns and modularity. Here's the directory structure:
```tree
.
└── src/
    ├── e2e-tests/
    │   └── data.test.ts
    └── domain/
        └── data/
            ├── data.controller.ts
            ├── data.controller.spec.ts
            ├── data.service.ts
            ├── data.service.spec.ts
            ├── data.repository.ts
            ├── data.repository.spec.ts
            ├── entity/
            │   ├── data.entity.ts
            │   └── data.entity.test.ts
            └── events/
                └── update-redis/
                    ├── update-redis.handler.ts
                    └── update-redis.handler.test.ts
```
### Explanation of Components:
- **Controller**: The entry point for API requests. It processes incoming HTTP requests, validates them, and forwards them to the service layer.
- **Service**: Contains the business logic, processes data, and interacts with the repository layer.
- **Repository**: Handles all database interactions.
- **Entity**: Defines the data structure used by the application.
- **Events**: This handles event-based actions, such as updating Redis cache.
- **End-to-End Tests**: Tests that cover the full application flow, simulating a real scenarios.

Each component also has its corresponding test file to ensure robust unit testing and maintain code quality.


## Setup Instructions

### Prerequisites
1. [Docker](https://docs.docker.com/get-docker/) installed.
2. A Kubernetes cluster.
3. Install the Kubernetes CLI (`kubectl`).

### Local Development Setup
1. Clone the repository:
    ```bash
    git clone https://github.com/diogo1z/microservice-design.git
    cd microservice-design
    ```
2. Install dependencies:
    ```bash
    npm install
    ```

3. Run the service locally:
    ```bash
    npm run start
    ```

### Docker Setup
1. Build the Docker image:
    ```bash
    docker build -t microservice-api .
    ```
2. Run the container:
    ```bash
    docker run -p 3000:3000 microservice-api
    ```

---

## Build, Test, and Deploy Instructions

### Build
To build the microservice locally:
```bash
npm run build
```

### Testing
To build the microservice locally:
```bash
npm run test
```


---

### 2. Architecture
This document will explain the architecture in more depth, including diagrams that describe the flow of data and scalability considerations.

## Microservice Architecture

### 1. API Layer (Controller)
- **Responsibility**:
  - Handles incoming HTTP requests (`POST /data` and `GET /data`).
  - Validates the incoming request data and forwards it to the service layer.
  - Constructs and returns appropriate HTTP responses based on the results from the service layer.


### 2. Service Layer
- **Responsibility**:
  - Contains the core business logic of the application.
  - Processes data, applies business rules, and communicates with the repository for data persistence.
  - Performs any necessary data transformation before saving or retrieving it from the database.
  - Manages communication with the **Repository** for CRUD operations and **Event Handlers** for cache updates.

### 3. Repository Layer
- **Responsibility**:
  - Manages interactions with the database (e.g., PostgreSQL or MongoDB).
  - Performs Create, Read, Update, and Delete (CRUD) operations.
  - Abstracts database access, allowing for easier modifications if the underlying database changes in the future.


### 4. Entity Layer
- **Responsibility**:
  - Defines the structure and schema of the data used in the application.
  - For SQL databases like PostgreSQL, it defines tables and fields, while for NoSQL databases like MongoDB, it represents collections and documents.

### 5. Event-Driven Architecture (Redis Cache Update)
- **Responsibility**:
  - Handles event-driven actions, such as updating Redis cache when data is modified in the database.
  - The `update-redis.handler.ts` listens for relevant events and ensures that the cache is updated, improving read performance.


### 6. End-to-End Testing
- **Responsibility**:
  - Contains end-to-end tests to simulate real scenarios.
  - These tests verify that all the components (controller, service, repository, events) work together correctly and that the full flow of data, from API to persistence, functions as expected.

### Project Diagram
![Project Diagram](./diagrams/project-diagram.png)

---

### Scalability and Performance Considerations:

1. **Horizontal Scaling**:
   - The service is designed for deployment in a Kubernetes cluster, allowing horizontal scaling by adding more instances of the microservice.
   - This ensures that the system can handle increased load by distributing traffic across multiple pods.

2. **Caching**:
   - Redis is integrated for caching frequently accessed data, reducing the need for repeated database reads and improving performance.
   - The cache is kept up to date using an event-driven approach, where changes to the data automatically trigger cache updates.

---

### Request Flow:

1. **POST /data**:
    - The controller receives the incoming data request, validates it, and forwards it to the service layer.
    - The service layer processes and validates the data and uses the repository to store it in the database.
    - An event is triggered to update the Redis cache, ensuring the latest data is available for future reads.

2. **GET /data**:
    - The controller handles the incoming request for data.
    - The service checks the Redis cache for the requested data.
    - If the data is found in the cache, it is returned directly. If not, the service tries to retrieves the data from the Postgres db.

### Request Flow Diagram
![Request Flow Diagram](./diagrams/request-flow.png)

---

## Performance Considerations

- **Horizontal Scaling**: Achieved using Kubernetes, which scales pods based on traffic.
- **Caching**: Redis is used to cache frequently accessed data, ensuring sub-500ms response times.
- **Database Sharding**: For PostgreSQL, horizontal scaling through sharding ensures data is split across nodes to handle large data volumes.


# CI/CD Pipeline

## Overview

The CI/CD pipeline automates the process of building, testing, and deploying the microservice. It uses GitHub Actions for continuous integration and deployment.

### Key Steps:
1. **Code Integration**: On each push or pull request, GitHub Actions triggers a workflow.
2. **Automated Tests**: Unit tests are executed to verify core functionality.
3. **Build Docker Image**: The service is containerized, and the Docker image is pushed to the registry.
4. **E2E Tests**: E2E tests are executed to verify the service is working properly.
5. **Kubernetes Deployment**: The latest image is deployed to a Kubernetes cluster using `kubectl`.

### Pipeline Diagram
![CI/CD Pipeline Diagram](./diagrams/cicd-flow.png)

---

## GitHub Actions Workflow

The following actions are performed in the pipeline:
- **Unit Testing**: Verifies that the code works correctly.
- **Linting**: Ensures code quality and follows best practices.
- **Docker Build & Push**: Builds the Docker image and pushes it to a container registry.
- **E2E Tests**: Ensures the service as all it's working correctly.
- **Kubernetes Deployment**: Deploys the Docker image to a Kubernetes cluster.


# Kubernetes Deployment

## Overview

The microservice will be deployed in a Kubernetes environment using the following resources:
- **Deployment**: Manages replicas and ensures the service is running.
- **Service**: Exposes the service internally to other components or externally via Ingress.
- **Ingress**: Provides access to the service via an HTTPs endpoint.

### Kubernetes Manifests

- **Deployment**: Defines the number of replicas and container images.
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: data-api
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: data-api
      template:
        metadata:
          labels:
            app: data-api
        spec:
          containers:
          - name: api-container
            image: data-api:{{commitSHA}}
            ports:
            - containerPort: 3000
    ```

- **Service**: Exposes the data microservice.
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: data-api
    spec:
      type: ClusterIP
      selector:
        app: data-api
      ports:
      - protocol: TCP
        port: 80
        targetPort: 3000
    ```

- **Ingress**: Exposes the service to external traffic.
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: api-ingress
    spec:
      rules:
      - host: data.domain.com
        http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: data-api
                port:
                  number: 80
    ```
