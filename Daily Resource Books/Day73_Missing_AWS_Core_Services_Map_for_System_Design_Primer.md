# Day 73 — Missing Primer: The AWS Services Map You Need for System-Design Answers

**Do this first on Day 73 (≈ 60 min), then continue with Day 73's regular plan.**
**Needed by:** Day 74 (AWS networking/EC2/RDS diagram), Day 75 (IAM), Days 120–133 (every HLD answer names cloud building blocks), Day 135 (deployment).
**Why it is missing:** the Day 74 micro-primer defines only EC2 and RDS. DynamoDB, CloudFront, load balancers, Route 53 and CloudWatch appear nowhere in the books (the curriculum map says the primer also covers VPC/Terraform; it does not).
**Already in place:** the AWS EC2/RDS micro-primer (Day 74), caching/CDN ideas (Day 123), load balancing (Day 72). You do **not** need an AWS account — this plan uses AWS at diagram level.

---

## 1. Map each building block to a service
| HLD building block | AWS service | One fact worth knowing |
|---|---|---|
| virtual servers | **EC2** (+ **Auto Scaling Groups**) | scale out behind a load balancer; AZ = failure domain |
| load balancer | **ALB** (L7, HTTP routing) / **NLB** (L4, TCP, static IP) | ALB for web/microservices, NLB for raw TCP/ultra-low latency |
| DNS | **Route 53** | routing policies: weighted, latency-based, failover, geolocation |
| CDN | **CloudFront** | caches at edge locations; origins S3/ALB; invalidations; signed URLs |
| object storage | **S3** | durable, strongly consistent reads-after-write (since 2020); storage classes; presigned URLs |
| relational DB | **RDS / Aurora** | **Multi-AZ** = standby for failover; **read replicas** = async copies for read scaling — different things |
| key-value / NoSQL | **DynamoDB** | partition key (hash) + optional sort key; GSIs; eventually consistent reads by default, strongly consistent optional; on-demand or provisioned; TTL; Streams; beware hot partitions |
| cache | **ElastiCache** (Redis/Memcached) | managed Redis |
| queue | **SQS** | standard (at-least-once, best-effort order) vs FIFO (ordered per message group); visibility timeout; dead-letter queue |
| pub/sub fan-out | **SNS** | topic → SQS queues / Lambda / HTTP |
| streaming | **Kinesis**, **MSK** (managed Kafka) | MSK = the Kafka you already know |
| serverless compute | **Lambda** | event-driven, pay per invocation; cold starts; execution-time limit (15 min) |
| containers / Kubernetes | **ECS**, **EKS**, **ECR** | EKS = managed Kubernetes control plane |
| API front door | **API Gateway** | managed routing/auth/throttling |
| observability | **CloudWatch** | metrics, logs, alarms, dashboards |
| identity & secrets | **IAM**, **Secrets Manager**, **KMS** | roles over long-lived keys; least privilege |

## 2. Networking in one picture
A **VPC** is your private network, split into **subnets** per Availability Zone. **Public** subnets route to an **Internet Gateway**; **private** subnets reach the internet only through a **NAT gateway**. **Security groups** are *stateful* instance-level firewalls; **NACLs** are *stateless* subnet-level. Put load balancers in public subnets and app/DB tiers in private ones.

## 3. Infrastructure as code (vocabulary)
**Terraform** (declarative, cloud-agnostic; `plan` shows changes, `apply` makes them, a *state* file tracks reality), **CloudFormation** (AWS-native), **Pulumi** (real languages). Helm (Day 135) packages Kubernetes manifests — a different layer.

## 4. Exercise (≈ 25 min, paper)
1. Redraw the platform on AWS: Route 53 → CloudFront → ALB → EKS/ECS services → RDS (per service) + ElastiCache + MSK; mark public vs private subnets and the AZs.
2. For each of Day 120–133's designs you have done, replace generic boxes with AWS names.
3. Answer in two lines each: Multi-AZ vs read replica; SQS vs SNS vs Kinesis; ALB vs NLB.

## Checklist
- [ ] Can name the AWS service for each block in §1 and one distinguishing fact.
- [ ] Can explain Multi-AZ vs read replica, and public vs private subnet.
- [ ] Can say what Terraform does vs Helm. Continue with Day 73's regular plan.
