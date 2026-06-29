# Deployment Guide

## ECS Rolling Deploy Strategy

PredictIQ uses ECS Fargate with a **zero-downtime rolling deploy** strategy.

### Configuration

| Setting | Value | Rationale |
|---|---|---|
| `deployment_minimum_healthy_percent` | 100 | Never reduce capacity below 100 % during a deploy |
| `deployment_maximum_percent` | 200 | Allow new tasks to start before old tasks are drained |
| `deployment_circuit_breaker.enable` | true | Automatically detect a failed deploy |
| `deployment_circuit_breaker.rollback` | true | Automatically roll back to the previous task definition on failure |

### Deploy Sequence

1. **Run database migrations** before deploying new tasks.  
   Migrations must be backward-compatible with the currently running task version so both versions can co-exist during the rollover window.

2. **ECS starts new tasks** (up to `maximum_percent` total capacity) using the new task definition.

3. **ALB health checks** confirm the new tasks are healthy (`/health` returns 200).

4. **ECS drains old tasks** — the load balancer stops routing traffic to them and waits for in-flight requests to complete before terminating.

5. **Circuit breaker monitors** the deploy. If the new tasks fail health checks repeatedly, ECS automatically rolls back to the previous stable task definition and raises a `SERVICE_DEPLOYMENT_FAILED` CloudWatch event.

### Deployment Steps (Manual)

```bash
# 1. Build and push the new image
docker build -t predictiq-api:$VERSION .
docker push $ECR_REPO:$VERSION

# 2. Run migrations against the target environment
DATABASE_URL=$PROD_DATABASE_URL ./services/api/scripts/run_migrations.sh

# 3. Update the ECS service (GitHub Actions does this automatically on merge to main)
aws ecs update-service \
  --cluster predictiq-prod \
  --service predictiq-prod-api \
  --task-definition predictiq-prod-api:$NEW_REVISION \
  --region us-east-1

# 4. Wait for the deploy to stabilise
aws ecs wait services-stable \
  --cluster predictiq-prod \
  --services predictiq-prod-api \
  --region us-east-1
```

### Rollback (Manual)

If automatic rollback did not trigger:

```bash
# Find the last known-good task definition revision
aws ecs describe-services \
  --cluster predictiq-prod \
  --services predictiq-prod-api \
  --query 'services[0].deployments'

# Force rollback to a specific revision
aws ecs update-service \
  --cluster predictiq-prod \
  --service predictiq-prod-api \
  --task-definition predictiq-prod-api:$PREVIOUS_REVISION \
  --region us-east-1
```

### AWS CodeDeploy Blue-Green — Assessment

A full blue-green deployment (via AWS CodeDeploy) would eliminate even the brief overlap window of rolling deploys by switching traffic at the ALB listener level in one atomic step.

**Warranted when:**
- Migrations are not backward-compatible and cannot be made so.
- The team needs a well-defined, instant traffic cutover with a single-click rollback.
- Compliance requirements mandate zero request failures during deploy.

**Current recommendation:** The rolling strategy with `minimum_healthy_percent = 100` already delivers zero-downtime deploys for backward-compatible migrations. CodeDeploy blue-green adds significant operational complexity (separate target groups, CodeDeploy application/deployment group resources, lifecycle hook Lambdas). Given that PredictIQ migrations are currently written to be additive and backward-compatible, the rolling strategy is sufficient. Revisit this decision if a migration arises that requires a hard cutover.
