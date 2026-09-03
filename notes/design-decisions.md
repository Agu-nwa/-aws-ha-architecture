# Architecture Design Decisions

## VPC CIDR

The VPC uses:

10.0.0.0/16

A /16 network provides a large private IP address space that can be divided
into multiple smaller subnets.

This allows the architecture to create separate public and private subnets
across multiple Availability Zones while leaving room for future expansion.

## Internet Gateway

Created and attached an Internet Gateway (`aws-ha-igw`) project VPC.
This connects public subnets to internet.
