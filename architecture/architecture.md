# Architecture

```mermaid
flowchart TD

    U[users]

    U--> ALB[Application Load Balancer]
    ALB --> EC2A[EC2 Instance - AZ A]
    ALB --> EC2B[EC2 Instance - AZ B]

    EC2A --> RDS[(Amazon RDS)]  
    EC2B --> RDS
    
    EC2A --> S3[Amazon S3]
    EC2B --> S3
