# AWS Cost Estimate

Estimated monthly costs for this architecture running in **us-east-1**, based on the configuration used in this project.

> All prices in USD. Based on AWS public pricing as of early 2026. Use the [AWS Pricing Calculator](https://calculator.aws) for up-to-date estimates.

---

## Lab Configuration (this project)

| Service | Config | Est. Monthly Cost |
|---|---|---|
| ECS Fargate | 2 tasks × 0.25 vCPU × 0.5 GB, ~720h/mo | ~$6 |
| RDS MySQL | db.t3.micro, 20 GiB gp3, Single-AZ | ~$15 |
| EFS | Regional, ~1 GB stored, Bursting | ~$0.30 |
| ALB | 1 ALB, ~2 LCU | ~$18 |
| EC2 (test instance) | t2.micro, ~40h | ~$0.50 |
| Data transfer | Minimal (lab) | ~$1 |
| Secrets Manager | 2 secrets | ~$0.80 |
| CloudWatch | Container Insights, basic metrics | ~$3 |
| **Total (approx.)** | | **~$44/month** |

---

## Production-ready Configuration

For a real production deployment with HA, backups, and encryption:

| Service | Config | Est. Monthly Cost |
|---|---|---|
| ECS Fargate | 4 tasks avg × 0.5 vCPU × 1 GB | ~$50 |
| RDS MySQL | db.t3.small, 100 GiB gp3, **Multi-AZ** | ~$70 |
| EFS | Regional, 10 GB, lifecycle policy | ~$3 |
| ALB | 1 ALB, ~10 LCU | ~$25 |
| NAT Gateway | 2 AZs (private subnets) | ~$65 |
| ACM | Free (public cert) | $0 |
| Secrets Manager | 2 secrets | ~$0.80 |
| CloudWatch | Enhanced monitoring + alarms | ~$10 |
| **Total (approx.)** | | **~$224/month** |

---

## Cost Optimization Notes

### Things disabled in this lab to reduce cost
- RDS automated backups → OFF (saves storage costs)
- RDS Multi-AZ → OFF (saves ~50% on RDS)
- EFS automatic backups → OFF
- EFS lifecycle management → OFF
- RDS encryption → OFF
- NAT Gateway → not used (tasks in public subnets instead)

### In production you would enable
- **RDS Multi-AZ** — critical for HA, doubles RDS cost but ensures failover
- **RDS automated backups** — retention 7+ days
- **EFS lifecycle policy** — move infrequent files to cheaper IA storage tier
- **NAT Gateway** — tasks should be in private subnets; this is the biggest cost add (~$32/AZ/month)
- **EFS encryption at rest** — no extra cost, just a configuration flag

### Auto Scaling saves money
The CPU-based auto scaling policy (target 50%) means tasks scale down automatically during off-hours. At minimum 2 tasks, the base cost stays low. The max of 8 tasks only runs during traffic spikes.

---

## Fargate Pricing Formula

```
vCPU cost = num_tasks × vCPU × hours × $0.04048/vCPU-hour
Memory cost = num_tasks × GB × hours × $0.004445/GB-hour
```

Example for 2 tasks, 720 hours/month:
```
vCPU: 2 × 0.25 × 720 × $0.04048 = $14.57
Memory: 2 × 0.5 × 720 × $0.004445 = $3.20
Total: ~$17.77/month for compute
```
