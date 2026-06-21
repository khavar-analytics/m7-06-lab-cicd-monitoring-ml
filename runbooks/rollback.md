# Rollback Runbook — Vision Moderation

## When to Roll Back
Trigger rollback if **any** of the following persist > 5 min:
- [ ] Error rate > 0.3% (exceeds 99.7% availability SLO)
- [ ] p99 latency > 500 ms
- [ ] `AvailabilityBurnFast` or `LatencyP99High` alert firing
- [ ] `ModelVersionMismatch` alert firing
- [ ] Canary error rate > 2× baseline
---
## How to Roll Back
**Option A — workflow_dispatch (preferred):**
```
GitHub → Actions → "Deploy Model Service" → Run workflow

action: rollback

target_env: production

rollback_sha: <previous-sha>
```

**Option B — CLI:**
```bash
./scripts/rollback.sh production <previous-sha>
```
Find `<previous-sha>` in: GitHub Actions → last green `deploy-production` run.
---
## What to Verify
- [ ] `AvailabilityBurnFast` / `AvailabilityBurnSlow` resolved
- [ ] `LatencyP99High` resolved
- [ ] `ModelVersionMismatch` resolved
- [ ] Error rate < 0.3% on Grafana → Vision Moderation → Overview
- [ ] p99 latency < 500 ms on Grafana → Vision Moderation → Latency
- [ ] All replicas report the previous model version (`model_version` header)
---
## Who to Notify
- [ ] **#incidents** (Slack) — open incident thread immediately
- [ ] **On-call engineer** — already paged via PagerDuty
- [ ] **ML team lead** — DM with rolled-back SHA and symptoms
- [ ] **#vision-moderation** (Slack) — brief status after rollback confirmed
---
## What NOT to Do
- [ ] Do **not** roll forward to a new version before root cause is identified
- [ ] Do **not** silence alerts before the service is stable
- [ ] Do **not** rollback staging independently — production state is the source of truth
- [ ] Do **not** restart pods as a substitute for rollback
---
## When to Roll Forward
Re-deploy only when **all** are true:
- [ ] Root cause identified and fix merged
- [ ] Fix verified in staging (smoke + canary green)
- [ ] ML team lead approves re-deploy
- [ ] Error budget has sufficient headroom (< 50% monthly budget consumed)
