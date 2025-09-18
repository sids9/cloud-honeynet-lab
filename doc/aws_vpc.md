# Multi-Cloud Honeypot VPC/VNet Architecture

## AWS — VPC ap-south-1

```mermaid
flowchart TB
  subgraph AWS["VPC 10.0.0.0/24 aws ap-south-1"]
    direction TB
    IGW[Internet Gateway]
    RT[Route Table public to IGW]
    NACL[NACL optional]
    PUB_SUB[Public Subnet 10.0.0.0/26]
    HPA1[Honeypot VM1 ENI Public IP Docker cowrie opencanary dionaea]
    HPA2[Honeypot VM2 ENI Public IP]
    SG_HP[Security Group honeypot Ingress 22 23 80 443 21 445 3389 3306 27017 6379 53udp Source 0.0.0.0/0 Egress DENY except collector CIDR]
    ENCRYPTED_LOGS[VPC Flow Logs to CloudWatch Logs or S3]
    S3_ARCH[Object Storage S3 Parquet]
    COLLECTOR[Central Collector IP CIDR Receiver TLS endpoint]
    BASTION[Optional Bastion Jump admin only Strict SSH MFA]

    IGW --- RT
    RT --- PUB_SUB
    PUB_SUB --- HPA1
    PUB_SUB --- HPA2
    HPA1 --- SG_HP
    HPA2 --- SG_HP
    HPA1 -->|ship logs TLS only| COLLECTOR
    HPA2 -->|ship logs TLS only| COLLECTOR
    PUB_SUB --> ENCRYPTED_LOGS
    ENCRYPTED_LOGS --> S3_ARCH
    BASTION -.-> PUB_SUB
  end
