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


## Step 3 — Launching Public Compute (EC2)

With the network boundary in place, a public EC2 instance was launched
inside the public subnet. This introduces a real, externally reachable
compute resource and marks the point where exploitation becomes possible.

### SSH Key Pair Creation

Create the Key Pair (Do This Exactly)

On that same screen:

Click Create new key pair

Set:

Key pair name:

cloud-lab-00-key


Key pair type: ED25519 (recommended)
(RSA is fine if ED25519 isn’t available)

Private key file format: .pem

Click Create key pair

Your browser will download the .pem file immediately

A dedicated SSH key pair was created for secure administrative access
to the EC2 instance.

<img width="637" height="607" alt="image" src="https://github.com/user-attachments/assets/4df33a5c-10ca-4dad-8610-5023ce172ab2" />

**Security Relevance**
SSH keys represent root-level access to compute resources.
Key leakage results in full compromise of the instance.

Action — Launch the EC2 Instance
In AWS Console:

1.Search EC2

2.Click Instances

3.Click Launch instance

Instance Configuration (Do Not Deviate)
Name:
cloud-lab-00-ec2

AMI (Operating System)

Choose Amazon Linux 2023
(Amazon Linux 2 is also acceptable if that’s what you see)

Why:

-Common

-Secure defaults

-Real-world usage

Instance Type

-t2.micro or t3.micro (Free Tier)

Key Pair

-Select: cloud-lab-00-key

⚠️ This key = root-level access.
Never upload it anywhere.


<img width="879" height="909" alt="image" src="https://github.com/user-attachments/assets/eee87245-922e-4a54-aec3-cd2c5ec08994" />


Network Settings (CRITICAL)

Click Edit under Network settings.

Set exactly:

-VPC: cloud-lab-00-vpc

-Subnet: cloud-lab-00-public-subnet

-Auto-assign public IP: ✅ Enable

-Firewall (security group): Create new

  -Name: cloud-lab-00-sg

  -Description: Public SSH access (restricted)

Inbound Rules:

-Type: SSH

-Source: My IP
NOT 0.0.0.0/0 (yet)


<img width="914" height="770" alt="image" src="https://github.com/user-attachments/assets/acec79c0-87f2-4870-9d74-e5281a4e1050" />

Launch the instance

Click Launch instance.


<img width="1533" height="135" alt="image" src="https://github.com/user-attachments/assets/3e761b9f-453a-4bd1-b143-e4ad13d69d88" />

### Security Relevance
The EC2 instance is publicly reachable due to:
- Placement in a public subnet
- Route table with internet gateway access
- Assigned public IPv4 address

Any service exposed on this instance becomes a potential attack vector.

### Threat Perspective

At this stage, an attacker could:
- Discover the public IP via scanning
- Attempt SSH access
- Enumerate exposed services

Security group rules determine the immediate risk level.

## Step 4 — Proving Impact (SSH Access)

With the EC2 instance running in a public subnet and assigned a public IPv4
address, SSH access was tested to validate real-world exposure and impact.

Action — Prepare Your SSH Key (Local Machine) your pc
On Windows (PowerShell)

-Make sure your key is in a safe path, for example:

C:\keys\cloud-lab-00-key.pem


-Fix permissions (important — SSH will fail without this):

icacls C:\keys\cloud-lab-00-key.pem /inheritance:r
icacls C:\keys\cloud-lab-00-key.pem /grant:r "$($env:USERNAME):(R)"


This prevents other users from reading the key.

Action — SSH Into the Instance

From PowerShell:

ssh -i C:\keys\cloud-lab-00-key.pem ec2-user@<PUBLIC_IPV4>


Replace <PUBLIC_IPV4> with the IP shown in EC2.

When prompted:

Are you sure you want to continue connecting (yes/no)?


Type:

yes

✅ Success Criteria

You should see:

A terminal prompt

You are logged in as ec2-user

<img width="842" height="211" alt="image" src="https://github.com/user-attachments/assets/e40e9c47-ce80-4428-82cf-ccbcc2c71a9b" />

