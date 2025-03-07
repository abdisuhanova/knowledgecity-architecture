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

## 2.1 System Components

### Video Conversion/Encoding Microservice

**Stack:**
- AWS Elemental MediaConvert
- S3
- Lambda
- SQS/SNS
- EKS/Fargate
- CloudFront

**Description:**
- **AWS Elemental MediaConvert:** Serverless video processing service for format conversion and adaptive bitrate streaming.
- **S3:** Stores raw uploads (`S3 Standard`) and processed videos (`S3 Standard-Infrequent Access` for cost efficiency).
- **Lambda:** Automatically triggers encoding workflows upon S3 upload events.
- **SQS/SNS:** Manages asynchronous job queues and notifications between uploads, encoding, and delivery.
- **EKS/Fargate:** Orchestrates containerized encoding tasks for scalability.
- **CloudFront:** Delivers processed videos via a global CDN with regional edge caching.

### Additional Microservices

**Stack:**
- EKS/GKE
- Docker
- Helm
- API Gateway
- AWS App Mesh/Consul

**Description:**
- **EKS/GKE:** Kubernetes clusters for consistent deployment and scaling of future services.
- **Docker/Helm:** Standardizes packaging and deployment across teams.
- **API Gateway:** Manages API versioning, rate limiting, and authentication for new microservices.
- **Service Mesh (AWS App Mesh/Consul):** Enables service discovery, observability, and secure communication between microservices.

## 2.2 Data Distribution and Multi-Regional Setup

### Regional Data Residency
- **Saudi Arabia:** User data stored in AWS Middle East (Bahrain) region using Aurora Global Database (read replicas in multiple AZs).
- **United States:** User data stored in AWS US East (N. Virginia) with DynamoDB Global Tables for cross-region replication.

### Global Educational Content
- **Central Repository:** Stored in S3 (Standard class) in a primary region (e.g., US East).
- **Regional Caching:** Replicated to edge locations via CloudFront, with cache TTLs optimized for course updates.

### Multi-AZ Redundancy
- Databases deployed across 3 AZs per region.
- Microservices run in EKS clusters spanning 2+ AZs, with pod anti-affinity rules.

## 2.4 Media Processing and Delivery

### Upload Workflow
1. Users upload raw videos to regional S3 buckets (e.g., `US-East` for U.S. users).
2. S3 event triggers Lambda, which submits a job to AWS Elemental MediaConvert via SQS.

### Encoding
- MediaConvert processes videos into HLS/DASH formats for adaptive streaming.
- Processed videos stored in S3 with lifecycle policies to transition to cheaper storage classes after 30 days.

### Delivery
- CloudFront serves videos from edge locations, with geo-routing via Route 53.

### Cache Strategies
- Hot content cached at edge (24-hour TTL).
- Cold content fetched from origin with stale-while-revalidate headers.

## 2.5 Optimization and Scalability

### Autoscaling
- **EC2/ASG:** Scales PHP monolith instances based on CPU/RAM thresholds.
- **Karpenter (EKS):** Automatically provisions nodes for microservices during traffic spikes.
- **Lambda:** Serverless video encoding scales to zero when idle.

### Cost Optimization
- **Storage Tiering:** S3 Intelligent-Tiering for videos, Glacier Deep Archive for backups.
- **Spot Instances:** Used for non-critical batch jobs (e.g., analytics).

### Regional Expansion
- **Infrastructure as Code (IaC):** Terraform modules deploy repeatable stacks (VPC, EKS, RDS) to new regions.
- **Database Sharding:** Future-proofs user growth with horizontal partitioning.

## 3. Security, Observability, and Compliance

### Video Processing Security
- **Signed URLs (CloudFront):** Restrict access to processed videos.
- **S3 bucket policies:** Enforce encryption and block public access.

### Compliance
- **Data Residency:** AWS Config rules audit regional data storage.
- **GDPR/Regional Laws:** Encryption (AWS KMS) and tokenization for PII.

### Observability
- **Video Metrics:** CloudFront Real-Time Logs track buffer rates and playback errors.
- **ECS/EKS:** Prometheus exporters collect container-level metrics.

---

## Deployment

### Pre-requisites
- AWS CLI
- Terraform for IaC
- Docker for containerized services
- Helm for Kubernetes deployments
