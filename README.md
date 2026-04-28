# Kanboard on AWS ECS — Highly Available Deployment

A step-by-step project deploying the open-source Kanban application [Kanboard](https://kanboard.org/) on AWS ECS Fargate with full high availability, persistent storage, load balancing, auto scaling, and secrets management.

> **Context:** This project was completed as part of module UD08 — *Containerized Service Orchestration* at IES Juan de Garay (Valencia), as part of the *Cloud Systems Administration* specialization course.

---

## Architecture

![Architecture diagram](images/01-architecture.png)

Check [docs/architecture.md](docs/architecture.md) for the infrastructure explanation.

---

## Repo Structure

```
kanboard-ecs/
├── README.md
├── docs/
│   ├── architecture.md        # Architecture description
│   ├── step-by-step.md        # Full deployment guide
│   ├── troubleshooting.md     # Issues encountered and solutions
│   └── cost-estimate.md       # AWS cost breakdown
├── iac/
│   ├── 01-network.yaml        # VPC, subnets, security groups
│   ├── 02-storage.yaml        # EFS + RDS
│   ├── 03-ecs-cluster.yaml    # ECS cluster + task definitions
│   └── 04-services.yaml       # ECS services + ALB + auto scaling
└── images/                    # Project screenshots
```

---

## Quick Start (using CloudFormation)

```bash
# 1. Deploy network layer
aws cloudformation deploy \
  --template-file cloudformation/01-network.yaml \
  --stack-name kanboard-network \
  --capabilities CAPABILITY_IAM

# 2. Deploy storage (EFS + RDS) — takes ~5 min for RDS
aws cloudformation deploy \
  --template-file cloudformation/02-storage.yaml \
  --stack-name kanboard-storage \
  --parameter-overrides DBPassword=yourpassword

# 3. Deploy ECS cluster and task definitions
aws cloudformation deploy \
  --template-file cloudformation/03-ecs-cluster.yaml \
  --stack-name kanboard-ecs \
  --capabilities CAPABILITY_IAM

# 4. Deploy services with ALB and auto scaling
aws cloudformation deploy \
  --template-file cloudformation/04-services.yaml \
  --stack-name kanboard-services \
  --capabilities CAPABILITY_IAM
```

---

## Prerequisites

- AWS Academy lab account or AWS account with sufficient permissions
- EC2 instance with Ubuntu (for EFS mount tests and siege stress testing)

---

## Stress Test

Once deployed, you can verify auto scaling works with:

```bash
sudo apt install siege -y
siege -c 50 -t 600S -delay 0.1 http://<ALB-DNS-NAME>
```

Watch ECS scale from 2 → up to 8 tasks as CPU exceeds 50%, then scale back down after the test ends.

---

## Docs

- [Step-by-step deployment guide](docs/step-by-step.md)
- [Troubleshooting](docs/troubleshooting.md)
- [Cost estimate](docs/cost-estimate.md)
- [Architecture](docs/architecture.md)
