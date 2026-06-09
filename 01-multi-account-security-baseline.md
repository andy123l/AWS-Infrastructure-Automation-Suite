# Submission 1: AWS Multi-Account Security Baseline

## Prompt Title
AWS Multi-Account Security Baseline - One-Prompt Enterprise Security Setup

## Category
Code Development

## Tags
Advanced, Technical, Professional

## Recommended AI Model(s)
Claude (Anthropic), GPT-4, Amazon Bedrock (Claude 3.5 Sonnet or later)

---

## The Prompt

```
You are a senior AWS security architect with 15+ years of experience in enterprise cloud security. Help me build a production-ready AWS multi-account security baseline using AWS CDK v2 (Python).

## Context

My organization is migrating from a single AWS account to a multi-account architecture. We need enterprise-grade security controls that pass SOC2 and HIPAA audits. The team has intermediate AWS experience but limited knowledge of Organizations, SCP, and security services.

## Task

Generate a complete CDK Python project that sets up the following security baseline:

### Module 1: Organizations + OU Structure

Create an AWS Organizations structure with these OUs:
```
Root (Management Account)
├── Security OU
│   ├── LogArchive Account      (centralized audit logs)
│   └── SecurityTooling Account  (GuardDuty, Security Hub management)
├── Infrastructure OU
│   └── SharedServices Account   (shared DNS, ECR, CI/CD)
├── Sandbox OU
│   └── [dev-experiment accounts]
├── Staging OU
│   └── Staging Account
├── Production OU
│   └── Prod Account
└── Suspended OU
    └── [decommissioned accounts]
```

Requirements:
- Enable Organizations with ALL_FEATURES
- Enable SCP and TAG_POLICY policy types
- Register SecurityTooling as delegated admin for GuardDuty, Security Hub, Config, CloudTrail
- Create mandatory tag policy: Environment, Owner, CostCenter, Project

### Module 2: Service Control Policies (SCPs)

Create 8 SCP policies:
1. **DenyRootUser** - Block root user operations (except billing) on all member accounts
2. **DenyLeaveOrganization** - Prevent accounts from leaving the org
3. **DenyDisablingSecurityServices** - Block disabling CloudTrail, GuardDuty, Config, Security Hub
4. **RestrictRegions** - Allow only us-east-1, us-west-2, eu-west-1 (configurable)
5. **DenyPublicAccess** - Block public S3 buckets, public RDS, public Lambda URLs
6. **DenyLargeInstances** - Sandbox OU: only allow t3.*, t2.*, m5.large, m5.xlarge
7. **DenyEncryptionDisable** - Production/Staging: force S3 KMS encryption, EBS encryption, RDS encryption
8. **SuspendedFullDeny** - Deny all actions on suspended accounts

### Module 3: GuardDuty + Security Hub

Configure in SecurityTooling account:
- GuardDuty with S3, EKS, EBS, RDS, Lambda protection enabled
- Auto-enable for new organization members
- Export findings to S3 with KMS encryption and 90-day Glacier lifecycle
- Security Hub with AWS Foundational and CIS benchmarks
- SNS alerts for MEDIUM+ severity findings

### Module 4: CloudTrail Centralized Logging

- Organization-wide trail in Management account (all regions, all accounts)
- Log to LogArchive account S3 bucket with SSE-KMS encryption
- CloudWatch Logs integration in SecurityTooling account
- Metric filters for: Root login, IAM policy changes, Security group changes, Console login failures
- CloudWatch Alarms → SNS notifications
- Log lifecycle: 90 days Standard → Glacier → Delete after 365 days

### Module 5: AWS Config Compliance

- Enable Config in all member accounts with centralized delivery to SecurityTooling
- 28 Config rules covering: Security (MFA, password policy, Cloudtrail), Network (VPC flow logs, public IPs), Storage (S3 public access, encryption), Compute (SSM managed, stopped instances), Database (RDS encryption, DynamoDB PITR)
- Auto-remediation for: public S3 buckets (auto-block), VPC flow logs (auto-create)
- Compliance dashboard in CloudWatch

### Module 6: Cross-Account IAM Roles

Create 5 standardized roles:
1. **BreakGlassRole** - Emergency access, forced MFA, 1-hour session, alert on use
2. **SecurityAuditRole** - Read-only for security team, 4-hour session
3. **DeploymentRole** - CI/CD deployment from SharedServices, 1-hour session, minimal permissions
4. **ReadOnlyRole** - Developer read access to Sandbox/Staging, 8-hour session
5. **BillingRole** - Finance team billing view, 8-hour session

### Module 7: Cost Controls

- Organization-level budget ($10,000/month configurable) with 80%/100% alerts
- Per-OU budgets (Security $2K, Infra $3K, Sandbox $500, Staging $1.5K, Prod $5K)
- Cost anomaly detection ($100 threshold or 200% above average)
- Idle resource detection Lambda: EC2 (CPU<5% 7 days), unattached EBS, unused EIP, idle RDS, empty ALB
- Cost optimization recommendations: RI/Savings Plan analysis, S3 storage class optimization, right-sizing

## Output Format

Generate the complete CDK project with:
- `app.py` - CDK entry point
- `stacks/` - One stack per module
- `lambda/` - Idle resource detector and cost optimizer
- `policies/` - SCP JSON files
- `cdk.json` - CDK configuration
- `requirements.txt` - Python dependencies
- `README.md` - Setup instructions, prerequisites, verification steps

## Constraints

- Use CDK L1 constructs (CfnResource) for Organizations resources
- Include specific parameter values (not placeholders)
- Add cdk deploy checklist before each module
- Include troubleshooting section for common errors
- Align with AWS Well-Architected Framework (Security, Reliability, Cost Optimization, Operational Excellence)
- All S3 buckets must have: versioning, encryption, public access block, lifecycle policies
- All IAM roles must have: session duration limits, condition keys where applicable

## Verification Commands

After each module, provide CLI commands to verify:
1. OU structure: `aws organizations list-organizational-units-for-parent`
2. SCP attachment: `aws organizations list-policies-for-target`
3. GuardDuty status: `aws guardduty list-detectors`
4. CloudTrail status: `aws cloudtrail get-trail-status`
5. Config rules: `aws configservice describe-config-rules`
6. IAM roles: `aws iam list-roles --query 'Roles[?contains(RoleName, \`BreakGlass\`)]'`
```

