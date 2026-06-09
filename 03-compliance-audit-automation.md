# Submission 3: Compliance Audit Automation

## Prompt Title
SOC2/HIPAA Compliance Audit Automation for AWS

## Category
Code Development

## Tags
Advanced, Technical, Professional

## Recommended AI Model(s)
Claude (Anthropic), GPT-4, Amazon Bedrock (Claude 3.5 Sonnet or later)

---

## The Prompt

```
You are a senior AWS compliance architect with expertise in SOC2, HIPAA, and GDPR frameworks. Help me build a production-ready compliance audit automation system using AWS CDK v2 (Python).

## Context

Our startup is pursuing SOC2 Type II certification. The auditors require evidence of security controls across our AWS environment. We currently have a basic AWS setup with EC2, RDS, S3, and Lambda. Manual evidence collection is taking 40+ hours per audit cycle. We need automated compliance monitoring and evidence generation.

## Task

Generate a complete CDK Python project that automates compliance monitoring for SOC2 and HIPAA:

### Part 1: Compliance Framework Mapping

Create a compliance rules engine that maps AWS Config rules to compliance controls:

**SOC2 Trust Service Criteria Mapping:**
```
CC6.1 (Logical Access) →
  - IAM password policy configured
  - MFA enabled for console users
  - Root account has no access keys
  - IAM users have policies attached

CC6.2 (User Authentication) →
  - MFA enabled for IAM users
  - Password policy meets complexity requirements
  - Console access requires MFA

CC6.3 (User Authorization) →
  - IAM policies follow least privilege
  - No wildcard (*) permissions in production
  - Roles have session duration limits

CC6.6 (System Boundaries) →
  - Security groups restrict inbound traffic
  - VPC flow logs enabled
  - No public RDS instances
  - S3 buckets block public access

CC7.1 (Vulnerability Management) →
  - EC2 instances patched (SSM Patch Manager)
  - GuardDuty enabled
  - Security Hub findings addressed

CC7.2 (Monitoring) →
  - CloudTrail enabled in all regions
  - CloudWatch alarms for critical events
  - Config rules monitoring compliance

CC8.1 (Change Management) →
  - CloudFormation/CDK for infrastructure changes
  - No manual resource modifications in production
  - Change approval process documented
```

**HIPAA Technical Safeguards Mapping:**
```
§164.312(a)(1) (Access Control) →
  - Unique user identification
  - Emergency access procedure
  - Automatic logoff
  - Encryption of ePHI at rest

§164.312(b) (Audit Controls) →
  - CloudTrail logging all API calls
  - S3 access logging enabled
  - RDS audit logging enabled

§164.312(c)(1) (Integrity) →
  - S3 versioning enabled
  - S3 object lock for critical data
  - EBS encryption enabled

§164.312(e)(1) (Transmission Security) →
  - TLS 1.2+ enforced
  - VPC endpoints for AWS services
  - S3 bucket policies enforce HTTPS
