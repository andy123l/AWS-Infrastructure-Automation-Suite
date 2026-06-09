# Submission 5: Serverless Full-Stack Application Scaffold

## Prompt Title
Production-Ready Serverless Full-Stack App on AWS - Complete Scaffold

## Category
Code Development

## Tags
Technical, Professional

## Recommended AI Model(s)
Claude (Anthropic), GPT-4, Amazon Bedrock (Claude 3.5 Sonnet or later)

---

## The Prompt

```
You are a senior full-stack architect specializing in AWS serverless architectures. Help me build a production-ready serverless full-stack application scaffold using AWS CDK v2 (Python).

## Context

I'm building a SaaS web application. I want to use serverless services to minimize operational overhead and costs. The application needs user authentication, a REST API, a database, file storage, and a React frontend. I want everything deployed via CI/CD with proper monitoring and security.

## Task

Generate a complete CDK Python project that creates a production-ready serverless full-stack application:

### Part 1: Authentication (Amazon Cognito)

1. **User Pool Configuration**
   - Sign-in: Email + optional username
   - Password policy: 12+ chars, uppercase, lowercase, numbers, symbols
   - MFA: Optional (TOTP)
   - Email verification required
   - Account recovery: Email only

2. **User Pool Client**
   - SPA client (no secret)
   - OAuth2 flows: Authorization code grant
   - Scopes: openid, email, profile
   - Token validity: Access 1hr, ID 1hr, Refresh 30 days

3. **Identity Pool** (for direct AWS access)
   - Authenticated role: Limited S3 access to user's folder
   - Unauthenticated role: None (deny all)

4. **User Groups**
   - `admins` - Full access
   - `users` - Standard access
   - `readonly` - Read-only access

### Part 2: REST API (API Gateway + Lambda)

1. **API Gateway Configuration**
   - REST API (not HTTP API) for full features
   - CORS enabled for frontend domain
   - Request validation (JSON Schema)
   - Throttling: 1000 requests/second burst, 500 steady
   - API Key required (usage plans)
   - CloudWatch access logging

2. **Lambda Functions (Python 3.12)**

   **Auth Function:**
   - POST /auth/register - User registration
   - POST /auth/login - User login (returns JWT)
   - POST /auth/refresh - Refresh token
   - POST /auth/forgot-password - Password reset

   **User Function:**
   - GET /users/me - Get current user profile
   - PUT /users/me - Update profile
   - GET /users/{id} - Get user by ID (admin only)
   - GET /users - List users (admin only)

   **Resource Function (example CRUD):**
   - GET /resources - List resources (paginated)
   - POST /resources - Create resource
   - GET /resources/{id} - Get resource
   - PUT /resources/{id} - Update resource
   - DELETE /resources/{id} - Delete resource

   **Common Lambda Patterns:**
   - Structured JSON logging (aws-lambda-powertools)
   - X-Ray tracing enabled
   - Environment variables from SSM Parameter Store
   - Error handling with proper HTTP status codes
   - Request validation before processing

3. **Lambda Layers**
   - Shared utilities (validation, response formatting)
   - Database connection pool
   - Authentication helpers

### Part 3: Database (DynamoDB)

1. **Main Table Design**
   - Table: `app-main-table`
   - Partition Key: `PK` (String)
   - Sort Key: `SK` (String)
   - Single-table design pattern
   - On-demand capacity mode

   **Access Patterns:**
   ```
   User profile:      PK=USER#<userId>    SK=PROFILE
   User resources:    PK=USER#<userId>    SK=RESOURCE#<resourceId>
   Resource by ID:    PK=RESOURCE#<id>    SK=META
   Resource list:     GSI1PK=RESOURCE     GSI1SK=<createdAt>
   ```

2. **Global Secondary Indexes**
   - GSI1: `GSI1PK` + `GSI1SK` (for listing resources)
   - GSI2: `GSI2PK` + `GSI2SK` (for querying by status)

3. **DynamoDB Features Enabled**
   - Point-in-time recovery (35 days)
   - Encryption at rest (AWS managed key)
   - DynamoDB Streams (for real-time processing)
   - TTL on session items

4. **Auto-Scaling** (if using provisioned capacity)
   - Min: 5 RCUs/WCUs
   - Max: 1000 RCUs/WCUs
   - Target utilization: 70%

### Part 4: File Storage (S3)

1. **Uploads Bucket**
   - Private by default
   - CORS configured for frontend
   - Lifecycle: Move to IA after 30 days, Glacier after 90
   - Versioning enabled
   - Server-side encryption (SSE-S3)

2. **User Folder Structure**
   ```
   s3://bucket/
   ├── users/
   │   ├── {userId}/
   │   │   ├── avatar/
   │   │   ├── documents/
   │   │   └── exports/
   ```

3. **Presigned URL Generation**
   - Lambda function generates presigned URLs for upload/download
   - URL validity: 15 minutes
   - Max file size: 50MB
   - Allowed file types: configurable

4. **Static Website Hosting**
   - S3 bucket for React frontend
   - CloudFront distribution
   - ACM certificate for HTTPS
   - Custom domain support

### Part 5: Frontend (React + Vite)

1. **Project Structure**
   ```
   frontend/
   ├── src/
   │   ├── components/      # Reusable UI components
   │   ├── pages/           # Page components
   │   ├── hooks/           # Custom React hooks
   │   ├── services/        # API service layer
   │   ├── context/         # React context (auth)
   │   ├── utils/           # Utility functions
   │   └── types/           # TypeScript types
   ├── public/
   ├── package.json
   ├── vite.config.ts
   └── tsconfig.json
   ```

2. **Authentication Integration**
   - AWS Amplify Auth (Cognito integration)
   - Protected routes
   - Login/Register forms
   - Password reset flow
   - Token refresh handling

3. **API Integration**
   - Axios instance with interceptors
   - Automatic token injection
   - Error handling (401 → redirect to login)
   - Request/response types

4. **UI Framework**
   - Tailwind CSS for styling
   - shadcn/ui components
   - Responsive design
   - Dark mode support

### Part 6: CI/CD Pipeline

1. **Frontend Pipeline (GitHub Actions)**
   - Trigger: Push to main branch
   - Steps: Install → Lint → Test → Build → Deploy to S3
   - CloudFront invalidation after deploy
   - Preview deployments for PRs

2. **Backend Pipeline (CodePipeline)**
   - Trigger: Push to main branch
   - Steps: Build → Test → Deploy to staging → Integration tests → Deploy to production
   - Lambda function versioning
   - Rollback capability

3. **Infrastructure Pipeline**
   - CDK deploy via GitHub Actions
   - Separate stacks: Auth, API, Database, Storage, Frontend
   - Change set review before production deploy

### Part 7: Monitoring & Observability

1. **CloudWatch Dashboard**
   - API Gateway: Request count, latency, error rate
   - Lambda: Invocations, duration, errors, throttles
   - DynamoDB: Read/write capacity, throttles
   - S3: Bucket size, request count
   - Cognito: Sign-ups, sign-ins

2. **Alarms**
   - API 5xx error rate > 1%
   - Lambda error rate > 1%
   - Lambda duration p99 > 10 seconds
   - DynamoDB throttles > 0
   - API latency p99 > 3 seconds

3. **Logging**
   - Structured JSON logging in all Lambdas
   - API Gateway access logs
   - CloudWatch Logs Insights queries
   - Log retention: 30 days

4. **X-Ray Tracing**
   - End-to-end tracing across all services
   - Service map visualization
   - Latency analysis

### Part 8: Security

1. **WAF (Web Application Firewall)**
   - Rate limiting: 2000 requests per 5 minutes per IP
   - SQL injection protection
   - XSS protection
   - Bot control

2. **Secrets Management**
   - AWS Secrets Manager for API keys
   - SSM Parameter Store for configuration
   - No hardcoded secrets in code

3. **IAM Roles**
   - Lambda execution roles with minimal permissions
   - No wildcard (*) permissions
   - Resource-level permissions where possible

4. **Data Protection**
   - Encryption at rest (DynamoDB, S3)
   - Encryption in transit (HTTPS only)
   - PII handling best practices

## Output Format

Generate the complete CDK project with:
- `app.py` - CDK entry point
- `stacks/auth_stack.py` - Cognito configuration
- `stacks/api_stack.py` - API Gateway + Lambda
- `stacks/database_stack.py` - DynamoDB tables
- `stacks/storage_stack.py` - S3 buckets
- `stacks/frontend_stack.py` - S3 + CloudFront
- `stacks/monitoring_stack.py` - Dashboard and alarms
- `stacks/security_stack.py` - WAF and security controls
- `lambda/functions/` - All Lambda function code
- `lambda/layers/` - Shared Lambda layers
- `frontend/` - React application code
- `.github/workflows/` - GitHub Actions CI/CD
- `README.md` - Complete setup guide

## Constraints

- All Lambda functions must use Python 3.12
- Use AWS Lambda Powertools for logging and tracing
- DynamoDB single-table design (no multiple tables)
- Frontend must be TypeScript (not JavaScript)
- Include environment variable configuration for dev/staging/prod
- Follow AWS Well-Architected Serverless Lens
- Include cost estimate for each service

## Verification

After deployment:
1. Register a new user via the frontend
2. Login and verify JWT token
3. Create a resource via the API
4. Upload a file to S3
5. Check CloudWatch dashboard for metrics
6. Verify X-Ray traces
7. Test error handling (invalid requests)
```

