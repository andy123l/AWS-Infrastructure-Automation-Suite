# Submission 2: AWS Cost Optimization Automation Suite

## Prompt Title
AWS Cost Optimization Automation - Stop Overspending on Cloud

## Category
Code Development

## Tags
Technical, Professional

## Recommended AI Model(s)
Claude (Anthropic), GPT-4, Amazon Bedrock (Claude 3.5 Sonnet or later)

---

## The Prompt

```
You are a senior AWS solutions architect and FinOps expert. Help me build a production-ready AWS cost optimization automation suite using AWS CDK v2 (Python).

## Context

Our AWS monthly bill has grown to $15,000/month and we suspect 20-30% is wasted on idle resources, oversized instances, and suboptimal storage classes. We need automated detection and optimization recommendations. The team has basic AWS experience but limited cost management knowledge.

## Task

Generate a complete CDK Python project with the following components:

### Part 1: Budget Alerts System

Create AWS Budgets with these configurations:

1. **Organization Total Budget**
   - Monthly budget: $12,000 (configurable via CDK context)
   - Alert thresholds:
     - 80% actual spend → Email notification
     - 100% actual spend → Email + SMS notification
     - 100% forecasted spend → Email notification
   - Use Cost Allocation Tags for filtering

2. **Service-Level Budgets**
   - EC2: $4,000/month
   - RDS: $2,500/month
   - S3: $800/month
   - Lambda: $300/month
   - Data Transfer: $1,000/month
   - Other: $3,400/month

3. **Environment Budgets** (using tags)
   - Production: $8,000/month
   - Staging: $2,500/month
   - Development: $1,500/month

### Part 2: Cost Anomaly Detection

1. **AWS Cost Anomaly Detector**
   - Create organization-wide anomaly detector
   - Monitor by Service and Account dimensions
   - Alert threshold: $200 or 150% above rolling average
   - Immediate SNS notification on detection

2. **Custom Anomaly Lambda**
   Schedule: Daily at 08:00 UTC (EventBridge rule)
   Checks:
   - Single resource daily cost > $100
   - Account daily cost > 150% of 7-day average
   - New high-cost services not seen in last 30 days
   - Spot instance interruption costs spike
   Output: SNS alert with anomaly details

### Part 3: Idle Resource Detector

Create a Lambda function (Python 3.12) that runs daily and detects:

**EC2 Instances:**
- CPU utilization < 3% for 7 consecutive days (CloudWatch metric)
- Instance stopped for > 30 days
- Recommendation: Stop, rightsize, or terminate

**EBS Volumes:**
- Volume state = "available" (not attached)
- Volume IOPS < 100 for 7 days (gp3/io2)
- Recommendation: Snapshot and delete, or downsize

**Elastic IPs:**
- Association state = "false" (not attached to running instance)
- Recommendation: Release EIP ($0.005/hour waste each)

**RDS Instances:**
- Database connections < 1 for 7 days
- Instance stopped for > 14 days
- Recommendation: Snapshot and delete

**S3 Buckets:**
- No GetObject/PutObject in 90 days (CloudTrail analysis)
- Missing lifecycle policy
- Recommendation: Add lifecycle to transition to IA/Glacier

**Lambda Functions:**
- Zero invocations in 90 days
- Old versions not cleaned (> 5 versions)
- Recommendation: Delete or archive

**Load Balancers:**
- HealthyHostCount = 0 for 7 days
- RequestCount = 0 for 7 days
- Recommendation: Delete ALB/NLB

**Output Format:**
```json
{
  "scan_date": "2026-06-08",
  "total_potential_savings": "$1,234.56/month",
  "resources": [
    {
      "type": "EC2",
      "id": "i-0abc123def456",
      "name": "old-web-server",
      "reason": "CPU < 3% for 7 days",
      "estimated_savings": "$45.60/month",
      "recommendation": "Rightsize from m5.xlarge to t3.medium",
      "tags": {"Environment": "staging", "Owner": "team-a"}
    }
  ]
}
```

Store results in:
- DynamoDB table (for querying)
- S3 bucket (CSV report, 90-day retention)
- SNS notification (daily summary)

### Part 4: Cost Optimization Advisor

Create a Lambda function that generates optimization recommendations:

1. **Reserved Instance Recommendations**
   - Analyze On-Demand usage patterns (minimum 14 days data)
   - Recommend 1-year vs 3-year RI
   - Calculate All Upfront vs Partial Upfront savings
   - Output: RI purchase recommendations with ROI

2. **Savings Plan Recommendations**
   - Analyze compute usage across EC2, Fargate, Lambda
   - Recommend Compute SP vs EC2 Instance SP
   - Suggest commitment amount based on baseline usage

3. **S3 Storage Class Optimization**
   - Identify objects in Standard not accessed in 30+ days
   - Recommend transition to Standard-IA (30-90 days)
   - Recommend transition to Glacier (90+ days)
   - Calculate lifecycle policy savings

4. **Right-Sizing Recommendations**
   - Analyze EC2 CPU/Memory/Network utilization
   - Recommend downsizing for underutilized instances
   - Flag instances for upgrade that are consistently at >80% utilization

**Output:** Weekly report emailed to FinOps team with:
- Total potential monthly savings
- Top 10 optimization actions by impact
- Implementation difficulty rating (Easy/Medium/Hard)
- Estimated time to implement

### Part 5: Cost Dashboard

Create a CloudWatch Dashboard with these widgets:

1. **Monthly Cost Trend** - Line chart, last 6 months
2. **Cost by Service** - Stacked bar chart, Top 10 services
3. **Cost by Environment** - Pie chart using tags
4. **Idle Resources Summary** - Table with count and savings
5. **Budget vs Actual** - Gauge chart with threshold markers
6. **Anomaly History** - Table of detected anomalies last 30 days

### Part 6: Automated Actions (Optional - Gated)

Create a Lambda for automated cleanup with approval workflow:

1. **Auto-Delete Unattached EBS** (after snapshot)
   - Gate: Only if tag `AutoCleanup=true`
   - Action: Create snapshot → Delete volume → Notify owner

2. **Auto-Release Unused EIPs**
   - Gate: Only if tag `AutoCleanup=true`
   - Action: Release EIP → Notify owner

3. **Auto-Stop Idle EC2** (non-production only)
   - Gate: Tag `Environment != production` AND `AutoCleanup=true`
   - Action: Stop instance → Notify owner

## Output Format

Generate the complete CDK project with:
- `app.py` - CDK entry point
- `stacks/budget_stack.py` - AWS Budgets
- `stacks/anomaly_stack.py` - Cost anomaly detection
- `stacks/dashboard_stack.py` - CloudWatch dashboard
- `stacks/alerting_stack.py` - SNS topics and subscriptions
- `lambda/idle_detector/` - Idle resource detection Lambda
- `lambda/cost_advisor/` - Cost optimization recommendations Lambda
- `lambda/auto_cleanup/` - Automated cleanup Lambda (optional)
- `config/thresholds.json` - Configurable thresholds
- `README.md` - Setup guide, prerequisites, cost estimates

## Constraints

- Lambda timeout: 15 minutes (for large accounts)
- Use pagination for API calls (DescribeInstances, ListObjectsV2)
- Handle throttling with exponential backoff
- All cost data in USD
- Include error handling for missing CloudWatch metrics
- Tag all resources created by this project with `ManagedBy=cost-optimizer`
- Follow AWS Well-Architected Cost Optimization pillar

## Verification

After deployment, verify:
1. Budget alerts: `aws budgets describe-budgets --account-id <id>`
2. Anomaly detector: `aws ce get-anomaly-detectors`
3. Lambda execution: Check CloudWatch Logs for idle detector
4. Dashboard: Open CloudWatch console → Dashboards
5. Test alert: Create a test resource and verify SNS notification
```