---

## Description & Use Case

### What Problem Does This Solve?

Most AWS users start with a single account, but as teams and workloads grow, single-account architecture faces critical challenges:

- **Security Risk**: One breach affects all workloads
- **Compliance Difficulty**: Cannot isolate different compliance levels
- **Cost Opacity**: All expenses mixed together
- **Permission Chaos**: Hard to implement least privilege

AWS recommends using AWS Organizations for multi-account management, but the configuration process is extremely complex - involving OU structure design, SCP policies, security service enablement, IAM role systems, and dozens of steps where any misconfiguration can create security vulnerabilities.

### Who Should Use This?

- **Startup CTOs**: Build correct security architecture from day one
- **DevOps Engineers**: Set up standardized AWS environments for teams
- **Security Engineers**: Quickly deploy security baselines
- **Solution Architects**: Design multi-account architectures for clients
- **Career Changers**: Learn AWS security best practices

### Expected Outcome

After running `cdk deploy` for all modules, you will have:

1. A complete AWS Organizations with 6 OUs and 5 member accounts
2. 8 SCP policies preventing common security mistakes
3. GuardDuty and Security Hub enabled across all accounts with centralized alerting
4. Organization-wide CloudTrail with centralized logging and real-time alerts
5. 28 Config rules with auto-remediation for critical issues
6. 5 standardized cross-account IAM roles with security controls
7. Budget alerts, anomaly detection, and idle resource monitoring

### Prerequisites

| Requirement | Description |
|-------------|-------------|
| AWS Account | A Management Root account (new or existing) |
| IAM Permissions | AdministratorAccess on the Management account |
| AWS CLI | Installed and configured (`aws configure`) |
| CDK CLI | `npm install -g aws-cdk` |
| Python | 3.9+ with pip |
| Node.js | 18+ (CDK dependency) |

### Well-Architected Framework Alignment

| Pillar | Coverage |
|--------|----------|
| Security | SCP policies, GuardDuty, Security Hub, CloudTrail, encryption enforcement |
| Reliability | Multi-account isolation reduces blast radius |
| Cost Optimization | Budget alerts, idle resource detection, RI recommendations |
| Operational Excellence | Standardized OU structure, automated compliance, centralized logging |

---

## Example Output

The prompt generates a complete CDK project. Here's a sample of the generated SCP policy (Module 2):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyDisablingSecurityServices",
      "Effect": "Deny",
      "Action": [
        "cloudtrail:StopLogging",
        "cloudtrail:DeleteTrail",
        "guardduty:DeleteDetector",
        "config:DeleteConfigRule",
        "config:StopConfigurationRecorder",
        "securityhub:DisableSecurityHub"
      ],
      "Resource": "*"
    }
  ]
}
```

And a sample of the generated CDK stack (Module 1):

```python
from aws_cdk import (
    Stack,
    CfnOrganization,
    CfnOrganizationalUnit,
    CfnAccount,
    Tags,
)
from constructs import Construct

class OrganizationsStack(Stack):
    def __init__(self, scope: Construct, id: str, **kwargs):
        super().__init__(scope, id, **kwargs)

        # Create Organization
        self.organization = CfnOrganization(
            self, "Organization",
            feature_set="ALL_FEATURES"
        )

        # Create OUs
        self.security_ou = CfnOrganizationalUnit(
            self, "SecurityOU",
            name="Security",
            parent_id=self.organization.attr_root_id
        )
        # ... (continues with all OUs and accounts)
```

---

## Additional Notes

### Troubleshooting Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| `Organization already exists` | Account already belongs to an org | Leave existing org or use current one |
| `Account creation rate limit` | AWS limits account creation rate | Wait 24 hours or contact AWS support |
| SCP policy locks accounts | Policy too restrictive | Detach policy from Management account |
| GuardDuty not auto-enabling | Auto-enable takes time | Wait up to 24 hours |
| CloudTrail cross-account write fails | S3 bucket policy error | Check bucket policy for org ID |

### Cost Estimate

- GuardDuty: ~$1-4 per million events
- Security Hub: ~$0.0010 per finding per month
- Config: ~$0.003 per configuration item recorded
- CloudTrail: First trail free, data events ~$0.10 per 100,000 events
- Total estimated: $50-200/month for small to medium organizations

### Modular Usage

This prompt generates a modular project. You can deploy modules independently:
- **Minimum viable**: Module 1 + 2 (Organization + SCPs)
- **Standard security**: Modules 1-5 (add threat detection, audit, compliance)
- **Full enterprise**: All 7 modules
