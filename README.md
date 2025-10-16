# How to Organize Your Accounts in AWS: A Complete Guide

Organizing AWS accounts might seem straightforward at first, but as your infrastructure grows, managing multiple accounts becomes increasingly complex; the good news is AWS provides clear best practices and native tooling to make this scalable and safe.

## Understanding AWS Accounts

An AWS account is a container for your cloud resources, the users who access them, and the account’s settings and configurations, which makes it a fundamental isolation and governance boundary in AWS. Unlike an email or social account, its scope includes infrastructure, identity, billing, and controls, so how you structure accounts impacts security, cost, and operations.

## The Root User vs IAM Users

Each account has exactly one root user and it should only be used for critical tasks like billing or account settings; enable MFA for the root user as a baseline control to reduce risk. For day‑to‑day work, provision IAM users/roles with least privilege and prefer federation to avoid managing local users across accounts.

## Why Use Multiple AWS Accounts?

Multiple accounts help organize by environment or business unit, simplify least‑privilege at account scope, and avoid single‑account service limits and quota ceilings, improving blast‑radius reduction and clarity of ownership. The trade‑offs include decentralized billing/logs, cross‑account sharing, multi‑account access management, and configuration/auditing at scale—which AWS addresses with Organizations, RAM, IAM Identity Center and Control Tower.

## AWS Organizations: The Foundation

Organizations centralizes multi‑account management with a Management Account, Member Accounts, and Organizational Units (OUs) to group accounts by function or risk profile, enabling policy‑based governance at scale. Immediate benefits include consolidated billing, automated account provisioning, and isolated environments, plus potential centralization of security, permissions, logging, and configuration enforcement.

Tip: Don’t deploy workloads in the Management Account; reserve it for org governance and shared services control to reduce risk and complexity.

## Service Control Policies (SCPs)

SCPs act as guardrails at org/OU/account scope using IAM‑like syntax, never grant permissions, and are inherited down the hierarchy to define maximum allowable actions. Effective permissions = allowed by IAM AND not denied by an SCP; use deny‑lists for targeted blocks or allow‑lists for strict baselines when needed.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": ["organizations:LeaveOrganization"],
      "Resource": "*"
    }
  ]
}
```

This example prevents accounts from leaving the organization, which protects governance integrity across members.

## Managing Access with IAM Identity Center (AWS SSO)

Managing IAM users in each account does not scale; IAM Identity Center federates access from your IdP (e.g., Azure AD/Okta) to permission sets and roles across many accounts.

Flow:
```
User → Identity Provider → IAM Identity Center → Assume IAM Role → AWS Account
```

When a user signs in:
1. Authenticates with your IdP.
2. Sees authorized accounts and roles in IAM Identity Center.
3. Selects an account and assumes an IAM role.
4. Receives temporary credentials to access that account.

## Sharing Resources with AWS Resource Access Manager (RAM)

RAM lets you share resources like VPC subnets, Transit Gateways, Route 53 Resolver rules, CodeBuild projects and SageMaker across accounts, OUs, or the whole organization, using predefined permissions per resource type. This reduces duplication, improves security visibility, and enables centralized models (e.g., shared networking or centralized CI/CD).

## AWS Control Tower: Best Practices on Autopilot

Control Tower automates a landing zone with AWS‑managed guardrails, Account Factory and Core accounts (Log Archive and Audit) for governance and visibility. Guardrails come in three types—mandatory, strongly recommended, and elective—implemented with SCPs (preventive) and AWS Config (detective), and applied at OU level with inheritance.


## The Log Archive Account

The Log Archive account centralizes CloudTrail, AWS Config, CloudWatch and S3 access logs from enrolled accounts, improving security, forensics and automated responses. Isolating logs strengthens evidence protection and deters negligent or abusive actions at org level.

## Tag Policies and Governance

Tag policies standardize keys/values and resource types to enforce consistency for cost allocation, automation, and access control. Manage policies in Organizations and validate compliance in member accounts; for strict prevention of untagged resources, complement with SCPs as a preventive control.

## 9 Best Practices for Organizing AWS Accounts

- Organize by security and operational needs—environment, business unit, or compliance domain.  
- Apply guardrails at the OU level—manage SCPs and tag policies by OU for consistency.  
- Avoid deep OU hierarchies—keep to 2–3 levels for simplicity and clear inheritance.  
- Start small and expand—iterate as workloads and teams mature.  
- Don’t deploy workloads in the management account—reserve it for org governance.  
- Separate prod from non‑prod—critical for security, compliance, and blast‑radius control.  
- Small, related workloads per prod account—improves quotas, isolation, and operations.  
- Use federated access—adopt IAM Identity Center for scalable onboarding/offboarding.  
- Automate for agility and scale—use Control Tower and IaC (CloudFormation/CDK/Terraform).

## Example Organization Structure

```
Root
├── Management Account (no workloads)
├── Core OU
│   ├── Log Archive Account
│   └── Audit Account
├── Shared Services OU
│   └── Networking Account (Transit Gateway, VPCs)
├── Development OU
│   ├── Dev Account - Application A
│   └── Dev Account - Application B
├── Test OU
│   ├── Test Account - Application A
│   └── Test Account - Application B
└── Production OU
    ├── Prod Account - Application A
    └── Prod Account - Application B
```

## Getting Started Checklist

- Create an AWS Organization in your management account and define a minimal OU baseline.  
- Set up AWS Control Tower to enable mandatory guardrails and a secure baseline from day one.  
- Define your OU structure by security/ops needs; avoid deep hierarchies and plan iterative growth.  
- Configure IAM Identity Center for federated access with permission sets by role/environment.  
- Apply SCPs at OU level to enforce preventive boundaries and consistent governance.  
- Enable an organization‑level CloudTrail and AWS Config aggregators for cross‑account visibility.  
- Stand up the Log Archive account to centralize CloudTrail, Config, CloudWatch, and S3 access logs.  
- Implement tag policies and a tagging standard for cost allocation, automation, and access control.  
- Use Resource Access Manager to share common resources with OUs/accounts/organization as needed.  
- Regularly review structure, guardrails, and costs; adjust policies and OU design as you scale.

## Conclusion

Organizing AWS accounts effectively is crucial for security, compliance, and operational excellence. Start small, apply OU‑level guardrails, centralize logging and identity, and automate with Control Tower and IaC to scale with confidence and control.

