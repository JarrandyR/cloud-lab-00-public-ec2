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

Cloud-Lab-00 Architecture (What We’re Showing)

This lab is about one core mistake and its impact.
So the architecture should reflect clarity, not complexity.

Architecture Logic

One VPC

One public subnet

One internet-facing EC2

One IAM role (intentionally risky)

Clear attack path

Clear remediation point

Conceptual Diagram (Simple + Correct)



<img width="370" height="479" alt="image" src="https://github.com/user-attachments/assets/40b0dd4c-52c2-4d8a-8728-fddb3f61fe47" />


