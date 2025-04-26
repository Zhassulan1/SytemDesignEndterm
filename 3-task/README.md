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

![](internships.svg)