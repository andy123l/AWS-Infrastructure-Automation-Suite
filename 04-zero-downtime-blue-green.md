# Submission 4: Zero-Downtime Blue-Green Deployment

## Prompt Title
Zero-Downtime Blue-Green Deployment Pipeline for AWS

## Category
Code Development

## Tags
Technical, Professional

## Recommended AI Model(s)
Claude (Anthropic), GPT-4, Amazon Bedrock (Claude 3.5 Sonnet or later)

---

## The Prompt

```
You are a senior DevOps architect specializing in AWS deployment strategies. Help me build a production-ready zero-downtime blue-green deployment pipeline using AWS CDK v2 (Python).

## Context

Our team deploys a containerized web application (ECS Fargate) to production 3-5 times per week. We've had 2 production incidents in the last month caused by bad deployments. We need automated blue-green deployments with instant rollback capability. The team uses GitHub for source code and wants CI/CD with AWS CodePipeline.

## Task

Generate a complete CDK Python project that implements blue-green deployments:

### Part 1: ECS Fargate Blue-Green Architecture

Create the infrastructure:

1. **VPC Configuration**
   - 3 AZs with public and private subnets
   - NAT Gateway for private subnet internet access
   - VPC endpoints for ECR, CloudWatch Logs, S3

2. **ECS Cluster**
   - ECS Cluster with Container Insights enabled
   - Capacity providers: FARGATE and FARGATE_SPOT (70/30 mix)

3. **ALB Configuration**
   - Application Load Balancer with two target groups:
     - Blue target group (production traffic)
     - Green target group (new deployment)
   - ALB listener rules for traffic switching
   - Health check: HTTP /health, 200 OK, 30s interval

4. **Task Definition**
   - Fargate task with configurable CPU/Memory (default: 512/1024)
   - Environment variables from SSM Parameter Store
   - Secrets from AWS Secrets Manager
   - CloudWatch logging with JSON format
   - X-Ray tracing enabled

### Part 2: CodePipeline CI/CD Pipeline

Create a 4-stage pipeline:

**Stage 1: Source**
- GitHub repository (main branch)
- Webhook for automatic trigger on push
- Source artifact stored in S3

**Stage 2: Build (CodeBuild)**
- Build Docker image
- Run unit tests
- Run security scan (Trivy for container vulnerabilities)
- Push to ECR with commit SHA tag
- Generate buildspec.yml

**Stage 3: Deploy Staging**
- Deploy to staging ECS service (green target group)
- Run integration tests against staging
- Smoke test: HTTP 200 on /health, /api/status
- Performance test: p99 latency < 500ms

**Stage 4: Deploy Production (Blue-Green)**
- CodeDeploy blue-green deployment:
  1. Deploy new version to green target group
  2. Wait for green health checks to pass
  3. Shift 10% traffic to green (canary)
  4. Monitor error rate for 5 minutes
  5. If error rate < 1%, shift 100% traffic
  6. If error rate >= 1%, auto-rollback
  7. Terminate old blue tasks after 15 minutes

### Part 3: Deployment Configuration

Create CodeDeploy Application with:

1. **Deployment Strategy: Canary 10%**
   - 10% traffic shift → 5 minute bake time → 100% shift
   - Rollback if CloudWatch alarm triggers

2. **CloudWatch Alarms for Auto-Rollback**
   - HTTP 5xx error rate > 1% (2 consecutive periods)
   - Target response time p99 > 3 seconds (2 consecutive periods)
   - Healthy host count < 1 (1 period)
   - ECS service running count < desired count (2 periods)

3. **Pre-Traffic Hook**
   - Lambda function that runs smoke tests
   - Validates database connectivity
   - Checks external API dependencies
   - Returns PASS/FAIL to CodeDeploy

4. **Post-Traffic Hook**
   - Lambda function that validates production health
   - Runs end-to-end test suite
   - Sends deployment notification to Slack
   - Returns PASS/FAIL to CodeDeploy

### Part 4: Monitoring & Observability

1. **CloudWatch Dashboard**
   - Deployment status widget
   - Error rate comparison (blue vs green)
   - Latency comparison (blue vs green)
   - Request count per target group
   - ECS service CPU/Memory utilization

2. **Alarms**
   - Deployment failure alarm → SNS → Slack
   - Rollback triggered alarm → SNS → PagerDuty
   - Health check failure alarm → SNS → Email

3. **X-Ray Integration**
   - Trace requests across services
   - Identify latency bottlenecks
   - Compare performance between versions

### Part 5: Rollback Automation

Create a Lambda function for instant rollback:

1. **Manual Rollback Trigger**
   - API Gateway endpoint: POST /rollback
   - SSM Parameter Store trigger
   - CLI command: `aws deploy stop-deployment --deployment-id <id>`

2. **Automatic Rollback Conditions**
   - CloudWatch alarm triggers
   - Pre-traffic hook fails
   - Post-traffic hook fails
   - Health check failures exceed threshold

3. **Rollback Process**
   - Shift 100% traffic back to blue target group
   - Terminate green tasks
   - Send notification with rollback reason
   - Log rollback event to CloudTrail

### Part 6: Infrastructure as Code

Generate CDK constructs for reusability:

1. **VpcConstruct** - Reusable VPC with best practices
2. **EcsClusterConstruct** - ECS cluster with capacity providers
3. **AlbConstruct** - ALB with blue-green target groups
4. **PipelineConstruct** - CodePipeline with all stages
5. **MonitoringConstruct** - Dashboard and alarms

## Output Format

Generate the complete CDK project with:
- `app.py` - CDK entry point
- `stacks/vpc_stack.py` - VPC and networking
- `stacks/ecs_stack.py` - ECS cluster and services
- `stacks/alb_stack.py` - Load balancer configuration
- `stacks/pipeline_stack.py` - CodePipeline CI/CD
- `stacks/monitoring_stack.py` - Dashboard and alarms
- `constructs/` - Reusable CDK constructs
- `lambda/hooks/` - Pre/post traffic hook functions
- `lambda/rollback/` - Rollback automation
- `buildspec.yml` - CodeBuild build specification
- `appspec.yaml` - CodeDeploy specification
- `README.md` - Setup guide and deployment instructions

## Constraints

- All infrastructure in private subnets (no public ECS tasks)
- ALB must have WAF enabled for production
- ECR images scanned for vulnerabilities on push
- Secrets managed via AWS Secrets Manager (not environment variables)
- All resources tagged with Environment, Project, ManagedBy
- Follow AWS Well-Architected Reliability and Operational Excellence pillars
- Include cost estimate for running this infrastructure

## Verification

After deployment:
1. Push a code change to GitHub
2. Monitor pipeline execution in CodePipeline console
3. Verify staging deployment and tests pass
4. Verify production blue-green deployment
5. Test rollback by triggering a failure condition
6. Check CloudWatch dashboard for deployment metrics
```

