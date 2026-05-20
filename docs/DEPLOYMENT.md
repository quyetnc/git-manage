# Deployment Guide

## Branch Strategy & GitHub Flow

This project follows **GitHub Flow**, a simple and effective branching model for continuous delivery. We maintain a single `main` branch that is always production-ready, with all feature development happening in short-lived feature branches (`feature/*`). Each feature branch is created from `main`, developed independently, tested thoroughly via pull request review, and merged back to `main` once approved—eliminating the complexity of multiple long-lived branches (main, develop, release) and reducing merge conflicts. This decision prioritizes rapid iteration and reduced cognitive overhead while requiring strong CI/CD pipelines and comprehensive test coverage to maintain `main` stability.

## Branching Naming Convention

```
feature/{author}/{short-description}    # New features or enhancements
bugfix/{author}/{issue-id}              # Bug fixes
docs/{author}/{topic}                   # Documentation updates
```

**Examples:**
- `feature/quyet/add-deployment-docs`
- `bugfix/hao/fix-cors-headers` 

## Deployment Process

### 1. Development Phase

1. Create a feature branch from `main`:
   ```bash
   git checkout -b feature/{author}/{description}
   ```

2. Make meaningful commits with clear messages:
   ```bash
   git commit -m "feat: add user authentication endpoint"
   git commit -m "test: add auth controller tests"
   git commit -m "docs: update API documentation"
   ```

3. Push to remote and create a Pull Request:
   ```bash
   git push origin feature/{author}/{description}
   ```

### 2. Review & Testing

All PRs must satisfy these checks before merging to `main`:

- **Code Review**: At least 1 approval required
- **Automated Tests**: All tests passing (CI pipeline)
- **Test Coverage**: Maintain ≥80% code coverage
- **Linting**: Code style and lint checks pass
- **Documentation**: Updated README/docs if behavior changed

Use the PR template to describe:
- **What**: What changed and why
- **How to test**: Exact reproduction steps
- **Risk**: Potential impact (low/medium/high)

### 3. Merge & Deploy to Staging

Once approved:
1. Squash commits for clean history (optional but recommended)
2. Merge to `main` using GitHub UI
3. Delete the feature branch
4. Automated CI/CD pipeline deploys to staging environment
5. Run smoke tests against staging

### 4. Production Deployment

Deployments to production are manual (human-gated):
```bash
# On main branch after merged PR
npm run build       # Build application
npm test            # Run full test suite
npm run deploy      # Deploy to production
```

Or via GitHub Actions:
```bash
git tag v1.0.0      # Create version tag
git push origin v1.0.0  # Push tag to trigger production deployment
```

## Rollback Strategy

### Quick Rollback (Last 30 minutes)

```bash
git revert HEAD --no-edit
git push origin main
```

### Full Rollback (Any time)

```bash
git checkout main
git log --oneline -10        # Find commit hash of last good state
git reset --hard {commit-hash}
git push origin main --force  # Force push to revert
```

**Important**: Use force push only in emergencies. Always notify the team.

### Health Checks After Deployment

```bash
# Monitor application health
curl -s http://app:3000/health | jq .

# Check for error spikes in logs
docker logs app | tail -100 | grep ERROR

# Verify database migrations ran
npm run verify:migrations
```

## Emergency Hotfixes

For critical production bugs:

1. Create `hotfix/` branch from `main` (not from feature branches)
2. Make minimal, focused fix
3. Bypass normal PR review (single approval + on-call lead)
4. Merge directly to `main`
5. Tag for immediate production deploy
6. Create follow-up task to prevent recurrence

Example:
```bash
git checkout -b hotfix/quyet/database-connection-leak
# Make fix
git push origin hotfix/quyet/database-connection-leak
# Create PR with "🚨 HOTFIX" label for fast-track review
```

## Monitoring & Observability

Post-deployment checklist:

- [ ] Error rate < 0.1% (check Datadog/CloudWatch)
- [ ] P95 latency < 500ms (check APM dashboard)
- [ ] Database connections stable (check connection pool)
- [ ] No concerning log spikes (search ERROR/WARN)
- [ ] Uptime monitor shows 100% availability
- [ ] User-facing features work as expected

## Infrastructure as Code (IaC)

All infrastructure changes (new services, config, permissions) must:
1. Be defined in Terraform (`.tf` files)
2. Pass `terraform plan` validation
3. Be reviewed and approved alongside code changes
4. Be applied only after main deployment succeeds

```bash
# For infrastructure changes
terraform plan -out=tfplan
# Review tfplan carefully
terraform apply tfplan
```

## Rollback Checklist

In case of production issues:

```bash
[ ] Notify team on Slack #deployments
[ ] Gather context: What changed? What's broken?
[ ] Check recent commits: git log --oneline -5
[ ] Identify last good commit/tag
[ ] Decide: Revert vs Hotfix (revert if unsure)
[ ] Execute rollback (see section above)
[ ] Run health checks (see Monitoring section)
[ ] Create post-mortem ticket
[ ] Update team
```

## FAQ

**Q: Can I push directly to main?**  
A: No. Always use feature branches and PRs. Direct pushes bypass review and testing.

**Q: How long should a feature branch live?**  
A: Ideally 1-3 days. Longer branches risk larger merge conflicts and longer review cycles.

**Q: What if my PR conflicts with main?**  
A: Pull latest main into your branch and resolve conflicts locally:
```bash
git fetch origin
git merge origin/main
# Resolve conflicts, commit, push
```

**Q: Can I merge my own PR?**  
A: Only in emergencies. Always get peer review—it catches bugs and spreads knowledge.

**Q: What's the difference between revert and reset?**  
A: `revert` creates a new commit that undoes changes (safe, keeps history). `reset` rewrites history (dangerous, use only for local branches or emergencies).

---

**Last Updated**: 2026-05-20  
**Maintainer**: Quyet Nguyen (@quyetnc)