```

### Part 2: Automated Compliance Checks

Create Lambda functions for continuous compliance monitoring:

1. **IAM Compliance Checker**
   - Scan all IAM users for MFA status
   - Check password policy compliance
   - Identify unused credentials (>90 days)
   - Detect overly permissive policies
   - Output: Compliance report per control

2. **Network Compliance Checker**
   - Audit security groups for 0.0.0.0/0 ingress
   - Verify VPC flow logs enabled
   - Check for public subnets with sensitive resources
   - Validate NACL configurations
   - Output: Network compliance score

3. **Data Protection Checker**
   - Verify S3 bucket encryption (SSE-S3 or SSE-KMS)
   - Check S3 public access block settings
   - Validate RDS encryption at rest
   - Check EBS volume encryption
   - Verify DynamoDB encryption
   - Output: Data protection compliance score

4. **Logging & Monitoring Checker**
   - Verify CloudTrail enabled in all regions
   - Check CloudWatch alarms for critical events
   - Validate Config rules active
   - Check GuardDuty enabled
   - Output: Monitoring coverage score

5. **Change Management Checker**
   - Detect manual changes (not via CloudFormation)
   - Verify drift detection status
   - Check for resources without proper tags
   - Output: Change management compliance score

### Part 3: Evidence Collection

Automate evidence collection for auditors:

1. **Daily Evidence Snapshots**
   - IAM user list with MFA status
   - Security group configurations
   - S3 bucket policies and encryption settings
   - CloudTrail log integrity validation
   - Config rule compliance status

2. **Evidence Storage**
   - Store in dedicated S3 bucket: `compliance-evidence-{account-id}`
   - Organize by: `/{framework}/{control}/{date}/evidence.json`
   - Enable S3 Object Lock (WORM) for tamper-proof evidence
   - Lifecycle: 7 years retention (SOC2 requirement)

3. **Evidence Report Generator**
   - Generate PDF-ready compliance reports
   - Include: control description, AWS implementation, evidence, compliance status
   - Format for direct submission to auditors

### Part 4: Continuous Compliance Dashboard

Create CloudWatch Dashboard with:

1. **Overall Compliance Score** - Gauge chart (0-100%)
2. **Compliance by Framework** - SOC2 vs HIPAA scores
3. **Non-Compliant Resources** - Table with details
4. **Compliance Trend** - Line chart, last 90 days
5. **Critical Findings** - Alert count by severity
6. **Evidence Collection Status** - Last 30 days of evidence

### Part 5: Alerting & Remediation

1. **Real-time Alerts**
   - New non-compliant resource created → SNS
   - Critical security control violation → SNS + Email
   - Evidence collection failure → SNS
   - Compliance score drop > 5% → SNS

2. **Auto-Remediation** (with approval gate)
   - Public S3 bucket detected → Auto-block public access
   - Unencrypted EBS volume → Notify owner + create ticket
   - Security group with 0.0.0.0/0 → Notify security team
   - IAM user without MFA → Disable console access after 7 days

### Part 6: Audit-Ready Reports

Generate reports that auditors can directly use:

1. **Control Matrix Report**
   - Control ID → AWS implementation → Evidence → Status
   - Export as CSV/Excel for auditor review

2. **Gap Analysis Report**
   - Identify controls not yet implemented
   - Provide remediation steps
   - Estimate effort to achieve compliance

3. **Continuous Monitoring Report**
   - 30-day compliance history
   - Trend analysis
   - Exceptions and justifications

## Output Format

Generate the complete CDK project with:
- `app.py` - CDK entry point
- `stacks/compliance_stack.py` - Core compliance infrastructure
- `stacks/evidence_stack.py` - Evidence collection and storage
- `stacks/dashboard_stack.py` - Compliance dashboard
- `stacks/alerting_stack.py` - SNS and EventBridge rules
- `lambda/checkers/` - Compliance checker Lambda functions
- `lambda/evidence/` - Evidence collector Lambda
- `lambda/remediation/` - Auto-remediation Lambda
- `config/compliance-rules.json` - Framework-to-AWS mapping
- `templates/` - Report templates
- `README.md` - Setup guide and auditor instructions

## Constraints

- Evidence must be tamper-proof (S3 Object Lock)
- All compliance data retained for 7 years
- Compliance checks run daily (configurable)
- Include HIPAA-specific encryption requirements
- Support multi-account environments via Organizations
- Follow AWS Well-Architected Security pillar
- All Lambda functions must have proper error handling and logging

## Verification

After deployment:
1. Check compliance dashboard: CloudWatch → Dashboards
2. Verify evidence collection: `aws s3 ls s3://compliance-evidence-{account-id}/`
3. Run manual compliance check: Invoke Lambda with test event
4. Generate sample report: Check S3 for generated reports
5. Test alert: Create non-compliant resource and verify SNS
```

---

## Description & Use Case

### What Problem Does This Solve?

SOC2 and HIPAA compliance audits are painful for startups:
- **Manual evidence collection**: 40+ hours per audit cycle
- **Inconsistent documentation**: Different team members document differently
- **Audit anxiety**: Not knowing if you're compliant until the auditor arrives
- **Remediation scramble**: Fixing issues reactively during the audit

This prompt automates the entire compliance monitoring and evidence collection process, turning a 40-hour manual effort into an automated daily routine.

### Who Should Use This?

- **Startup CTOs**: Prepare for SOC2 certification without dedicated compliance staff
- **Security Engineers**: Automate compliance monitoring across AWS accounts
- **Compliance Officers**: Generate audit-ready reports automatically
- **DevOps Teams**: Integrate compliance checks into CI/CD pipelines
- **Healthcare Startups**: Meet HIPAA technical safeguard requirements

### Expected Outcome

After deployment, you will have:

1. Continuous compliance monitoring across all AWS resources
2. Daily evidence snapshots stored in tamper-proof S3 storage
3. Real-time alerts for compliance violations
4. Audit-ready reports that can be directly submitted to auditors
5. Auto-remediation for critical compliance issues
6. Estimated time savings: 35+ hours per audit cycle

### Prerequisites

| Requirement | Description |
|-------------|-------------|
| AWS Account | With existing workloads to monitor |
| IAM Permissions | AdministratorAccess or compliance-specific permissions |
| AWS CLI | Installed and configured |
| CDK CLI | `npm install -g aws-cdk` |
| Python | 3.9+ |
| SNS Email | Email address for compliance alerts |
| Compliance Framework | SOC2, HIPAA, or both |

### Well-Architected Framework Alignment

| Pillar | Coverage |
|--------|----------|
| Security | IAM compliance, encryption verification, network security, threat detection |
| Operational Excellence | Automated monitoring, evidence collection, audit preparation |
| Reliability | Tamper-proof evidence storage, 7-year retention |

---

## Example Output

### Sample Compliance Dashboard

```
┌─────────────────────────────────────────────────────────┐
│           AWS Compliance Dashboard                       │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Overall Compliance Score: 87% ████████████░░░           │
│                                                          │
│  SOC2: 92% ████████████████░░                            │
│  HIPAA: 82% █████████████░░░░                            │
│                                                          │
│  Non-Compliant Resources: 12                             │
│  ├── Critical: 2                                         │
│  ├── High: 4                                             │
│  └── Medium: 6                                           │
│                                                          │
│  Evidence Collected: 30/30 days                          │
│  Last Audit Score: 85% (2026-05-08)                      │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### Sample Evidence Report (JSON)

