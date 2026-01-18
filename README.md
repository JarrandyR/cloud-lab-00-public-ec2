# cloud-lab-00-public-ec2

Hands-on AWS cloud security lab demonstrating how public compute exposure
combined with over-permissioned identity access can lead to cloud-wide risk.

## Overview

This lab simulates a common real-world cloud misconfiguration scenario:
a publicly accessible virtual machine with excessive identity permissions.

The environment is intentionally designed to:
- Expose cloud infrastructure to the internet
- Demonstrate identity-based attack paths
- Validate the impact of instance metadata credential access
- Apply remediation aligned with cloud security best practices

## Key Focus Areas
- Cloud networking fundamentals (VPC, subnets, routing)
- Internet-facing resource exposure
- IAM role abuse via Instance Metadata Service (IMDS)
- Security hardening and attack surface reduction