---

## Description & Use Case

### What Problem Does This Solve?

Building a full-stack application from scratch is overwhelming:
- **Too many decisions**: Which database? Which auth service? Which hosting?
- **Boilerplate fatigue**: 80% of code is infrastructure, not business logic
- **Security pitfalls**: Easy to miss CORS, auth, encryption configurations
- **Deployment complexity**: CI/CD setup takes days

This prompt generates a complete, production-ready scaffold that handles all the infrastructure concerns, so developers can focus on building business features.

### Who Should Use This?

- **Solo Founders**: Launch MVPs quickly without DevOps expertise
- **Full-Stack Developers**: Start new projects with best practices baked in
- **Startup Teams**: Build scalable SaaS applications
- **Students/Learners**: Understand production-grade serverless architecture
- **Agencies**: Standardize project templates for clients

### Expected Outcome

After deployment, you will have:

1. Complete user authentication with Cognito (registration, login, MFA)
2. REST API with Lambda functions and DynamoDB
3. File storage with S3 and presigned URLs
4. React frontend with Tailwind CSS and shadcn/ui
5. CI/CD pipeline with GitHub Actions
6. Monitoring dashboard with CloudWatch
7. WAF protection for the API
8. Estimated time to first feature: 1 hour (vs 1 week from scratch)

