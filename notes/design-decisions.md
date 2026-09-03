# Architecture Design Decisions

## VPC CIDR

The VPC uses:

`10.0.0.0/16`

A /16 network provides a large private IP address space that can be divided
into multiple smaller subnets.

This allows the architecture to create separate public and private subnets
across multiple Availability Zones while leaving room for future expansion.

## Internet Gateway

Created and attached an Internet Gateway (`aws-ha-igw`) to the project VPC.

The Internet Gateway provides connectivity between the VPC and the public
internet.

Attaching an Internet Gateway alone does not make a subnet public. A subnet
must also be associated with a route table containing a route to the
Internet Gateway.

## Public and Private Subnet Routing

The architecture uses four subnets distributed across two Availability Zones:

| Tier | sa-east-1a | sa-east-1b |
|---|---|---|
| Public | `10.0.1.0/24` | `10.0.2.0/24` |
| Private | `10.0.11.0/24` | `10.0.12.0/24` |

A dedicated route table (`aws-ha-rt`) is associated with the two public
subnets.

The public route table contains:

- `10.0.0.0/16 → local`
- `0.0.0.0/0 → Internet Gateway`

The private subnets remain associated with the VPC main route table and do
not have a direct route to the Internet Gateway.

This separates internet-facing resources from resources intended to remain
private.

## Target Group

Created an EC2 target group named `aws-ha-tg`.

The target group uses:

- Target type: EC2 instances
- Protocol: HTTP
- Port: 80
- Protocol version: HTTP/1.1
- Health check protocol: HTTP
- Health check path: `/`

The target group provides the backend destination for traffic forwarded
by the Application Load Balancer.

Instances are not registered manually. The Auto Scaling Group will
automatically register and deregister EC2 instances as application
capacity changes.

Health checks allow the load balancer to identify unhealthy application
instances and avoid routing normal traffic to them.

## Application Load Balancer

Created an internet-facing Application Load Balancer named `aws-ha-alb`.

The ALB is deployed across the two public subnets in `sa-east-1a` and
`sa-east-1b`, reducing dependency on a single Availability Zone.

The load balancer uses `aws-ha-alb-sg`, which controls inbound web traffic.

An HTTP listener on port 80 forwards incoming requests to `aws-ha-tg`.

The target group will contain EC2 instances managed by the Auto Scaling
Group. This allows the ALB to distribute requests across multiple
application instances and avoid routing normal traffic to unhealthy targets.

## Auto Scaling Group

Created an Auto Scaling Group named `aws-ha-asg` using version 2 of
the `aws-ha-lt` launch template.

The Auto Scaling Group is configured with:

- Minimum capacity: 2
- Desired capacity: 2
- Maximum capacity: 4
- Target tracking metric: Average CPU utilization
- Target utilization: 70%
- Health checks: EC2 and ELB
- Health check grace period: 300 seconds

The Auto Scaling Group spans public subnets in `sa-east-1a` and
`sa-east-1b`.

A minimum and desired capacity of two maintains two application
instances during normal operation. The current deployment has one
instance running in each Availability Zone.

The maximum capacity of four allows the application tier to scale
out when demand increases.

The Auto Scaling Group is integrated with `aws-ha-tg`, allowing
instances to be automatically registered and deregistered from the
load-balancing layer.
