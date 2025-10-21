# dryFlyStream# DryFly Systems

A professional, demo that showcases an industry-standard stack: **AWS (API Gateway + Lambda + RDS MySQL)**, **Node.js (Fastify)**, **OpenAPI 3.1**, **Terraform IaC**, and **Salesforce** integration (pull schedules and optional push).

## Goals
- Demonstrate secure, production-style API + database + CI/CD.
- Sync data between MySQL and Salesforce (pull daily/weekly/monthly; optional push on change).
- Keep costs ultra-low while adhering to best practices.

## Repositories (suggested)
- `dryfly-systems-iac` — Terraform modules, envs; S3 state + DynamoDB lock.
- `dryfly-systems-api` — Fastify API with Prisma, OpenAPI 3.1, security headers, JWT.
- `dryfly-systems-web` — React + Ant Design admin UI (basic list/detail).

## Data Model (Education CRM)
Entities and Salesforce mapping:
- **School** → SF **Account** (Type = "School")
- **Contact** → SF **Contact** (lookup to Account)
- **Application** → SF **Opportunity** (Record Type "Application"; StageName ↔ status)
- **Event** → SF **Campaign** (Type "Event")
- **Registration** → SF **CampaignMember**

## Quickstart (Local, No Docker)
### Prerequisites
- Node.js 20+, MySQL 8+, Terraform 1.9+, AWS CLI
- VS Code extensions: Terraform, Prisma, YAML

### Clone
```bash
mkdir -p ~/Projects/dryfly-systems && cd ~/Projects/dryfly-systems
# (Create your GitHub repos first, then clone them)
```

### API Setup (dryfly-systems-api)
1. Install deps: `npm ci`
2. Configure `.env` (local dev):
```
DATABASE_URL="mysql://root:password@localhost:3306/dryfly"
JWT_SECRET="a-strong-local-secret"
NODE_ENV="development"
```
3. Prisma & seed:
```bash
npm run prisma:migrate
npm run seed
```
4. Run: `npm run dev`

### Web Setup (dryfly-systems-web)
- Standard React + Ant Design app. Point API URL to your local API.

## AWS & Terraform
1. Create Terraform backend:
```bash
aws s3 mb s3://dryfly-tfstate
aws dynamodb create-table   --table-name dryfly-tf-lock   --attribute-definitions AttributeName=LockID,AttributeType=S   --key-schema AttributeName=LockID,KeyType=HASH   --billing-mode PAY_PER_REQUEST
```
2. In `dryfly-systems-iac`, configure backend to S3/DynamoDB above.
3. `terraform init && terraform plan` (dev), then `terraform apply`.

### Infrastructure (frugal defaults)
- **API:** Lambda (Node.js) behind API Gateway (REST)
- **DB:** RDS MySQL (db.t3.micro, single-AZ)
- **Secrets:** AWS Secrets Manager (DB creds + JWT)
- **Logs/Metrics:** CloudWatch
- **Auth:** JWT (HS256), upgrade path to Cognito later

## Security & Headers
- HSTS, X-Content-Type-Options, Referrer-Policy, Permissions-Policy, CSP
- CORS allowlist: `http://localhost:3000` (add prod domain later)

## Salesforce Integration
- **Auth:** Named Credentials (OAuth 2.0)
- **Pull:** Scheduled Flow → HTTP Callout → `POST /sync/sf/pull?since=...` (daily/weekly/monthly)
- **Push (stretch):** Record-Triggered Flow → HTTP Callout → `POST /sync/sf/push` with `Idempotency-Key`

## Testing
- Unit & integration tests (Jest, supertest)
- Contract validation against OpenAPI
- Optional load (k6 @ 100 RPS for /applications reads)

## Scripts (suggested)
- `npm run dev` — start API locally
- `npm run test` — run tests
- `npm run prisma:migrate` — apply DB migrations
- `npm run seed` — seed dev data

## License
MIT
