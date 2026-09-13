# Micro-Primer: AWS EC2 / RDS Terminology — Needed Before Day 74

**First needed:** Day 74's AWS deployment diagram, which places "EC2 instances in a public subnet" and "RDS in a private subnet" without either term being defined anywhere first.

**EC2 (Elastic Compute Cloud)** is AWS's core "rent a virtual machine" service. An EC2 instance is a virtual server you can install anything on and run continuously — conceptually similar to a Kubernetes Node (Kubernetes Fundamentals primer, before Day 96), except rented directly from AWS rather than being a machine inside your own cluster.

**RDS (Relational Database Service)** is a *managed* relational database — AWS runs and maintains the actual PostgreSQL (or MySQL, etc.) server for you: backups, patching, failover, all handled without you operating the database server yourself, the way you've been doing locally since Day 36.

**Deferred:** IAM Roles/Users and S3 vs. EBS get their own full, proper treatment the very next day, Day 75 — this micro-primer covers only the two terms Day 74 itself needs to make sense of its own diagramming task.

### Checklist
- [ ] Can state what EC2 and RDS each are, in one sentence each.
- [ ] Ready for Day 74.