Run these commands in order:

whoami
hostname
ip a

<img width="871" height="413" alt="image" src="https://github.com/user-attachments/assets/9abf529a-4d97-4003-803f-c55e315f7a6d" />


Do not run anything else yet.

### Security Relevance
Successful SSH access confirms that the EC2 instance is reachable
and accessible from the internet.

This represents authenticated remote command execution and validates
the attack surface introduced by public compute resources.

### Threat Perspective

An attacker who obtains valid SSH credentials or a leaked private key
would gain immediate command execution on the instance.

This demonstrates why key management and network restrictions
are critical in cloud environments.

## Step 5 — Intentional Misconfiguration (Public SSH Access)

To demonstrate the impact of overly permissive network controls,
the EC2 security group was intentionally modified to allow SSH access
from any IP address.

Action — Open SSH to the World (On Purpose)
Go to:

EC2 → Instances → cloud-lab-00-ec2 → Security tab

Click the attached Security Group (cloud-lab-00-sg).

Edit Inbound rules
Change SSH rule:

From: My IP

To:

0.0.0.0/0


This allows anyone on the internet to attempt SSH.

Click Save.

<img width="1378" height="324" alt="Screenshot 2026-01-15 014350" src="https://github.com/user-attachments/assets/c669be0b-942b-49af-b6b5-89f316de4f83" />

### Threat Perspective

Allowing SSH access from 0.0.0.0/0 exposes the instance to:
- Global brute-force attempts
- Credential stuffing
- Automated scanning and exploitation

This configuration dramatically increases risk and is a common
cloud security misconfiguration.

Optional Validation (You Don’t Have to Run It)

From a second network (or imagining attacker behavior):

Port 22 would now respond globally

Any scanner (Shodan, masscan, nmap) could detect it

We don’t need to actually scan — the configuration alone is proof.

## Step 6 — IAM Role Abuse via Instance Metadata (IMDS)

To demonstrate cloud identity risk, an over-permissioned IAM role
was attached to the EC2 instance. Credentials were retrieved from
the instance metadata service to simulate post-exploitation access
to AWS APIs.

Action — Create an Over-Permissioned IAM Role (On Purpose)
Go to:

IAM → Roles → Create role

Trusted entity

AWS service

EC2

Permissions (INTENTIONALLY BAD)

Attach:

AdministratorAccess

Yes — this is deliberately dangerous.

Role name:
cloud-lab-00-overprivileged-role


Create the role.

<img width="1315" height="693" alt="image" src="https://github.com/user-attachments/assets/2e2bb5a2-1072-4f3e-a405-13abbe7ef335" />

Action — Attach Role to EC2 Instance
Go to:

EC2 → Instances → cloud-lab-00-ec2

Actions → Security → Modify IAM role

Select:

cloud-lab-00-overprivileged-role

Save.

<img width="516" height="172" alt="image" src="https://github.com/user-attachments/assets/e9bc97ff-9542-451d-aa37-0f9212221010" />

--Action--

(IMDSv2 — do this exactly)
Step 6A — Request an IMDSv2 token

Run this on the EC2 instance:

TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

This gets a temporary metadata access token.

Step 6B — Query the role name using the token

curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/iam/security-credentials/


Now you should see:

cloud-lab-00-overprivileged-role


If you see that, everything is working correctly.

Step 6C — Retrieve the credentials (still using the token)
curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/iam/security-credentials/cloud-lab-00-overprivileged-role


You will receive JSON containing:

AccessKeyId

SecretAccessKey

Token

Expiration

<img width="1351" height="550" alt="image" src="https://github.com/user-attachments/assets/708f1010-e471-481d-bf7d-b90f40c3f91d" />

These are live AWS credentials.

### IMDSv2 Enforcement

The EC2 instance enforced IMDSv2 by default, requiring a session token
to access instance metadata.

Unauthenticated IMDSv1-style requests returned no data, preventing
credential exposure until the correct IMDSv2 workflow was used.

