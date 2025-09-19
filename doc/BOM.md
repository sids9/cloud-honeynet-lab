
---

## What bills in this design

- **AWS VPC 1**
  - 2 EC2 instances t3 small
  - 2 Elastic IPs
  - 1 Internet Gateway and routing
  - 1 Security Group
  - VPC Flow Logs to CloudWatch Logs
  - 1 S3 bucket for parquet archives

- **Azure VPC 2**
  - 2 VMs B2s
  - 2 Public IPs
  - 1 NSG
  - Azure Monitor Log Analytics workspace
  - 1 Blob Storage container for parquet
  - Optional Azure Firewall if you centralize egress policy

- **GCP VPC 3**
  - 2 Compute Engine VMs e2 standard 2
  - 2 External IPs
  - 1 VPC firewall rule set
  - VPC Flow Logs to Cloud Logging
  - 1 GCS bucket for parquet

- **Central stack**
  - 1 Ingest VM small
  - 1 CTI worker VM small or combined with ingest
  - 1 OpenSearch or Elasticsearch VM single node
  - 1 Object storage bucket for long term parquet
  - Optional scheduled job for rule mining
  - Budget alerts service and auto shutdown logic

> Outbound from sensors is deny except to ingest endpoint and allowed mirrors or ntp.  
> No peering to production. Each VPC or project is isolated.  

