# Lab Walkthrough — Cloud Lab 00 (Public EC2 Exposure)

## Step 1 — Cost Guardrails & Threat Modeling

Before deploying any cloud infrastructure, cost controls were put in place.
Unmonitored cloud environments are a security risk due to potential abuse
(e.g., cryptomining, resource hijacking, denial-of-wallet attacks).

![AWS Budget Guardrail](../screenshots/01-budget-guardrail.png)

### Security Relevance
Cloud security includes financial abuse prevention.
Attackers often abuse misconfigured accounts to run unauthorized workloads.
### Initial Threat Model

Before infrastructure existed, the following risks were considered:

- Unrestricted cloud spending
- Unmonitored resource creation
- Abuse of compute resources

These risks were mitigated by implementing cost guardrails before deployment.