---

## Description & Use Case

### What Problem Does This Solve?

AWS bills silently grow. Teams deploy resources and forget them. A single forgotten m5.2xlarge instance costs $275/month. An unattached EBS volume costs $8/month. 50 unused Elastic IPs cost $60/month. These small wastes compound into thousands of dollars monthly.

Most teams don't optimize costs because:
- **No visibility**: Don't know where money goes
- **No automation**: Manual checks are tedious and inconsistent
- **No confidence**: Afraid to delete things that might be needed
- **No process**: No systematic approach to cost management

This prompt creates an automated cost optimization system that detects waste, provides actionable recommendations, and optionally auto-cleans with safety gates.

### Who Should Use This?

- **Startup Founders**: Maximize runway by eliminating waste
- **DevOps Teams**: Automate cost monitoring across all environments
- **FinOps Teams**: Generate reports and track optimization progress
- **Engineering Managers**: Understand team spending patterns
- **Solo Developers**: Keep personal AWS accounts lean

### Expected Outcome

After deployment, you will have:

1. Budget alerts that notify before overspending
2. Daily scans detecting idle and underutilized resources
3. Weekly optimization reports with specific savings recommendations
4. CloudWatch dashboard showing cost trends and waste
5. Optional automated cleanup for non-production resources
6. Estimated savings: 20-30% of monthly AWS spend

