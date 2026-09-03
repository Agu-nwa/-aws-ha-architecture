# AWS Highly Available Web Architecture

```mermaid
flowchart TD

    USER[Internet Users]

    subgraph AWS["AWS Cloud - sa-east-1"]

        IGW[Internet Gateway]

        subgraph VPC["aws-ha-vpc - 10.0.0.0/16"]

            subgraph PUBLIC["Public Tier"]

                ALB[Application Load Balancer<br/>aws-ha-alb]

            end

            TG[Target Group<br/>aws-ha-tg]

            subgraph ASG["Auto Scaling Group - aws-ha-asg<br/>Min: 2 | Desired: 2 | Max: 4"]

                subgraph AZA["Availability Zone - sa-east-1a"]
                    EC2A[EC2 Instance<br/>Nginx Web Server]
                end

                subgraph AZB["Availability Zone - sa-east-1b"]
                    EC2B[EC2 Instance<br/>Nginx Web Server]
                end

            end

            subgraph DATA["Database Tier"]
                RDS[(Amazon RDS<br/>PostgreSQL)]
            end

        end

        S3[Amazon S3]

    end

    USER --> IGW
    IGW --> ALB
    ALB --> TG

    TG --> EC2A
    TG --> EC2B

    EC2A --> RDS
    EC2B --> RDS

    EC2A --> S3
    EC2B --> S3
```

## Traffic Flow

1. Users send HTTP requests to the Application Load Balancer.
2. The ALB receives internet traffic across multiple Availability Zones.
3. The ALB forwards requests to `aws-ha-tg`.
4. The target group distributes requests across healthy EC2 instances.
5. EC2 instances are managed by `aws-ha-asg`.
6. The Auto Scaling Group maintains at least two instances and can scale to four.
7. Application instances communicate privately with Amazon RDS PostgreSQL.
8. RDS is not publicly accessible.
9. Application instances can use Amazon S3 for object storage.

## High Availability

The architecture avoids reliance on a single EC2 instance or Availability Zone.

The Auto Scaling Group distributes application instances across:

- `sa-east-1a`
- `sa-east-1b`

If an application instance fails, the Application Load Balancer stops routing
traffic to the unhealthy target while the Auto Scaling Group launches a
replacement instance.

## Scaling

The Auto Scaling Group is configured with:

| Setting | Value |
|---|---:|
| Minimum capacity | 2 |
| Desired capacity | 2 |
| Maximum capacity | 4 |
| Scaling metric | Average CPU utilization |
| Target | 70% |

This allows the application tier to automatically increase or decrease capacity
based on demand.

## Security

The architecture uses separate security groups for each tier.

- The ALB accepts public web traffic.
- EC2 instances accept application traffic from the ALB security group.
- RDS is not publicly accessible.
- PostgreSQL access should be restricted to the application security group.

This creates the intended traffic path:

`Internet → ALB → EC2 → RDS`

rather than exposing the EC2 application or database directly to the internet.
