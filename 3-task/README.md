# 3-Task International internships.

### Functional requirements:

1. Registration
2. Profile and CV
3. Search and matching
4. Posting internships
5. Document verification
6. Checking applications to labor regulations

### Non-functional requirements

1. 50 000 DAU
2. 99.99 uptime
3. Security
4. API calls < 200 ms P95
5. 95% of pages load in < 2 s

### Capacity

| Component | Instances | CPU | RAM | Storage | Notes |
| --- | --- | --- | --- | --- | --- |
| **Web/API Servers** | 3 | 8 vCPU each | 32 GB each | 100 GB SSD each | 1 vCPU : 4 GB RAM ratio [starrocks.io](https://www.starrocks.io/blog/starrocks-best-practices-capacity-planning-and-deployment); handles ~150 RPS per instance (0.5–1 CPU/RPS) [kirshatrov.com](https://kirshatrov.com/posts/capacity-planning-for-web-apps) |
| **API Gateway** | 2 | 4 vCPU each | 8 GB each | 50 GB SSD each | Rate-limiting, routing |
| **Database Cluster** | 2 | 16 vCPU each | 64 GB each | 2 TB NVMe SSD each | Primary + async replica; supports ~500 TPS |
| **Cache (Redis)** | 3 | 4 vCPU each | 16 GB each | N/A | Session store, match-score caching |
| **Document Store** | 1 | N/A | N/A | 10 TB object store | S3-compatible for resumes, transcripts |
| **Logging & Metrics** | 1 | 8 vCPU | 32 GB | 5 TB SSD | ELK/Prometheus stack; ~100 GB logs/month |

**Growth plan:** add one web/API instance per +150 concurrent users; scale DB RAM by +32 GB per 1,000 TPS increase; expand object store by +1 TB/year.

```mermaid
graph TD
    subgraph "Client Layer"
        WebApp("Web Application")
        MobileApp("Mobile Application")
    end

    subgraph "API Gateway Layer"
        APIGateway("API Gateway")
        RateLimiter("Rate Limiter")
        Auth("Authentication & Authorization")
    end

    subgraph "Load Balancer Layer"
        LB("Load Balancer")
        VIP("Virtual IP")
    end

    subgraph "Service Layer"
        UserService("User Service")
        CompanyService("Company Service")
        InternshipService("Internship Service")
        MatchingService("Matching Service")
        ApplicationService("Application Service")
        DocumentService("Document Service")
        NotificationService("Notification Service")
        ComplianceService("Compliance Service")
        SearchService("Search Service")
        AnalyticsService("Analytics Service")
        PaymentService("Payment Service")
        RecommendationService("Recommendation Service")
    end

    subgraph "Message Broker"
        Kafka("Kafka Cluster")
        KafkaConsumers("Kafka Consumers")
    end

    subgraph "Cache Layer"
        Redis("Redis Cluster")
        CDN("Content Delivery Network")
    end

    subgraph "Database Layer"
        subgraph "Primary Databases"
            UserDB[("User DB")]
            CompanyDB[("Company DB")]
            InternshipDB[("Internship DB")]
            ApplicationDB[("Application DB")]
            DocumentDB[("Document DB")]
            ComplianceDB[("Compliance DB")]
        end
        
        subgraph "Read Replicas"
            UserDBReplica[("User DB Replica")]
            CompanyDBReplica[("Company DB Replica")]
            InternshipDBReplica[("Internship DB Replica")]
            ApplicationDBReplica[("Application DB Replica")]
            DocumentDBReplica[("Document DB Replica")]
            ComplianceDBReplica[("Compliance DB Replica")]
        end
        
        Elasticsearch[("Elasticsearch")]
    end

    subgraph "Background Processing"
        BatchProcessing("Batch Processing")
        DataETL("Data ETL")
    end

    %% Connections
    WebApp --> VIP
    MobileApp --> VIP
    VIP --> LB
    LB --> APIGateway
    APIGateway --> RateLimiter
    RateLimiter --> Auth
    Auth --> UserService
    Auth --> CompanyService
    Auth --> InternshipService
    Auth --> ApplicationService
    Auth --> DocumentService
    Auth --> ComplianceService

    UserService --> UserDB
    UserService --> UserDBReplica
    UserService --> Redis
    CompanyService --> CompanyDB
    CompanyService --> CompanyDBReplica
    CompanyService --> Redis
    InternshipService --> InternshipDB
    InternshipService --> InternshipDBReplica
    InternshipService --> Redis
    ApplicationService --> ApplicationDB
    ApplicationService --> ApplicationDBReplica
    DocumentService --> DocumentDB
    DocumentService --> DocumentDBReplica
    ComplianceService --> ComplianceDB
    ComplianceService --> ComplianceDBReplica

    SearchService --> Elasticsearch
    SearchService --> Redis

    MatchingService --> Redis
    MatchingService --> Kafka
    RecommendationService --> Redis
    RecommendationService --> Kafka

    NotificationService --> Kafka
    PaymentService --> Kafka
    AnalyticsService --> Kafka

    UserService --> Kafka
    CompanyService --> Kafka
    InternshipService --> Kafka
    ApplicationService --> Kafka
    DocumentService --> Kafka
    ComplianceService --> Kafka

    Kafka --> KafkaConsumers
    KafkaConsumers --> BatchProcessing
    KafkaConsumers --> DataETL
    KafkaConsumers --> NotificationService
    KafkaConsumers --> AnalyticsService

    BatchProcessing --> UserDB
    BatchProcessing --> CompanyDB
    BatchProcessing --> InternshipDB
    BatchProcessing --> ApplicationDB
    BatchProcessing --> DocumentDB
    BatchProcessing --> ComplianceDB

    SearchService --> Elasticsearch
    Elasticsearch --> InternshipDBReplica
    Elasticsearch --> CompanyDBReplica

    UserService --> CDN
    CompanyService --> CDN
    InternshipService --> CDN
```