# AWS Certified Solutions Architect – Associate (SAA-C03) Prep Guide

## Exam Facts (verified current as of 2026)
- **Format:** 65 questions (50 scored + 15 unscored, unlabeled), multiple choice + multiple-response
- **Duration:** 130 minutes | **Passing score:** 720/1000 | **Cost:** $150
- **Multiple-response questions:** you must select the *exact* number of correct answers stated — no partial credit
- **Domain weights** (this drives how you should split study time):

| Domain | Weight | Focus |
|---|---|---|
| 1. Design Secure Architectures | **30%** | IAM, VPC security, encryption, shared responsibility model |
| 2. Design Resilient Architectures | 26% | Multi-AZ, DR strategies, decoupling, backup |
| 3. Design High-Performing Architectures | 24% | Compute/storage/DB selection, caching, scaling |
| 4. Design Cost-Optimized Architectures | 20% | Pricing models, right-sizing, storage tiers |

**Security is the single biggest domain — most candidates under-study it and it's the most common reason for failing.** Weight your study time accordingly (roughly 35% security, 25% resilience, 25% performance, 15% cost, adjusted as you find your weak spots).

---

## 24-Day Study Plan

| Day | Topic | What to actually learn |
|---|---|---|
| 1 | IAM Fundamentals | Users/Groups/Roles/Policies, policy evaluation logic (explicit deny > allow), cross-account roles |
| 2 | VPC Basics | Subnets (public/private), route tables, Internet Gateway, NAT Gateway vs NAT Instance |
| 3 | VPC Advanced | Security Groups vs NACLs, VPC Peering, Transit Gateway, VPC Endpoints (Gateway vs Interface) |
| 4 | EC2 Fundamentals | Instance types/families, purchasing options (On-Demand, Reserved, Spot, Savings Plans), placement groups |
| 5 | EC2 Advanced + Auto Scaling | AMIs, EBS-backed vs instance-store, ASG scaling policies, launch templates |
| 6 | Elastic Load Balancing | ALB vs NLB vs GWLB, target groups, health checks, cross-zone LB |
| 7 | **Review + light practice quiz (Days 1-6)** | Redo weak-area notes; don't learn new material |
| 8 | S3 Fundamentals | Storage classes, versioning, lifecycle policies, S3 Object Lock, consistency model |
| 9 | Other Storage | EBS types (gp3/io2/st1/sc1), EFS, FSx (Windows/Lustre/NetApp), Storage Gateway |
| 10 | RDS & Aurora | Multi-AZ vs Read Replicas, Aurora Global Database, backup/restore, parameter groups |
| 11 | DynamoDB | Partition keys, GSI/LSI, on-demand vs provisioned capacity, DAX caching |
| 12 | ElastiCache + Redshift | Redis vs Memcached use cases; Redshift for OLAP/data warehousing |
| 13 | Databases wrap-up | Neptune (graph), Timestream (time-series), DMS for migration |
| 14 | **Review + practice quiz (Days 8-13)** | Focus on DB selection decision tree |
| 15 | Lambda & Serverless | Cold starts, concurrency limits, event sources, Lambda + API Gateway pattern |
| 16 | Messaging & Integration | SQS (Standard vs FIFO), SNS, EventBridge, Step Functions, SES |
| 17 | Route 53 + CloudFront | Routing policies (weighted, latency, failover, geolocation), CloudFront + OAC, edge caching |
| 18 | Security Services | KMS (CMK vs AWS-managed), Secrets Manager vs Parameter Store, WAF, Shield, GuardDuty, Security Hub |
| 19 | Monitoring & Governance | CloudWatch (metrics/alarms/logs), CloudTrail, AWS Config, Organizations + SCPs |
| 20 | Resilience & DR | 4 DR strategies (Backup/Restore, Pilot Light, Warm Standby, Multi-Site Active/Active), RTO/RPO |
| 21 | Cost Optimization | Compute Savings Plans vs RIs, Spot best practices, S3 storage class analysis, Cost Explorer, Trusted Advisor |
| 22 | Migration + Well-Architected | Snow family, DataSync, MGN, the 6 pillars of Well-Architected Framework |
| 23 | **Full-length practice exam** (65 Q, timed 130 min) | Review every wrong answer — understand *why*, not just the right answer |
| 24 | **Final revision** | Go through the Memory Maps + Revision Tables below only; light second practice exam if time permits |

**Daily rhythm:** ~1.5-2 hrs/day — 45 min video/reading, 45 min hands-on in AWS Free Tier console, 15-30 min flashcard review of previous day.

---

## FREQUENTLY ASKED / COMMONLY TESTED QUESTIONS

