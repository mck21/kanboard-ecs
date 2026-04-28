# Troubleshooting — Issues Encountered

Real problems hit during deployment and how they were solved. This is my personal experience.

---

## 1. ECS Cluster creation failed on first attempt

**Error:** Stack CREATE_FAILED on first cluster creation attempt.

**Cause:** The IAM role `AWSServiceRoleForECS` does not exist until the ECS console is visited for the first time. The role is created automatically on the first console access, but if you try to create a cluster before it propagates, it fails.

**Fix:** Exit ECS console, re-enter it, then create the cluster again.

---

## 2. Kanboard DB migration error after redeployment

**Error:**
```
Internal Error: Unable to run SQL migrations: Running migration \Schema\version_130 ...
SQLSTATE[42S21]: Column already exists: 1060 Duplicate column name 'color_id'
```

**Cause:** The RDS database had been partially initialized by a previous task. When a new task started, Kanboard tried to re-run migrations that had already run, causing column conflicts.

**Fix:**
```sql
DROP DATABASE kanboard;
CREATE DATABASE kanboard CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
FLUSH PRIVILEGES;
```
Then also clear the EFS data volume:
```bash
sudo rm -rf /mnt/efs/*
```
Stop the running task and let ECS relaunch it — clean initialization.

---

## 3. EFS mount timeout in ECS tasks

**Error (in task logs):**
```
ResourceInitializationError: failed to invoke EFS utils commands to set up EFS volumes:
Mount attempt 1/3 failed due to timeout after 15 sec
mount.nfs4: Connection timed out
```

**Cause:** The ECS tasks were launched without a Security Group assigned (the service was created without selecting `GS-KANBOARD-SERVICE`). Without a SG, no inbound NFS traffic from the tasks could reach the EFS mount targets.

**Fix:** Delete and recreate the service, making sure to explicitly select `GS-KANBOARD-SERVICE` in the Networking section. The SG must have an inbound NFS (port 2049) rule sourced from `GS-KANBOARD-SERVICE`.

---

## 4. ALB targets stuck in Unhealthy state

**Symptom:** Load balancer resource map shows all targets as Unhealthy even though tasks are running.

**Cause:** The ALB health check only accepted HTTP `200` as a success code. Kanboard returns HTTP `302` (redirect to login) on the root path `/`, which the health check treated as a failure.

**Fix:**
1. EC2 → Target Groups → Kanboard-TG → Health checks → Edit
2. Set **Success codes** to `200,302`
3. Wait ~2 minutes for health checks to pass

---

## 5. ECS Deployment Circuit Breaker triggered

**Error:** `ECS Deployment Circuit Breaker was triggered`

**Cause:** This happened twice for different reasons:
- First time: tasks had no Security Group → EFS mount failed → tasks kept stopping → circuit breaker fired
- Second time: ALB health checks were failing (issue #4 above) before the 120s grace period expired

**Fix:** Resolve the underlying task failure first (EFS SG or health check codes), then either update the service or delete and recreate it.


