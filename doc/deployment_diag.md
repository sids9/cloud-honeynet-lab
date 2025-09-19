# Cloud Honeypot Deployment Diagram

```mermaid
flowchart LR

  %% =================== VPC 1 AWS ===================
  subgraph VPC1_AWS["VPC 1 AWS ap-south-1 10.0.0.0/24"]
    direction TB
    IGW1[Internet Gateway]
    RT1[Route Table public to IGW]
    SUB1[Public Subnet 10.0.0.0/26]

    EC2A[Honeypot VM1 t3 small]
    EIPA[Elastic IP for VM1]
    EC2B[Honeypot VM2 t3 small]
    EIPB[Elastic IP for VM2]
    SG1[Security Group honeypot]
    VPCFLOW1[VPC Flow Logs]
    CW1[CloudWatch Logs]
    S3A[S3 bucket archive parquet]

    IGW1 --- RT1
    RT1 --- SUB1
    SUB1 --- EC2A
    SUB1 --- EC2B
    EC2A --- SG1
    EC2B --- SG1
    EIPA --- EC2A
    EIPB --- EC2B
    SUB1 --> VPCFLOW1
    VPCFLOW1 --> CW1
    CW1 --> S3A
  end

  %% =================== VPC 2 Azure ===================
  subgraph VPC2_AZURE["VPC 2 Azure Central India 10.1.0.0/24"]
    direction TB
    SUB2[Public Subnet 10.1.0.0/26]
    VM2A[Honeypot VM1 B2s]
    PIP2A[Public IP for VM1]
    VM2B[Honeypot VM2 B2s]
    PIP2B[Public IP for VM2]
    NSG2[Network Security Group honeypot]
    LA2[Azure Monitor Log Analytics]
    BLOB2[Blob Storage parquet]
    AZFW2[Azure Firewall optional egress policy]

    SUB2 --- VM2A
    SUB2 --- VM2B
    VM2A --- NSG2
    VM2B --- NSG2
    PIP2A --- VM2A
    PIP2B --- VM2B
    SUB2 --> LA2
    LA2 --> BLOB2
    AZFW2 -.-> SUB2
  end

  %% =================== VPC 3 GCP ===================
  subgraph VPC3_GCP["VPC 3 GCP asia-south1 10.2.0.0/24"]
    direction TB
    SUB3[Public Subnet 10.2.0.0/26]
    GVM1[Honeypot VM1 e2 standard 2]
    GIP1[External IP for VM1]
    GVM2[Honeypot VM2 e2 standard 2]
    GIP2[External IP for VM2]
    FW3[VPC Firewall rules honeypot]
    VPCFLOW3[VPC Flow Logs]
    GLOG3[Cloud Logging]
    GCS3[GCS bucket parquet]

    SUB3 --- GVM1
    SUB3 --- GVM2
    GVM1 --- FW3
    GVM2 --- FW3
    GIP1 --- GVM1
    GIP2 --- GVM2
    SUB3 --> VPCFLOW3
    VPCFLOW3 --> GLOG3
    GLOG3 --> GCS3
  end

  %% =================== Central SIEM and Ingest ===================
  subgraph CENTRAL["Central stack choose any one cloud"]
    direction TB
    INGEST[Log ingest receiver TLS small VM]
    CTI[CTI enrich worker small VM]
    ES[OpenSearch or Elasticsearch VM 8 vcpu 32 gb ram]
    OBJ[Object Storage parquet archive]
    RULES[Rule mining Sigma Suricata YARA job]
    BUDGET[Budget alerts and auto shutdown guards]

    INGEST --> ES
    INGEST --> OBJ
    CTI --> ES
    CTI --> OBJ
    ES --> RULES
    BUDGET -.-> ES
    BUDGET -.-> INGEST
  end

  %% =================== One way log flow from sensors to ingest ===================
  EC2A -->|TLS logs| INGEST
  EC2B -->|TLS logs| INGEST
  VM2A  -->|TLS logs| INGEST
  VM2B  -->|TLS logs| INGEST
  GVM1  -->|TLS logs| INGEST
  GVM2  -->|TLS logs| INGEST
