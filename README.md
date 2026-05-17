# DevOps Interview Questions 2026

Questions I've been asked in real interviews, with the answers that actually got me past the technical rounds. Not a list of 500 questions with one-sentence answers. Each answer here is what you'd actually say to an interviewer.

I keep adding to this as I go through more interviews. If you got asked something good, open a PR.

## Terraform

**"What happens when you run terraform apply?"**

Terraform reads your .tf files, loads the current state from wherever you store it (usually S3), then calls the cloud provider API to see what actually exists right now. It computes the diff between what you want and what's there, shows you the plan, and after you confirm, makes the API calls to create/modify/destroy resources. Then it updates the state file.

The state file is important because it's how Terraform knows what it already created. Without it, Terraform would try to create everything from scratch every time.

**"How do you handle secrets in Terraform?"**

Never put secrets in .tf files. Use terraform.tfvars (which goes in .gitignore), environment variables like TF_VAR_database_password, or a secrets manager like AWS Secrets Manager or Vault. Mark sensitive variables with `sensitive = true` so they don't show up in plan output. Encrypt your remote state.

**"What is state locking?"**

It stops two people from running terraform apply at the same time. Without it, both would read the same state, make changes, and one would overwrite the other. You end up with orphaned resources nobody knows about. DynamoDB is the standard choice when using S3 for state.

## Kubernetes

**"A pod is in CrashLoopBackOff. How do you debug it?"**

```bash
kubectl get pod my-pod -n production          # See the status
kubectl logs my-pod -n production             # Current logs
kubectl logs my-pod -n production --previous  # Logs from the LAST crash
kubectl describe pod my-pod -n production     # Events section at the bottom
```

Usually it's one of these: bad image tag, missing env var, OOMKilled (needs more memory), or the liveness probe is failing.

**"Deployment vs StatefulSet?"**

Deployments are for stateless stuff like APIs. Pod names are random, they can scale in any order, and they share storage. StatefulSets are for databases and things like Kafka where each pod needs its own persistent volume and a predictable name (pod-0, pod-1). Scaling happens in order.

**"Liveness vs readiness probes?"**

Liveness: "Is this container alive?" If it fails, K8s kills and restarts the container. Readiness: "Is this container ready for traffic?" If it fails, K8s stops sending traffic to it but doesn't restart it. You need both. A Java app that takes 30 seconds to start needs a readiness probe so it doesn't get traffic before it's ready, and a liveness probe to catch when the JVM hangs later.

**"What Service types are there?"**

ClusterIP is internal only (default). NodePort exposes on each node's IP at a static port. LoadBalancer creates a cloud load balancer. ExternalName is just a DNS alias to an external service.

## Docker

**"CMD vs ENTRYPOINT?"**

CMD is the default command that gets overridden when you pass arguments to docker run. ENTRYPOINT is the fixed command that always runs. Best practice is ENTRYPOINT for the process (like `python`) and CMD for default arguments (like `app.py`). That way `docker run myimage test.py` runs `python test.py`.

**"How do you make Docker images smaller?"**

Use slim or alpine base images. Multi-stage builds where you build in one stage and copy only the binary to the final image. Combine RUN commands so you don't create unnecessary layers. Use .dockerignore. Don't install stuff you don't need.

## CI/CD

**"Design a pipeline for a microservice."**

Push code, lint it, run unit tests, build the Docker image, scan it for vulnerabilities (Trivy or Snyk), push to registry, deploy to staging, run integration tests, wait for manual approval, deploy to production, run smoke tests, monitor for 15 minutes, auto-rollback if error rate spikes.

## Linux

**"A server is slow. Walk me through how you'd diagnose it."**

```bash
top                              # What's eating CPU?
uptime                           # Load average vs core count
free -h                          # Memory usage, swap activity
df -h                            # Disk full?
iostat -x 1                      # Disk I/O (await > 10ms = slow disk)
ss -s                            # Connection count
ps aux --sort=-%cpu | head       # Top CPU hogs
ps aux --sort=-%mem | head       # Top memory hogs
```

Check CPU first, then memory, then disk, then network. 90% of the time it's one of those four.

## AWS

**"What is a VPC?"**

Your own isolated network inside AWS. Think of it as your own mini data center. You need it because it isolates your stuff from everyone else's, lets you control what traffic gets in and out, and lets you separate public-facing servers from private databases.

**"Security Groups vs NACLs?"**

Security Groups are per-instance and stateful (if traffic is allowed in, the response is automatically allowed out). NACLs are per-subnet and stateless (you have to explicitly allow both inbound and outbound). Security Groups can only allow, NACLs can allow and deny. Most people only use Security Groups.

## Monitoring

**"What metrics do you monitor for a web app?"**

The four golden signals: latency (how long requests take, track p95 and p99), traffic (requests per second), errors (5xx rate), and saturation (CPU, memory, disk, connection pool). Also watch disk space, container restarts, and network bandwidth.

## Scenario Questions

**"Production is down at 2 AM. What do you do?"**

Check the monitoring dashboard to understand the scope. Is it one service or everything? Alert the on-call team. Check if anyone deployed anything in the last 2 hours. If yes, rollback. If it's a load issue, scale up. Once it's fixed, verify metrics are back to normal. Write a post-mortem within 24 hours.

They're testing whether you have a process or if you just panic.

**"AWS bill jumped 40%. How do you investigate?"**

Cost Explorer, filter by service, sort by cost change. Check NAT Gateway charges first (that's the most common surprise, $0.045/GB adds up fast). Look for unused EC2 instances, oversized instance types, forgotten test environments, and S3 buckets full of old logs. Set up AWS Budgets with alerts so it doesn't happen again.

## Behavioral

**"Tell me about a time you broke production."**

Use the STAR method. Situation: what was the context. Task: what you were trying to do. Action: what happened (be honest). Result: how you fixed it, what you changed so it wouldn't happen again.

They want to see that you're honest, you have a process for dealing with failures, and you learn from mistakes. Everyone has broken production. If you say you haven't, they don't believe you.

## Contributing

Got asked something good? Open a PR with the question and how you answered it.

Star this if it helped your prep.

## About

Built by [Mohamed Arkid](https://moarkid.com). Updating this as I go through interviews myself.
