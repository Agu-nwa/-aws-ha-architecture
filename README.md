# Highly Available AWS Web Architecture

## Project Overview

This project demonstrates the design and deployment of a highly available,
scalable web application architecture on AWS.

The architecture distributes application servers across multiple Availability
Zones and uses AWS managed services to improve availability, scalability,
performance, and reliability.

## Objectives

- Deploy application servers across multiple Availability Zones
- Distribute traffic using an Application Load Balancer
- Automatically scale EC2 capacity based on demand
- Deploy a managed relational database
- Store static assets in Amazon S3
- Eliminate single points of failure
- Apply AWS security best practices

## AWS Services

- Amazon VPC
- Amazon EC2
- Application Load Balancer
- Auto Scaling
- Amazon RDS
- Amazon S3
- IAM
- Security Groups

## Network Design

The application runs inside a dedicated Amazon VPC.

| Resource | Configuration |
|---|---|
| VPC | saa-ha-project-vpc |
| CIDR | 10.0.0.0/16 |
| Region | To be documented |
| Availability Zones | 2 |
