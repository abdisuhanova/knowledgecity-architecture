### **System Design Document (SDD)**  
**Centralized Logging Solution with EKS, Fluentd, Elasticsearch, and Kibana**  

---

### **1. Introduction**  
**Purpose**:  
The system is designed to provide a scalable, fault-tolerant logging solution for applications running on Amazon EKS. It aggregates logs from distributed services, processes them via Fluentd, stores them in Elasticsearch, and enables visualization/analysis through Kibana.  

**Key Objectives**:  
- Centralize log collection across EKS clusters.  
- Enable real-time log analysis and troubleshooting.  
- Ensure scalability for high log volumes.  
- Secure access to logs and infrastructure.  

---

### **2. Architecture Overview**  
**Diagram Highlights**:  
![Architecture Diagram](link_to_your_diagram_here)  
1. **EKS Clusters**: Host containerized applications emitting logs.  
2. **Fluentd (DaemonSet)**: Deployed on each EKS node to collect logs.  
3. **Elasticsearch**: Central log storage with indexing and search capabilities.  
4. **Kibana**: Visualization layer for querying and dashboarding.  
5. **Terraform**: Infrastructure-as-Code (IaC) for provisioning AWS resources.  

**Data Flow**:  
1. Apps → Fluentd → Elasticsearch → Kibana (see [Section 4](#4-data-flow) for details).  

---

### **3. Components**  
#### **Amazon EKS Clusters**  
- **Role**: Managed Kubernetes service for running containerized apps.  
- **Key Features**: Auto-scaling, integration with AWS services (IAM, VPC).  

#### **Fluentd**  
- **Role**: Log collector/aggregator deployed as a DaemonSet.  
- **Configuration**:  
  - Parses logs from `/var/log/containers`.  
  - Filters/enriches logs (e.g., add Kubernetes metadata).  
  - Outputs to Elasticsearch.  

#### **Elasticsearch**  
- **Role**: Distributed search and analytics engine for log storage.  
- **Settings**:  
  - 3-node cluster (1 master, 2 data nodes) for fault tolerance.  
  - Index lifecycle management (ILM) for log retention policies.  

#### **Kibana**  
- **Role**: UI for log visualization, dashboards, and ad-hoc queries.  
- **Features**: Prebuilt dashboards for Kubernetes logs.  

#### **Terraform**  
- **Role**: Automate provisioning of AWS resources (EKS, VPC, IAM).  
- **Modules**: `eks-cluster`, `elasticsearch`, `fluentd-config`.  

---

### **4. Data Flow**  
1. **Application Logs**:  
   - Apps write logs to `stdout/stderr` (captured by Docker/containerd).  
   - Logs stored at `/var/log/containers` on EKS nodes.  

2. **Fluentd Collection**:  
   - Fluentd DaemonSet tails log files, adds metadata (pod name, namespace).  
   - Logs buffered locally (to handle network outages).  

3. **Elasticsearch Ingestion**:  
   - Fluentd forwards logs to Elasticsearch via HTTP/HTTPS.  
   - Logs indexed by timestamp/namespace for efficient querying.  

4. **Kibana Visualization**:  
   - Users query Elasticsearch via Kibana’s Discover/Dashboard interfaces.  

---

### **5. Scalability & Fault Tolerance**  
- **Elasticsearch**:  
  - Scale horizontally by adding data nodes.  
  - Use shard replication (1 primary + 1 replica shard per index).  
- **Fluentd**:  
  - Buffer logs to disk during Elasticsearch outages.  
  - Deploy as DaemonSet to ensure per-node log collection.  
- **EKS**:  
  - Cluster Autoscaler adjusts node count based on workload.  

---

### **6. Security & IAM Roles**  
- **EKS Nodes**: IAM role with permissions for ECR, CloudWatch, and SSM.  
- **Fluentd**: IAM role allowing write access to Elasticsearch domain.  
- **Elasticsearch**:  
  - TLS encryption for data-in-transit.  
  - Role-Based Access Control (RBAC) via Kibana.  
- **Kibana**: AWS Cognito integration for user authentication.  
- **Terraform**: Least-privilege IAM roles for resource provisioning.  

---

### **7. Technology Stack**  
| **Component**       | **Version/Configuration**                   | **Justification**                                      |  
|----------------------|---------------------------------------------|--------------------------------------------------------|  
| Kubernetes (EKS)     | 1.27                                        | AWS-managed, reduces operational overhead.             |  
| Fluentd              | 1.16 + `fluent-plugin-elasticsearch`       | Lightweight, Kubernetes-native, and extensible.        |  
| Elasticsearch        | 8.9 (OpenSearch)                            | Managed service (Amazon OpenSearch), built-in security.|  
| Kibana               | 8.9                                         | Tight integration with Elasticsearch.                  |  
| Terraform            | 1.5+ with AWS provider 5.0                  | Stable version with EKS/OpenSearch module support.     |  

**Why Fluentd over Logstash?**  
- Lower resource footprint.  
- Better suited for Kubernetes log aggregation with DaemonSet pattern.  

**Why EKS over Self-Managed K8s?**  
- Automated patching, scaling, and integration with AWS services.  

---

### **8. Project Roadmap**  
| **Phase**                     | **Tasks**                                   | **Timeframe** |  
|-------------------------------|---------------------------------------------|---------------|  
| **Terraform Setup**           | Define VPC, IAM roles, S3 backend.         | 1 week        |  
| **EKS Cluster Deployment**    | Deploy cluster, node groups, and add-ons.   | 2 weeks       |  
| **Fluentd Log Pipeline**      | Configure DaemonSet, filters, Elasticsearch.| 1 week        |  
| **Elasticsearch & Kibana**    | Deploy OpenSearch domain, set up Kibana.   | 1 week        |  
| **CI/CD Automation**          | GitHub Actions for Terraform/Fluentd.       | 1 week        |  
| **Testing & Validation**      | Load testing, security audits.             | 1 week        |  

**Total Estimated Time**: 6 weeks  

---

### **9. Deliverables**  
1. **System Design Document (SDD)**: This document.  
2. **Wireframes**: Kibana dashboard mockups (e.g., log search, pod error rates).  
3. **Technology Stack Docs**: Version matrix, IaC templates, Fluentd configs.  
4. **Project Roadmap**: Detailed timeline with milestones.  

---

**Next Steps**:  
- Finalize architecture diagram annotations.  
- Review Terraform modules for security best practices.  
- Draft Kibana dashboard wireframes for stakeholder approval.