### Prerequisites

| Requirement | Description |
|-------------|-------------|
| AWS Account | With billing enabled |
| IAM Permissions | AdministratorAccess |
| Node.js | 18+ (for CDK and frontend) |
| Python | 3.9+ (for CDK and Lambda) |
| GitHub Account | For CI/CD pipeline |
| Domain (optional) | For custom domain |

### Well-Architected Framework Alignment

| Pillar | Coverage |
|--------|----------|
| Security | Cognito auth, WAF, encryption, IAM least privilege |
| Reliability | Multi-AZ, auto-scaling, error handling |
| Performance Efficiency | DynamoDB on-demand, Lambda cold start optimization |
| Cost Optimization | Serverless pay-per-use, S3 lifecycle policies |
| Operational Excellence | CI/CD, monitoring, structured logging |

---

## Example Output

### Sample API Response

```json
// GET /resources?page=1&limit=10
{
  "data": [
    {
      "id": "res_abc123",
      "title": "My First Resource",
      "description": "A sample resource",
      "status": "active",
      "createdAt": "2026-06-08T10:00:00Z",
      "updatedAt": "2026-06-08T10:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 42,
    "hasNext": true
  }
}
```

### Sample Lambda Function

```python
from aws_lambda_powertools import Logger, Tracer
from aws_lambda_powertools.event_handler import APIGatewayRestResolver
from aws_lambda_powertools.utilities.typing import LambdaContext

logger = Logger()
tracer = Tracer()
app = APIGatewayRestResolver()

@app.post("/resources")
@tracer.capture_method
def create_resource():
    user = app.current_event.request_context.authorizer
    body = app.current_event.json_body

    # Validate input
    if not body.get("title"):
        return {"statusCode": 400, "body": {"error": "Title is required"}}

    # Create in DynamoDB
    resource = {
        "PK": f"USER#{user['sub']}",
        "SK": f"RESOURCE#{uuid.uuid4()}",
        "title": body["title"],
        "description": body.get("description", ""),
        "status": "active",
        "createdAt": datetime.utcnow().isoformat(),
    }
    table.put_item(Item=resource)

    logger.info("Resource created", extra={"resource_id": resource["SK"]})
    return {"statusCode": 201, "body": resource}

@logger.inject_lambda_context
@tracer.capture_lambda_handler
def handler(event: dict, context: LambdaContext) -> dict:
    return app.resolve(event, context)
```