---

## Description & Use Case

### What Problem Does This Solve?

Production deployments are stressful. A bad deployment can cause:
- **Downtime**: Users can't access the application
- **Data corruption**: Bad code writes bad data
- **Rollback panic**: Manual rollback takes 30+ minutes
- **Team anxiety**: Developers afraid to deploy

Blue-green deployments solve this by maintaining two identical environments. Traffic switches instantly, and rollback is just a traffic switch back. This prompt creates a complete, automated blue-green deployment pipeline that deploys safely and rolls back instantly.

### Who Should Use This?

- **DevOps Engineers**: Implement production-grade deployment pipelines
- **Backend Developers**: Deploy containerized applications safely
- **Engineering Managers**: Reduce deployment-related incidents
- **SRE Teams**: Improve deployment reliability and observability
- **Startup CTOs**: Build deployment infrastructure that scales

### Expected Outcome

After deployment, you will have:

1. Fully automated CI/CD pipeline from GitHub to production
2. Zero-downtime deployments with canary analysis
3. Instant rollback capability (automatic and manual)
4. Comprehensive monitoring dashboard for deployments
5. Pre/post deployment validation hooks
6. Estimated deployment time: 10-15 minutes (including tests)

### Prerequisites

| Requirement | Description |
|-------------|-------------|
| AWS Account | With appropriate service limits |
| IAM Permissions | AdministratorAccess or custom policy |
| GitHub Repository | With containerized application |
| Docker | For local testing (optional) |
| AWS CLI | Installed and configured |
| CDK CLI | `npm install -g aws-cdk` |
| Python | 3.9+ |
| Domain (optional) | For custom domain with ACM certificate |

