# Architecture Overview — Cloud Lab 00

This lab deploys a minimal AWS environment consisting of:

- A Virtual Private Cloud (VPC)
- A public subnet
- An internet gateway
- A publicly reachable EC2 instance
- Security group–based network controls

The goal is to understand how attack surface is introduced as infrastructure becomes reachable.

## Network Design

- VPC CIDR: 10.0.0.0/16
- Public Subnet: 10.0.1.0/24
- Internet Gateway attached
- Default route (0.0.0.0/0) enabled

This design intentionally introduces a public network boundary for analysis.

## Remediated Architecture

The final architecture removes direct public exposure, enforces
IMDSv2 protections, applies least-privilege IAM, and restricts
administrative access.

Attack paths demonstrated earlier are no longer viable.
