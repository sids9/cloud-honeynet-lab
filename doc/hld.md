# High-Level Honeypot Architecture

```mermaid
flowchart LR
  subgraph AWS["AWS VPC ap-south-1"]
    A1[Honeypot VM1 - Public IP]
    A2[Honeypot VM2 - Public IP]
    SG_A[SG Ingress: SSH, RDP, Web, DB<br/>Egress: DENY except collector]
    A1 --- SG_A
    A2 --- SG_A
  end

  subgraph AZ["Azure VNet Central India"]
    Z1[Honeypot VM1 - Public IP]
    Z2[Honeypot VM2 - Public IP]
    NSG_AZ[NSG Ingress rules<br/>Egress: DENY except collector]
    Z1 --- NSG_AZ
    Z2 --- NSG_AZ
  end

  subgraph GCP["GCP VPC asia-south1"]
    G1[Honeypot VM1 - Public IP]
    G2[Honeypot VM2 - Public IP]
    FW_GCP[Firewall rules<br/>Egress: DENY except collector]
    G1 --- FW_GCP
    G2 --- FW_GCP
  end

  subgraph COLLECTOR["Central Analytics"]
    C1[Log Ingest + CTI Enrichment]
    C2[OpenSearch / Elasticsearch]
    C3[Object Storage Parquet Archive]
    C4[Rule Mining Sigma Suricata YARA]
    C1 --> C2
    C1 --> C3
    C2 --> C4
  end

  A1 -->|TLS logs| C1
  A2 -->|TLS logs| C1
  Z1 -->|TLS logs| C1
  Z2 -->|TLS logs| C1
  G1 -->|TLS logs| C1
  G2 -->|TLS logs| C1