```json
{
  "control_id": "CC6.1",
  "control_name": "Logical Access Controls",
  "framework": "SOC2",
  "aws_implementation": "IAM Password Policy + MFA",
  "compliance_status": "COMPLIANT",
  "evidence": {
    "password_policy": {
      "minimum_length": 14,
      "require_symbols": true,
      "require_numbers": true,
      "require_uppercase": true,
      "require_lowercase": true,
      "max_age_days": 90
    },
    "mfa_status": {
      "total_users": 15,
      "mfa_enabled": 15,
      "mfa_disabled": 0,
      "compliance_rate": "100%"
    },
    "root_account": {
      "access_keys": 0,
      "mfa_enabled": true,
      "last_used": "2026-01-15T10:30:00Z"
    }
  },
  "collected_at": "2026-06-08T08:00:00Z",
  "collected_by": "compliance-evidence-collector"
}
```

### Sample Gap Analysis Report

```
SOC2 Gap Analysis Report
Generated: 2026-06-08

COMPLIANT CONTROLS (23/28):
✅ CC6.1 - Logical Access Controls
✅ CC6.2 - User Authentication
✅ CC6.3 - User Authorization
...

NON-COMPLIANT CONTROLS (5/28):
❌ CC6.6 - System Boundaries
   Issue: 3 security groups allow 0.0.0.0/0 on port 22
   Resources: sg-0abc123, sg-0def456, sg-0ghi789
   Remediation: Restrict SSH access to VPN IP ranges
   Effort: 2 hours

⚠️ CC7.1 - Vulnerability Management
   Issue: 5 EC2 instances missing critical patches
   Resources: i-0abc123, i-0def456, ...
   Remediation: Run SSM Patch Manager baseline
   Effort: 4 hours
```

---

## Additional Notes

### Cost Estimate

- Lambda (5 checkers daily): ~$3/month
- S3 (evidence storage): ~$5-20/month depending on volume
- DynamoDB (compliance state): ~$2/month
- CloudWatch Dashboard: Free
- SNS notifications: ~$1/month
- Total: ~$12-27/month

### Integration with Existing Tools

This system can integrate with:
- **Jira/Linear**: Auto-create tickets for non-compliant resources
- **Slack/Teams**: Real-time compliance alerts
- **Vanta/Drata**: Export evidence in their formats
- **AWS Audit Manager**: Complement with additional controls

### Customization

Edit `config/compliance-rules.json` to:
- Add custom compliance controls
- Modify thresholds for compliance checks
- Add new AWS services to monitor
- Customize evidence retention periods

### Important Disclaimers

- This tool assists with compliance but does not guarantee certification
- Always work with qualified compliance professionals for audit preparation
- Evidence should be reviewed by compliance team before auditor submission
- HIPAA compliance requires additional organizational controls beyond AWS