### Well-Architected Framework Alignment

| Pillar | Coverage |
|--------|----------|
| Reliability | Multi-AZ deployment, automatic rollback, health checks |
| Operational Excellence | Automated pipeline, monitoring, deployment notifications |
| Security | WAF, secrets management, container scanning, private subnets |
| Performance Efficiency | Canary analysis, latency monitoring, capacity providers |

---

## Example Output

### Sample Deployment Flow

```
[2026-06-08 14:30:00] Developer pushes code to main branch
[2026-06-08 14:30:05] GitHub webhook triggers CodePipeline
[2026-06-08 14:30:10] Stage 1: Source - Code pulled from GitHub
[2026-06-08 14:31:30] Stage 2: Build - Docker image built and pushed to ECR
[2026-06-08 14:31:35] Stage 2: Security scan - No critical vulnerabilities
[2026-06-08 14:33:00] Stage 3: Staging - Deployed to staging environment
[2026-06-08 14:35:00] Stage 3: Integration tests - All 47 tests passed
[2026-06-08 14:36:00] Stage 4: Production - Blue-green deployment started
[2026-06-08 14:36:30] Stage 4: Green tasks healthy (5/5)
[2026-06-08 14:37:00] Stage 4: Pre-traffic hook - PASSED
[2026-06-08 14:37:05] Stage 4: Shifting 10% traffic to green
[2026-06-08 14:42:05] Stage 4: Bake time complete, error rate 0.2%
[2026-06-08 14:42:10] Stage 4: Shifting 100% traffic to green
[2026-06-08 14:42:15] Stage 4: Post-traffic hook - PASSED
[2026-06-08 14:42:20] Deployment complete! Total time: 12m 20s
[2026-06-08 14:57:20] Old blue tasks terminated (15min cooldown)
```

### Sample Rollback Scenario

```
[2026-06-08 15:00:00] Deployment started for v2.3.0
[2026-06-08 15:08:00] 10% traffic shifted to green
[2026-06-08 15:09:30] CloudWatch alarm triggered: 5xx error rate 3.2%
[2026-06-08 15:09:35] Auto-rollback initiated
[2026-06-08 15:09:40] 100% traffic shifted back to blue
[2026-06-08 15:09:45] Green tasks terminated
[2026-06-08 15:09:50] Rollback complete. Notification sent to Slack
[2026-06-08 15:09:55] Incident: v2.3.0 deployment failed, auto-rolled back
```

### Sample CloudWatch Dashboard

```
┌─────────────────────────────────────────────────────────┐
│           Deployment Dashboard                           │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Last Deployment: v2.3.1 (2 hours ago)                   │
│  Status: ✅ Successful                                    │
│  Duration: 12m 20s                                       │
│                                                          │
│  Error Rate (Blue):  0.1% ████░░░░░░                     │
│  Error Rate (Green): 0.2% █████░░░░░                     │
│                                                          │
│  Latency p99 (Blue):  180ms                              │
│  Latency p99 (Green): 195ms                              │
│                                                          │
│  Requests/min: 1,245                                     │
│  Healthy Hosts: 5/5                                      │
│                                                          │
│  Rollbacks (30 days): 1                                  │
│  Success Rate (30 days): 96% (24/25)                     │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Additional Notes

### Cost Estimate

- ALB: ~$22/month + data processing
- ECS Fargate (2 services x 2 tasks): ~$70/month
- NAT Gateway: ~$45/month + data processing
- CodePipeline: ~$1/month (first pipeline free)
- CodeBuild: ~$5/month (100 build minutes)
- CloudWatch: ~$10/month
- Total: ~$153/month (excludes data transfer)

### Customization Options

- **Deployment strategy**: Modify canary percentages and bake times
- **Scaling**: Add auto-scaling policies for ECS services
- **Multi-region**: Extend to multi-region with Route 53 failover
- **Database migrations**: Add pre-deployment migration hooks
- **Feature flags**: Integrate with AWS AppConfig for feature toggles

### Alternative Deployment Strategies

This prompt uses canary (10% → 100%), but can be modified for:
- **Linear**: 10% → 20% → 30% → ... → 100% (every 1 minute)
- **All-at-once**: 0% → 100% (for non-critical services)
- **Custom**: Define your own traffic shifting strategy