This demonstrates AWS’s defense-in-depth controls against metadata abuse.

## Step 7 — AWS Enumeration Using Stolen Credentials

Temporary AWS credentials retrieved from IMDS were used to authenticate
to the AWS API and enumerate account-level resources, validating the
impact of IAM role compromise.

Action — Configure AWS CLI on the EC2 Instance

You are still SSH’d into the EC2 instance.

Check if AWS CLI exists
aws --version


Amazon Linux 2023 usually has it preinstalled. If not:

sudo dnf install -y awscli

Action — Export the Stolen Credentials (TEMPORARY)

Do NOT paste credentials into chat or GitHub

From the IMDS output you saw earlier, export them like this:

export AWS_ACCESS_KEY_ID=<AccessKeyId>
export AWS_SECRET_ACCESS_KEY=<SecretAccessKey>
export AWS_SESSION_TOKEN=<Token>


These only exist in memory for this shell.

Validation — Who Am I?

Run:

aws sts get-caller-identity

Expected output:

Your AWS Account ID

ARN showing:

cloud-lab-00-overprivileged-role


<img width="1125" height="160" alt="image" src="https://github.com/user-attachments/assets/7cb672f9-51ac-4296-9233-dc74f43b4a83" />


Enumeration — Prove the Blast Radius

Run these one by one:

aws ec2 describe-instances --query 'Reservations[].Instances[].InstanceId'

aws iam list-users

aws s3 ls

aws iam list-roles


If AdministratorAccess is attached, all of these will succeed.

<img width="972" height="179" alt="image" src="https://github.com/user-attachments/assets/3c1d7428-f6ba-40fc-b3a3-4cbfc0fd78e0" />

### Security Relevance
Stolen instance metadata credentials enabled full access to AWS APIs.
This demonstrates how over-permissioned IAM roles can lead to
account-wide compromise following initial host access.

---STEP 8 FINAL---

This step proves you understand how to fix what you broke — that’s what hiring managers actually care about.!

Remediation Actions (Do These in Order)
8.1 Remove Over-Privileged IAM Role ❌
AWS Console

EC2 → Instances → your instance

Actions → Security → Modify IAM role

Set to No IAM role

Save

Why this matters

This immediately kills credential theft via IMDS.

<img width="1405" height="295" alt="image" src="https://github.com/user-attachments/assets/c1582a3a-1576-435b-a89b-4c6a4d01994a" />

8.2 Enforce IMDSv2 (Mandatory) 🔒
EC2 → Instance → Actions

Modify instance metadata options

Set:

IMDSv2: Required

Hop limit: 1

Save

Why this matters

Blocks SSRF-style metadata theft

Required in real enterprise environments

<img width="526" height="489" alt="image" src="https://github.com/user-attachments/assets/c7e9c498-ec48-4d35-9459-1433ff6de1bc" />

8.3 Lock Down SSH Exposure 🚫
Security Group → Inbound Rules

Change:

SSH (22) — 0.0.0.0/0


To:

SSH (22) — YOUR_PUBLIC_IP/32


Or remove SSH entirely.

as i did 
<img width="1513" height="254" alt="image" src="https://github.com/user-attachments/assets/f2ccdfc4-10b7-472e-b324-a2978093a225" />

8.4 (Optional but Strong) Stop or Terminate EC2

You’ve proven the risk.
In real life, the instance would be rebuilt.


<img width="737" height="458" alt="image" src="https://github.com/user-attachments/assets/93be7679-8951-4cd1-b74e-b6cad2c18cdb" />

## Final Assessment

This lab demonstrated how a publicly exposed EC2 instance with an
over-permissioned IAM role can lead to full AWS account visibility
via instance metadata credential theft.

Remediation steps eliminated the attack surface by enforcing IMDSv2,
removing IAM role exposure, and restricting network access.

This mirrors real-world cloud breaches caused by IAM misconfiguration
rather than software vulnerabilities.