### Prerequisites

| Requirement | Description |
|-------------|-------------|
| AWS Account | Any AWS account with existing resources |
| IAM Permissions | AdministratorAccess or custom policy with Budgets, Cost Explorer, Lambda, CloudWatch, SNS, DynamoDB, S3 |
| AWS CLI | Installed and configured |
| CDK CLI | `npm install -g aws-cdk` |
| Python | 3.9+ |
| Cost Allocation Tags | Activated in Billing console (takes 24 hours) |

### Well-Architected Framework Alignment

| Pillar | Coverage |
|--------|----------|
| Cost Optimization | Budget management, idle resource detection, RI/SP recommendations, right-sizing |
| Operational Excellence | Automated monitoring, scheduled reports, actionable alerts |
| Security | Cost anomalies can signal security incidents (crypto mining) |

---

## Example Output

### Sample Idle Resource Detection Report

```json
{
  "scan_date": "2026-06-08",
  "account_id": "123456789012",
  "total_potential_savings": "$847.32/month",
  "summary": {
    "ec2_instances": {"count": 3, "savings": "$412.00"},
    "ebs_volumes": {"count": 12, "savings": "$96.00"},
    "elastic_ips": {"count": 5, "savings": "$36.00"},
    "rds_instances": {"count": 1, "savings": "$180.00"},
    "load_balancers": {"count": 2, "savings": "$123.32"}
  },
  "top_recommendations": [
    {
      "rank": 1,
      "resource": "i-0abc123 (old-api-server)",
      "type": "EC2",
      "current": "m5.xlarge ($140/month)",
      "recommended": "t3.medium ($30/month)",
      "reason": "CPU avg 2.1%, Memory avg 15%",
      "savings": "$110.00/month",
      "difficulty": "Easy"
    },
    {
      "rank": 2,
      "resource": "db-prod-replica",
      "type": "RDS",
      "current": "db.r5.xlarge ($180/month)",
      "recommended": "Delete",
      "reason": "Zero connections for 14 days",
      "savings": "$180.00/month",
      "difficulty": "Medium"
    }
  ]
}
```

### Sample Budget Alert Email

```
Subject: AWS Budget Alert - 80% of monthly budget consumed

Your AWS budget "Production-Environment" has reached 80% of the monthly limit.

Budget: $8,000.00/month
Current Spend: $6,423.45 (80.3%)
Forecasted End-of-Month: $9,156.00 (114.5%)

Top 3 Spending Services:
1. EC2-Instance: $3,245.00
2. RDS: $1,567.00
3. S3: $432.00

Action Required:
- Review EC2 instances for right-sizing opportunities
- Check for unused RDS instances
- Consider Reserved Instances for steady-state workloads

View details: https://console.aws.amazon.com/billing/home#/budgets
```

---

## Additional Notes

### Cost of Running This Solution

- Lambda (daily idle scan): ~$0.50/month
- Lambda (weekly advisor): ~$0.10/month
- DynamoDB (idle resource storage): ~$1.00/month
- S3 (reports storage): ~$0.50/month
- CloudWatch Dashboard: Free (up to 3 dashboards)
- Total: ~$2-3/month (negligible vs savings)

### Customization

Edit `config/thresholds.json` to adjust:
- CPU utilization threshold (default: 3%)
- Idle days threshold (default: 7)
- Cost anomaly threshold (default: $200)
- Budget amounts per environment

### Safety Features

- Auto-cleanup only works on resources tagged `AutoCleanup=true`
- Snapshots are created before deleting EBS volumes
- Production resources are never auto-deleted
- All actions are logged to CloudTrail
- Dry-run mode available for testing