### Sample CloudWatch Dashboard

```
┌─────────────────────────────────────────────────────────┐
│           Serverless App Dashboard                       │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  API Requests (24h): 12,456                              │
│  Error Rate: 0.3% ████░░░░░░░░░░                        │
│  Latency p50: 45ms  p99: 890ms                          │
│                                                          │
│  Lambda Invocations: 8,234                               │
│  Lambda Errors: 12 (0.15%)                               │
│  Lambda Duration p99: 2.3s                               │
│                                                          │
│  DynamoDB:                                               │
│    Read Capacity: 25 RCU (auto-scaled)                   │
│    Write Capacity: 10 WCU (auto-scaled)                  │
│    Throttles: 0                                          │
│                                                          │
│  Cognito:                                                │
│    Sign-ups (7d): 45                                     │
│    Sign-ins (7d): 312                                    │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Additional Notes

### Cost Estimate (Small Scale)

- API Gateway: ~$3.50 per million requests
- Lambda: ~$0.20 per million invocations + $0.0000166667 per GB-second
- DynamoDB: On-demand pricing, ~$1.25 per million write requests
- S3: ~$0.023 per GB stored
- Cognito: Free for first 50,000 MAUs
- CloudFront: ~$0.085 per GB transferred
- **Total for small app**: ~$20-50/month

### Customization

- **Database**: Replace DynamoDB with Aurora Serverless for relational needs
- **Auth**: Add social login (Google, GitHub) via Cognito
- **Real-time**: Add WebSocket API for real-time features
- **Search**: Add OpenSearch Serverless for full-text search
- **Email**: Add SES for transactional emails
- **Payments**: Add Stripe integration via Lambda

### Monorepo vs Polyrepo

This scaffold uses a monorepo structure:
```
project/
├── infra/          # CDK infrastructure
├── backend/        # Lambda functions
├── frontend/       # React application
└── shared/         # Shared types/utils
```

This keeps everything in one repository for easier development and deployment.

### Migration Path

When you outgrow this scaffold:
- **Database**: DynamoDB → Aurora Serverless v2
- **API**: API Gateway → ALB (for WebSocket support)
- **Auth**: Cognito → Auth0 or Clerk (for advanced features)
- **Frontend**: S3+CloudFront → Vercel or AWS Amplify Hosting
