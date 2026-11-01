# Day 73 — Missing Primer: AWS EC2 and RDS — the Two Terms the Next AWS Files Use Before Defining

**Do this first on Day 73 (≈ 10 min — *before* the Day 73 AWS services-map primer), then continue with Day 73's regular plan.**
**Needed by:** Day 73's own AWS services-map primer (its table lists EC2 and RDS in one line each), Day 74's AWS deployment diagram ("EC2 instances in a public subnet", "RDS in a private subnet" — no book defines either term), Day 75 (IAM), Days 120–133 (HLD answers name cloud building blocks).
**Why it is missing / why it moved:** this file replaces `Day74_Missing_AWS_EC2_RDS_MicroPrimer.md`. The Day 73 services-map primer lists EC2 and RDS as "already in place" from the Day 74 file — **a day too late**. The content is unchanged in substance; the day is corrected, the header follows the standard format, and a forward reference to Kubernetes (taught on Days 95–99) is removed.
**Already in place:** the local PostgreSQL you have run since Day 36, load balancers and L4/L7 (Day 72), Docker (Day 43). You do **not** need an AWS account — this plan uses AWS at diagram level.

---

## 1. EC2 — Elastic Compute Cloud
AWS's "rent a virtual machine" service. An **EC2 instance** is a virtual server you can install anything on and run continuously; you choose its size (CPU / memory), operating system and the network it sits in, and you pay for the time it runs. Think of it as a computer in someone else's data centre that you control over the network. Everything you have run on your Mac — a Spring Boot jar, a Docker container — could run on an EC2 instance instead. *(When you reach Kubernetes on Days 95–99 you will see that an EC2 instance is usually the "node" a cluster runs on.)*

## 2. RDS — Relational Database Service
A **managed** relational database. AWS runs the actual PostgreSQL (or MySQL, etc.) server for you: backups, patching, and failover to a standby are handled by AWS, so you do not operate the database server yourself the way you have locally since Day 36. You still design the schema, write the queries and choose the indexes (Day 43's file).

## 3. Why the diagram places them differently
In Day 74's diagram the application servers (EC2) sit in a **public subnet** — reachable from the internet through a load balancer — while the database (RDS) sits in a **private subnet** with no direct internet access, reachable only from the application. That split is the point of the exercise: the database should never be exposed to the internet.

## 4. Deferred
IAM (users, roles, policies) and S3 vs EBS get their own treatment on Day 75; DynamoDB, CloudFront, Route 53 and the rest of the map are in Day 73's services-map primer, which you do next.

## Checklist
- [ ] Can say in one sentence each what EC2 and RDS are.
- [ ] Can say why the app tier is public-facing and the database tier is private.
- [ ] Continue with Day 73's AWS services-map primer, then Day 73's regular plan.