**Q: What's the difference between a Security Group and a Network ACL?**
A: Security Groups are **stateful** (return traffic auto-allowed), operate at the **instance level**, and only support "allow" rules. NACLs are **stateless** (must explicitly allow both directions), operate at the **subnet level**, and support both "allow" and "deny" rules, evaluated in rule-number order.

**Q: When do you use Multi-AZ RDS vs Read Replicas?**
A: Multi-AZ = **high availability** (synchronous standby in another AZ, automatic failover, NOT used for read scaling — you can't query the standby). Read Replicas = **read scaling** (asynchronous copies, can be in the same or different region, you can query them directly, and you can promote one to standalone in a DR scenario).

**Q: SQS Standard vs FIFO — when does the difference actually matter?**
A: Standard = at-least-once delivery, best-effort ordering, nearly unlimited throughput. FIFO = exactly-once processing, strict ordering (within a message group), throughput capped (300 msg/sec without batching, 3000 with batching). Use FIFO only when order/duplicates would break correctness (e.g., financial transactions); default to Standard otherwise for cost/throughput.

**Q: S3 storage class — how do you pick?**
A: Standard (frequent access) → Standard-IA (infrequent, but need millisecond access, min 30-day storage) → One Zone-IA (same but non-critical, can be recreated) → Intelligent-Tiering (unknown/changing access patterns — auto-moves objects) → Glacier Instant Retrieval (archive, millisecond access) → Glacier Flexible Retrieval (archive, minutes-hours) → Glacier Deep Archive (cheapest, 12-hour retrieval, long-term compliance).

**Q: ALB vs NLB vs Gateway Load Balancer?**
A: ALB = Layer 7, HTTP/HTTPS, path/host-based routing, WebSocket support. NLB = Layer 4, ultra-high performance, static IP support, TCP/UDP — use when you need extreme throughput/low latency or a fixed IP. GWLB = for deploying third-party virtual appliances (firewalls, IDS/IPS) transparently in the traffic path.

**Q: On-Demand vs Reserved Instances vs Savings Plans vs Spot — when does each make sense?**
A: On-Demand = unpredictable/short-term workloads. Reserved Instances = steady-state, predictable usage, 1 or 3-year commitment for discount (up to 72%), less flexible (tied to instance family/region in Standard RIs). Savings Plans = similar discount but more flexible (commit to $/hr spend, not a specific instance type). Spot = fault-tolerant/flexible workloads (batch, CI/CD, stateless web tiers) — up to 90% discount but can be reclaimed with 2-min warning.

**Q: KMS vs CloudHSM — what's the real difference?**
A: KMS is a managed, multi-tenant service — AWS manages the underlying HSM hardware; simpler, integrates natively with most AWS services. CloudHSM gives you a **dedicated, single-tenant** hardware security module — needed when compliance mandates FIPS 140-2 Level 3 or you need full control over the HSM (you manage key material, no AWS access).

**Q: Secrets Manager vs Systems Manager Parameter Store?**
A: Secrets Manager: automatic rotation (native RDS/Redshift/DocumentDB integration), built-in for secrets, costs more. Parameter Store (Standard tier): free, no automatic rotation (must build your own with Lambda), fine for config values / non-rotating secrets. Choose Secrets Manager whenever rotation is a stated requirement.

**Q: What's RTO vs RPO, and how does that map to DR strategy choice?**
A: RTO (Recovery Time Objective) = how long you can be down. RPO (Recovery Point Objective) = how much data you can afford to lose. Backup & Restore = highest RTO/RPO, cheapest. Pilot Light = core infra always running minimally, scale up on failover. Warm Standby = scaled-down full replica running, quick scale-up. Multi-Site Active/Active = lowest RTO/RPO (near zero), most expensive — traffic actively served from multiple regions.

**Q: Why would you use a VPC endpoint instead of just going over the internet/NAT gateway?**
A: VPC endpoints let private subnet resources reach AWS services (S3, DynamoDB, etc.) **without traversing the public internet** — better security (traffic stays on the AWS network) and can reduce NAT Gateway data-processing costs. Gateway endpoints (free) support S3 and DynamoDB only; Interface endpoints (small hourly + data cost, use PrivateLink) support most other services.

**Q: DynamoDB partition key design — why does it matter so much?**
A: A poorly chosen partition key (e.g., low-cardinality, or a hot key like a fixed date) causes **hot partitions** — throttling on that partition even if overall table capacity is fine. Choose high-cardinality keys with evenly distributed access patterns; use composite keys (partition + sort key) when you need range queries within a partition.

**Q: When do you actually need Transit Gateway instead of VPC Peering?**
A: VPC Peering works fine for a small number of VPCs but doesn't scale — it's point-to-point (no transitive routing), so N VPCs need ~N²/2 peering connections. Transit Gateway acts as a central hub — one connection per VPC, supports transitive routing, and scales to hundreds of VPCs/VPNs across accounts and regions.

**Q: What does "stateless" mean for NACLs and why should you care on the exam?**
A: Because NACLs are stateless, if you allow inbound traffic on a port, you must **also** explicitly allow the corresponding outbound (ephemeral port range, typically 1024-65535) or return traffic gets blocked — a very common exam trick question.

**Q: CloudFront + S3 — why use Origin Access Control (OAC) instead of a public S3 bucket?**
A: OAC lets CloudFront access a **private** S3 bucket on the backend while still serving content publicly through CloudFront — so the S3 bucket itself is never directly publicly accessible, reducing attack surface, while still getting edge caching/performance benefits.

**Q: Placement Groups — Cluster vs Spread vs Partition, when to use each?**
A: Cluster = instances packed close together in a single AZ for lowest network latency (HPC workloads) — but higher correlated-failure risk. Spread = instances on distinct underlying hardware (max 7 per AZ) — for small numbers of critical instances needing isolation from each other. Partition = groups of instances isolated from other partitions' hardware — for large distributed systems (Hadoop, Cassandra, Kafka) needing rack-awareness.

---

## MEMORY MAPS (Decision Trees)

### 1. Storage Selection
```
Need to store data?
├─ Object storage (files, images, backups, static sites)?
│   └─ Amazon S3
│       ├─ Frequent access → Standard
│       ├─ Infrequent, need ms access → Standard-IA / One Zone-IA
│       ├─ Unknown/changing pattern → Intelligent-Tiering
│       └─ Archive/compliance → Glacier (Instant / Flexible / Deep Archive)
├─ Block storage attached to ONE EC2 instance?
│   └─ Amazon EBS (gp3 general purpose, io2 high-IOPS DB, st1/sc1 throughput/cold HDD)
├─ Shared file system, Linux, multiple EC2s?
│   └─ Amazon EFS
├─ Shared file system, Windows?
│   └─ FSx for Windows File Server
├─ High-performance computing file system?
│   └─ FSx for Lustre
└─ On-premises needs cloud-backed storage / hybrid?
    └─ Storage Gateway (File / Volume / Tape gateway)
```

### 2. Database Selection
```
What's the data access pattern?
├─ Relational, need SQL joins/ACID transactions?
│   ├─ Standard workload → Amazon RDS (MySQL/Postgres/MariaDB/Oracle/SQL Server)
│   └─ Need higher performance/scale, AWS-native → Aurora (MySQL/Postgres compatible)
├─ Key-value or document, massive scale, single-digit ms latency?
│   └─ DynamoDB (+ DAX for microsecond caching)
├─ In-memory cache / session store / leaderboard?
│   └─ ElastiCache (Redis = persistence/complex structures, Memcached = simple/multi-threaded)
├─ Data warehouse / OLAP / BI analytics on structured data?
│   └─ Redshift
├─ Highly connected data (social graphs, recommendation engines)?
│   └─ Neptune
└─ Time-series data (IoT sensors, metrics)?
    └─ Timestream
```

### 3. Compute Selection
```
How much infrastructure control do you need?
├─ Full OS control, long-running, custom software?
│   └─ EC2
├─ Containers, don't want to manage servers?
│   └─ ECS/EKS with Fargate (serverless containers)
├─ Containers, need full control over underlying nodes?
│   └─ ECS/EKS with EC2 launch type
├─ Short-lived, event-driven functions, no idle cost desired?
│   └─ Lambda (mind the ~15-min max execution + cold starts)
└─ Large-scale batch/parallel jobs?
    └─ AWS Batch
```

### 4. Load Balancer Selection
```
What layer / protocol?
├─ HTTP/HTTPS, need content-based routing (path/host), WebSockets?
│   └─ Application Load Balancer (ALB)
├─ TCP/UDP, need extreme performance, static IP, or PrivateLink?
│   └─ Network Load Balancer (NLB)
└─ Deploying 3rd-party virtual appliances (firewall/IDS) transparently?
    └─ Gateway Load Balancer (GWLB)
```

### 5. Disaster Recovery Strategy (by RTO/RPO, ascending cost)
```
How much downtime/data-loss can you tolerate?
├─ Hours of downtime OK, cheapest → Backup & Restore
├─ Some minutes-hours downtime OK → Pilot Light (core services always on, minimal)
├─ Minutes downtime OK, faster scale-up → Warm Standby (scaled-down full copy running)
└─ Near-zero downtime/data-loss, cost no object → Multi-Site Active/Active
```

### 6. Security / Encryption Selection
```
What do you need?
├─ Managed encryption keys, native AWS service integration?
│   └─ KMS (AWS-managed key = simplest; Customer-managed key = more control/audit)
├─ Dedicated single-tenant HSM, strict compliance (FIPS 140-2 L3)?
│   └─ CloudHSM
├─ Secrets needing automatic rotation (DB credentials)?
│   └─ Secrets Manager
├─ Simple config values / secrets, no rotation needed, cost-sensitive?
│   └─ Systems Manager Parameter Store
├─ Protect web apps from common exploits (SQLi, XSS)?
│   └─ WAF
├─ Protect against DDoS?
│   └─ Shield (Standard = free/automatic; Advanced = paid, larger attacks + cost protection)
├─ Continuous threat detection using ML on logs/traffic?
│   └─ GuardDuty
└─ Centralized security posture/compliance dashboard?
    └─ Security Hub (+ Config for resource compliance history)
```

### 7. Migration Tool Selection
```
What are you migrating?
├─ Ongoing file sync between on-prem and AWS (network-based)?
│   └─ DataSync
├─ One-time bulk data transfer, network too slow/expensive?
│   └─ Snow Family (Snowball/Snowball Edge = TBs-PBs, Snowmobile = exabyte scale)
├─ Database migration (homogeneous or heterogeneous engines)?
│   └─ Database Migration Service (DMS) (+ Schema Conversion Tool for heterogeneous)
└─ Lift-and-shift entire servers/VMs?
    └─ Application Migration Service (MGN)
```

### 8. Messaging / Integration Selection
```
What's the communication pattern?
├─ Decouple producer/consumer, need a durable queue?
│   └─ SQS (Standard = high throughput/best-effort order; FIFO = strict order/exactly-once)
├─ One message to many subscribers (fan-out)?
│   └─ SNS (often SNS → multiple SQS queues for fan-out + durability)
├─ Event-driven routing between AWS services / SaaS / custom apps?
│   └─ EventBridge
└─ Multi-step workflow orchestration with retries/branching?
    └─ Step Functions
```

---

## QUICK REVISION TABLE (last-day cram)

| Service | One-line "why" | Common trap |
|---|---|---|
| IAM | Least-privilege access control | Explicit DENY always wins, even over an ALLOW |
| S3 | Object storage | Not for OS boot volumes — that's EBS |
| EBS | Block storage, one instance at a time (except io2 Multi-Attach) | Data persists only if "Delete on Termination" is off |
| EFS | Shared file storage, multi-AZ, Linux | More expensive than EBS; not for Windows (use FSx) |
| RDS Multi-AZ | HA, NOT read scaling | Can't query the standby directly |
| Read Replica | Read scaling, NOT auto-failover (must promote manually) | Async replication = possible lag |
| DynamoDB | Serverless NoSQL, massive scale | Hot partition if key design is poor |
| ElastiCache | Sub-millisecond cache | Cache invalidation is your app's job, not automatic |
| Auto Scaling | Elastic capacity based on demand | Scaling policies need CloudWatch alarms behind them |
| ALB | Layer 7, smart routing | Doesn't support static IP directly (NLB does) |
| NLB | Layer 4, extreme performance | No content-based routing |
| Route 53 | DNS + routing policies | Health checks needed for failover routing to work |
| CloudFront | CDN, edge caching | Needs OAC for private S3 origins |
| KMS | Managed encryption keys | CMK you manage rotation frequency; AWS-managed keys auto-rotate yearly |
| Secrets Manager | Auto-rotating secrets | Costs more than Parameter Store — use only when rotation is required |
| WAF | App-layer firewall (SQLi/XSS rules) | Works with ALB/CloudFront/API Gateway, not raw EC2 |
| Shield Advanced | DDoS protection + cost protection | Only worth it for large/critical internet-facing apps |
| SQS Standard | High-throughput queue | At-least-once — consumers must be idempotent |
| SQS FIFO | Ordered, exactly-once | Throughput capped — don't default to this |
| Lambda | Event-driven compute, no server mgmt | 15-min max runtime; cold starts affect latency-sensitive apps |
| Step Functions | Workflow orchestration | Use over chaining Lambdas manually when workflow has branching/retries |
| Trusted Advisor | Best-practice checks (cost, security, performance) | Full checks need Business/Enterprise support plan |
| Well-Architected Framework | 6 pillars: Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability | Exam scenarios often map directly to one pillar — identify which one is being tested |

---

## Final week tips
- Take at least 2 full-length timed practice exams before test day — review every wrong answer for the *underlying concept*, not just memorize the right option.
- On the real exam, when two options both seem technically correct, pick the one that best matches the **stated constraint** in the question (cost, "least operational overhead," "most available," etc.) — SAA-C03 questions are scenario-based and the qualifier word is usually the deciding factor.
- Don't skip the Well-Architected Framework — many ambiguous questions resolve once you identify which of the 6 pillars the question is really testing.
