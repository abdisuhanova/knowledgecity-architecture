# KnowledgeCity Platform Architecture

## 1. Overview

KnowledgeCity is a globally accessible educational platform designed for high availability, low latency, and scalability. The system consists of a front-end Single Page Application (SPA), a monolithic PHP backend, multiple microservices, and a multi-regional deployment strategy ensuring data residency compliance.

## 2. Architectural Design

### 2.1 System Components

#### Front-End:
- **SPA built with React or Svelte.**
- **Stack**: CloudFront, S3, Route 53, WAF.

##### Description:
- **CloudFront**: For low-latency access, AWS native service, global CDN, caching at edge locations.
- **S3**: Highly available and cost-effective, hosting the front-end side of the app.
- **Route 53**: Manages DNS routing, routes traffic to CloudFront based on location (Arabia or USA).
- **WAF**: Inspects incoming requests for malicious activities, blocks excessive requests, protects against DDoS attacks, SQL injection, and XSS.

#### Backend Services:

##### Monolithic PHP Application:
- **Core business logic**, hosted in multi-regional deployments.

##### Stack: 
- EC2, EKS (or ECS), ALB, S3, Lambda, API Gateway, SQS/SNS, ASG.

##### Description:
- **EC2**: Hosting monolith applications with low-level configuration.
- **ALB**: Balancing traffic to multiple app instances based on user location for storing data in Arabia or USA for high performance and availability.
- **ASG**: Autoscaling the number of instances based on traffic.
- **SQS**: Queues from the PHP app to ensure requests are not lost.
- **Lambda**: For high availability and performance, if the app can be deployed to Lambda.
- **API Gateway**: Used when the app is deployed in Lambda to expose APIs for backend services.

#### Analytics Microservice (ClickHouse-based):
- **Stack**: EKS, Amazon Glue, S3, Helm, Docker, Karpenter, Ingress, K8s service, ALB, HPA.

##### Description:
- **EKS**: Container orchestrator that is cloud-agnostic, highly scalable, and provides performance and availability.
- **Glue**: ETL processes and data transformation for analytics.
- **S3**: Cost-effective, scalable storage.
- **Helm**: Managing containerized apps, including networking, configuration management, and role controls.
- **Docker**: Containerizing the microservice.
- **Karpenter**: Node autoscaler based on traffic, ensuring high performance.
- **Ingress**: Exposing the analytics app outside the cluster.
- **K8s Service**: Creating ALB for the analytics app.
- **ALB**: Load balancing for analytics apps for performance and high availability.
- **HPA**: Horizontal Pod Autoscaling, scaling app pods based on traffic.

#### Video Conversion/Encoding Microservice:
- Converts videos into optimal formats.

#### Additional Microservices:
- Designed with containerized deployments to allow future expansion.

### 2.2 Multi-Regional Deployment

- **Saudi Arabian users**: Data resides in Saudi Arabia (AWS Middle East region).
- **U.S. users**: Data resides in the U.S. (AWS US region).
- **Educational content**: Cached globally via a CDN.

##### Multi-AZ Redundancy:
- Each region utilizes at least two availability zones for high availability (99.99% SLA).

### 2.3 Global Content Delivery

- **CDN** (e.g., AWS CloudFront, Fastly, or Cloudflare): For caching educational content globally.
- **Edge caching strategies**: Reduce latency for video playback.
- **Multi-tier caching** (Redis/Memcached): Improves backend response times.

### 2.4 Scalability

- **Autoscaling Groups**: For backend services.
- **Kubernetes (EKS/GKE) or ECS/Fargate**: For orchestrating containerized microservices.
- **Horizontal Scaling**: Based on traffic spikes.
- **Cost Optimization**: Through tiered storage and on-demand compute scaling.

### 2.5 Infrastructure Components

| Component               | Technology Choice         | Justification                                                |
|-------------------------|---------------------------|--------------------------------------------------------------|
| **Cloud Provider**       | AWS/GCP                   | Global coverage, compliance, and reliability.                |
| **Container Orchestration** | Kubernetes (EKS/GKE) or ECS/Fargate | Simplifies microservices management, autoscaling.             |
| **CDN**                  | AWS CloudFront/Fastly     | Low latency, global caching.                                 |
| **Database**             | Amazon RDS (PostgreSQL/MySQL) | Regional compliance, high availability.                       |
| **Analytics**            | ClickHouse                | Efficient analytical queries.                                |
| **Caching**              | Redis/Memcached           | Improves response times.                                     |
| **Object Storage**       | S3/Google Cloud Storage   | Cost-effective for video storage.                            |

## 3. Security, Observability, and Compliance

### 3.1 Security Measures

- **Access Control**: IAM roles, RBAC, least privilege principle.
- **DDoS Protection**: AWS Shield/Cloudflare DDoS mitigation.
- **WAF Usage**: Web Application Firewall for filtering malicious traffic.
- **Encryption**:
  - **Data at rest**: Encrypted using AES-256.
  - **Data in transit**: TLS 1.2+ enforced.
- **Secrets Management**: AWS Secrets Manager/HashiCorp Vault.

### 3.2 Observability (Monitoring, Logging, Tracing)

- **Monitoring Tools**: Prometheus, Grafana for real-time metrics.
- **Logging Stack**: EFK (Elasticsearch, Fluentd, Kibana) or Loki for structured logging.
- **Tracing**: OpenTelemetry for distributed tracing.

##### Critical Metrics:
- API response times, error rates, CPU/memory utilization.
- CDN cache hit ratios, video streaming performance.

### 3.3 Compliance & Regulatory Adaptation

- **Data Residency Laws**: Ensuring Saudi Arabian data stays in Saudi Arabia.
- **Anomaly Detection**: AI-based monitoring for unusual activity (e.g., AWS GuardDuty).
- **Future Adaptability**: Infrastructure as Code (Terraform/CDK) to modify deployments as regulations change.

## 4. Conclusion

This architecture ensures high availability, low latency, compliance with data regulations, and future scalability while optimizing costs. The use of multi-regional deployments, CDNs, container orchestration, and security best practices guarantees a robust and efficient educational platform.
