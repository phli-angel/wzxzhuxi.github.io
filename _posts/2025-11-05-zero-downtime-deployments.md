---
layout: post
title: "Zero-Downtime Deployments: A DevOps Journey"
date: 2025-11-05 16:45:00 +0800
categories: devops deployment kubernetes
author: Sam Rodriguez
---

Deploying during business hours without downtime used to be a pipe dream. Not anymore. Here's how we achieved true zero-downtime deployments.

## The Old Way (Pain)

- Deploy at 2 AM
- Hope nothing breaks
- Wake up to alerts anyway
- Rollback in panic

Sound familiar?

## The New Way

### 1. Blue-Green Deployments

We run two identical production environments. Deploy to the idle one, test, then switch traffic.

```yaml
# Kubernetes service switching
apiVersion: v1
kind: Service
metadata:
  name: app-service
spec:
  selector:
    app: myapp
    version: blue  # Switch to 'green' when ready
  ports:
    - port: 80
```

**Advantages**:
- Instant rollback (just switch back)
- Full environment testing before go-live
- Zero user impact

**Drawback**: 2x infrastructure cost (we optimize by scaling down blue after switch)

### 2. Health Checks Are Critical

Kubernetes won't route traffic until your app is actually ready.

```yaml
readinessProbe:
  httpGet:
    path: /health/ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 3

livenessProbe:
  httpGet:
    path: /health/live
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 10
```

Our `/health/ready` checks:
- Database connectivity
- Redis cache
- External API availability

**Lesson learned**: Don't just return HTTP 200. Actually verify dependencies.

### 3. Rolling Updates with Pod Disruption Budgets

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: app-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: myapp
```

This ensures Kubernetes never kills too many pods during updates. We maintain minimum capacity.

### 4. Database Migrations (The Hard Part)

Schema changes can break deployments. Our approach:

**Backward-compatible migrations**:
1. Add new column (nullable)
2. Deploy code that writes to both old and new
3. Backfill data
4. Deploy code that reads from new column
5. Remove old column (separate migration)

**Never**:
- Drop columns in the same release as code changes
- Rename columns directly (add new, migrate, remove old)
- Change data types without intermediate steps

### 5. Feature Flags

Deploy dark. Enable gradually.

```javascript
if (featureFlags.isEnabled('new-checkout', user.id)) {
  return <NewCheckout />;
}
return <LegacyCheckout />;
```

We use LaunchDarkly. Canary new features to 5% of users, monitor errors, then gradually roll out.

## Monitoring During Deployments

Our deployment dashboard shows:
- Error rate (spike = rollback)
- Response time p95
- Active requests
- CPU/Memory usage

Auto-rollback triggers:
- Error rate > 1% for 2 minutes
- P95 latency > 2x baseline
- Any HTTP 5xx > 0.5%

## Results

**Before**:
- 3-4 production incidents per month
- Deployments: Friday 10 PM only
- Average rollback time: 25 minutes

**After**:
- 0.3 incidents per month (94% reduction)
- Deploy anytime (20+ deploys/week)
- Rollback time: 45 seconds

## Tools We Use

- **Kubernetes**: Orchestration
- **ArgoCD**: GitOps-based deployments
- **Prometheus + Grafana**: Monitoring
- **Flagger**: Progressive delivery (canary, A/B)

## Next Steps

We're experimenting with:
- Service mesh (Linkerd) for traffic splitting
- Automated canary analysis (ML-based anomaly detection)

Zero-downtime deployments aren't magic - they're process, tooling, and discipline.

---

*Want to know more about our DevOps setup? Ask away!*
