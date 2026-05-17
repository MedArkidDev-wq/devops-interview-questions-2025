# DevOps Interview Questions 2025

> Real questions from real interviews — with **full answers**, not just question lists. Updated monthly.

[![Stars](https://img.shields.io/github/stars/MedArkidDev-wq/devops-interview-questions-2025?style=social)](https://github.com/MedArkidDev-wq/devops-interview-questions-2025)
[![Last Updated](https://img.shields.io/github/last-commit/MedArkidDev-wq/devops-interview-questions-2025)]()

---

## Contents

- [Terraform / IaC Questions](#-terraform--iac-questions)
- [Kubernetes Questions](#-kubernetes-questions)
- [Docker Questions](#-docker-questions)
- [CI/CD Questions](#-cicd-questions)
- [Linux Questions](#-linux-questions)
- [AWS Cloud Questions](#-aws-cloud-questions)
- [Monitoring Questions](#-monitoring--observability-questions)
- [Scenario-Based Questions](#-scenario-based-questions)
- [Behavioral Questions](#-behavioral-questions)

---

## Terraform / IaC Questions

### Q: "What happens when you run `terraform apply`?"

**Answer:**
1. Terraform reads your `.tf` configuration files
2. Loads the current state from the backend (e.g., S3)
3. Calls the cloud provider API to get the actual current state
4. Computes the **diff** between desired state (your code) and actual state
5. Shows you the plan (what will be created/modified/destroyed)
6. After you confirm, applies changes via API calls
7. Updates the state file with the new reality

**Follow-up:** "What is the state file and why does it matter?"

The state file maps your Terraform code to real cloud resources. Without it, Terraform can't know what already exists and would try to recreate everything on every apply.

---

### Q: "What's the difference between `terraform plan` and `terraform apply`?"

**Answer:**
- `plan` = preview mode. Shows what WOULD happen. Creates nothing. Safe to run anytime.
- `apply` = execution mode. Actually creates/modifies/destroys resources.

**Best practice:** Always run `plan` first, review carefully, then `apply`. In CI/CD: run `plan` on Pull Request, `apply` on merge to main.

---

### Q: "How do you handle secrets in Terraform?"

**Answer:**
1. **Never** commit secrets to `.tf` files or version control
2. Use `terraform.tfvars` (added to `.gitignore`) for local development
3. Use environment variables: `TF_VAR_database_password`
4. Use a secrets manager (AWS Secrets Manager, HashiCorp Vault)
5. Mark variables as `sensitive = true` in variable definitions
6. Use remote state encryption (S3 bucket with KMS)

---

### Q: "What is Terraform state locking and why is it important?"

**Answer:**
State locking prevents two people (or CI pipelines) from running `terraform apply` at the same time. Without locking, both would read the same state, apply changes, and one would overwrite the other — resulting in orphaned or duplicate resources.

DynamoDB is commonly used with S3 backends for state locking.

---

## Kubernetes Questions

### Q: "A pod is in `CrashLoopBackOff` — how do you debug it?"

**Answer (step by step):**
```bash
# 1. See pod status
kubectl get pod my-pod -n production

# 2. Read current logs
kubectl logs my-pod -n production

# 3. Read PREVIOUS crash logs
kubectl logs my-pod -n production --previous

# 4. Get detailed events
kubectl describe pod my-pod -n production
# Look at the Events section at the bottom

# Common causes:
# - Application code throwing uncaught exception
# - Wrong Docker image tag
# - Missing environment variable
# - Insufficient resources (OOMKilled)
# - Failed health check (livenessProbe)
```

---

### Q: "What's the difference between a Deployment and a StatefulSet?"

**Answer:**

| Feature | Deployment | StatefulSet |
|---|---|---|
| Pod names | Random (pod-abc123) | Ordered (pod-0, pod-1) |
| Scaling | Any order | Sequential (0, 1, 2...) |
| Storage | Shared or ephemeral | Persistent per pod |
| Use case | Stateless apps (APIs) | Databases, Kafka, Redis |
| DNS | Generic | Predictable (pod-0.service) |

---

### Q: "Explain the difference between liveness and readiness probes."

**Answer:**
- **Liveness probe**: "Is this container alive?" If it fails, Kubernetes **kills and restarts** the container.
- **Readiness probe**: "Is this container ready to receive traffic?" If it fails, Kubernetes **removes it from the Service** (stops sending traffic) but doesn't restart it.

Example: A Java app that takes 30 seconds to start. Readiness probe prevents traffic during startup. Liveness probe detects if the JVM hangs later.

---

### Q: "What is a Kubernetes Service and what types exist?"

**Answer:**

| Type | Access | Use Case |
|---|---|---|
| **ClusterIP** | Internal only | Backend APIs, databases |
| **NodePort** | External via node IP:port | Development, testing |
| **LoadBalancer** | External via cloud LB | Production traffic |
| **ExternalName** | DNS CNAME | External service mapping |

---

## Docker Questions

### Q: "What's the difference between CMD and ENTRYPOINT?"

**Answer:**
- **CMD**: Default command that can be **overridden** when running the container
- **ENTRYPOINT**: Fixed command that **cannot be easily overridden**
- Best practice: Use ENTRYPOINT for the main process, CMD for default arguments

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
# docker run myimage         -> python app.py
# docker run myimage test.py -> python test.py
```

---

### Q: "How do you reduce Docker image size?"

**Answer:**
1. Use slim/alpine base images (`python:3.12-slim` not `python:3.12`)
2. Multi-stage builds (build in one stage, copy only artifacts to final)
3. Combine RUN commands to reduce layers
4. Use `.dockerignore` to exclude unnecessary files
5. Don't install unnecessary packages
6. Clean up package manager cache in the same RUN layer

---

## CI/CD Questions

### Q: "Design a CI/CD pipeline for a microservice."

**Answer:**
```
Push code
  |-> Lint + Format check
  |-> Unit tests
  |-> Build Docker image
  |-> Security scan (Trivy/Snyk)
  |-> Push to registry
  |-> Deploy to staging
  |-> Integration tests
  |-> Manual approval gate
  |-> Deploy to production
  |-> Smoke tests
  |-> Monitor for 15 min
  |-> Rollback if error rate > threshold
```

---

## Linux Questions

### Q: "A server is slow. How do you diagnose it?"

**Answer (systematic approach):**
```bash
# 1. CPU
top          # See which process is eating CPU
uptime       # Check load average (should be < number of cores)

# 2. Memory
free -h      # Available vs used memory
vmstat 1     # Watch swap activity (si/so should be 0)

# 3. Disk
df -h        # Disk usage
iostat -x 1  # Disk I/O (await > 10ms is slow)
iotop        # Which process is doing I/O

# 4. Network
ss -s        # Connection summary
iftop        # Bandwidth usage

# 5. Processes
ps aux --sort=-%cpu | head  # Top CPU consumers
ps aux --sort=-%mem | head  # Top memory consumers
```

---

## AWS Cloud Questions

### Q: "What is a VPC and why do you need one?"

**Answer:**
A VPC (Virtual Private Cloud) is your own isolated network in AWS. It's like having your own data center inside AWS.

**You need it because:**
- Isolates your resources from other AWS customers
- Controls who can reach your servers (security groups, NACLs)
- Defines IP ranges and subnets
- Separates public-facing servers from private databases

---

### Q: "What's the difference between Security Groups and NACLs?"

**Answer:**

| Feature | Security Group | NACL |
|---|---|---|
| Level | Instance/ENI | Subnet |
| Rules | Allow only | Allow + Deny |
| State | Stateful | Stateless |
| Default | Deny all inbound | Allow all |
| Evaluation | All rules checked | Rules checked in order |

---

## Monitoring / Observability Questions

### Q: "What metrics would you monitor for a web application?"

**Answer (The Four Golden Signals):**
1. **Latency**: How long requests take (p50, p95, p99)
2. **Traffic**: Requests per second
3. **Errors**: HTTP 5xx rate, failed health checks
4. **Saturation**: CPU usage, memory usage, disk I/O, connection pool usage

Plus infrastructure metrics: disk space, network bandwidth, container restarts.

---

## Scenario-Based Questions

### "Production is down at 2 AM. What do you do?"

**Framework:**
1. **Acknowledge** — Check monitoring dashboard to understand scope
2. **Communicate** — Alert on-call team and stakeholders
3. **Assess** — Is it one service, one region, or everything?
4. **Identify** — Check recent deployments (anything changed in last 2 hours?)
5. **Mitigate** — Rollback if bad deploy. Scale up if load issue.
6. **Verify** — Confirm metrics return to normal
7. **Document** — Write post-mortem within 24 hours

**What they're testing:** Do you panic, or do you have a process?

---

### "Your AWS bill jumped 40% this month. How do you investigate?"

**Answer:**
1. Check **AWS Cost Explorer** — filter by service, sort by cost change
2. Look for **NAT Gateway charges** (most common surprise — $0.045/GB)
3. Check for **unused EC2 instances** or oversized instance types
4. Review **S3 storage** — old logs accumulating?
5. Check **data transfer costs** — cross-region or cross-AZ
6. Look for **forgotten resources** — test environments still running
7. Set up **AWS Budgets** and billing alerts to prevent it next time

---

## Behavioral Questions

### "Tell me about a time you broke production."

**Framework (STAR method):**
- **Situation**: Deployed a config change on a Friday afternoon
- **Task**: Needed to update database connection strings for migration
- **Action**: Pushed without adequate testing. Monitoring caught 5xx spike in 2 minutes. Rolled back within 5 minutes. Total impact: 3 minutes of degraded service.
- **Result**: No data loss. Wrote post-mortem. Implemented mandatory staging deployment step. Added pre-merge integration tests.

**What they're looking for:** Honesty, process, and learning from mistakes.

---

## Contributing

Did you get asked a question not listed here? Share it!
Open a PR with the question and your answer.

---

**If this helped your interview prep — please star this repo!**

## Author

**Mohamed Arkid** — DevOps Engineer and Cloud Consultant

- [moarkid.com](https://moarkid.com)
- [LinkedIn](https://www.linkedin.com/in/mohamed-arkid)
