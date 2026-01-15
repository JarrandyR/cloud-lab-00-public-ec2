# Lab Walkthrough — Cloud Lab 00 (Public EC2 Exposure)

## Step 1 — Cost Guardrails & Threat Modeling

Before deploying any cloud infrastructure, cost controls were put in place.
Unmonitored cloud environments are a security risk due to potential abuse
(e.g., cryptomining, resource hijacking, denial-of-wallet attacks).
Action — Set a Cost Budget (If Not Done Yet)
In AWS Console:

Use the top search bar → Billing

Click Budgets

Create a Cost budget

Amount: $5

Period: Monthly

Alerts: 80% and 100%

Email: your email

![AWS Budget Guardrail]

<img width="1718" height="1244" alt="image" src="https://github.com/user-attachments/assets/476d7f0a-4339-458d-9efc-78e1da037ff1" />

### Security Relevance
Cloud security includes financial abuse prevention.
Attackers often abuse misconfigured accounts to run unauthorized workloads.
### Initial Threat Model

Before infrastructure existed, the following risks were considered:

- Unrestricted cloud spending
- Unmonitored resource creation
- Abuse of compute resources

These risks were mitigated by implementing cost guardrails before deployment.

## Step 2 — Creating the Network Boundary (VPC + Public Subnet)

Before deploying compute resources, a dedicated network boundary was created.
This defines how resources communicate internally and whether they are reachable
from the public internet.
Action — Create the VPC (AWS Console)
In AWS Console:

Search for VPC

Click Your VPCs

Click Create VPC

Choose:

VPC only

Name tag: cloud-lab-00-vpc

IPv4 CIDR block: 10.0.0.0/16

Leave everything else default

Click Create VPC

<img width="1465" height="698" alt="image" src="https://github.com/user-attachments/assets/c41af6fd-1332-4f8e-bb90-efeaed1489ed" />
### Security Relevance
A VPC provides network isolation.
Misconfigured routing or overly broad CIDR ranges can unintentionally expose resources.

Action — Create a Public Subnet
In VPC console:

Click Subnets

Click Create subnet

Configure:

VPC ID: cloud-lab-00-vpc

Subnet name: cloud-lab-00-public-subnet

Availability Zone: pick one (any is fine)

IPv4 CIDR block: 10.0.1.0/24

Click Create subnet

<img width="1522" height="1152" alt="image" src="https://github.com/user-attachments/assets/0d2df8fd-201b-4b19-8114-4d41d560283e" />
### Security Relevance
Subnets segment network traffic.
Public subnets introduce exposure when routing is configured to allow internet access.

Action — Create and Attach an Internet Gateway
Still in VPC console:

Click Internet Gateways

Click Create internet gateway

Name: cloud-lab-00-igw

Create

Click Actions → Attach to VPC

Select cloud-lab-00-vpc


<img width="1463" height="371" alt="image" src="https://github.com/user-attachments/assets/5c59ccbe-3e25-40de-bf8d-bd36073d1e1c" />
### Security Relevance
Internet gateways enable inbound and outbound internet connectivity.
Any subnet routed through an internet gateway becomes part of the external attack surface.


Update Route Table (This Makes It PUBLIC)
In VPC console:

Click Route Tables

Find the route table associated with cloud-lab-00-vpc

Edit routes

Add route:

Destination: 0.0.0.0/0

Target: cloud-lab-00-igw

Save

Then:

Associate this route table with cloud-lab-00-public-subnet


<img width="1673" height="365" alt="image" src="https://github.com/user-attachments/assets/d42e52e5-b836-4a17-aa21-736b18f44c9b" />
<img width="1471" height="226" alt="image" src="https://github.com/user-attachments/assets/2a79474b-a5fc-407d-b1bf-b9014b21f2d6" />
### Security Relevance
Routing determines reachability.
This route enables internet access and marks the subnet as public.


### Threat Perspective

At this stage, the environment has no compute resources,
but the network boundary allows internet connectivity.

Any future resources placed in this subnet may become externally reachable.
This is where cloud attack surface begins.

