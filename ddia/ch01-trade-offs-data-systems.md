# Ch01 — Trade-Offs in Data Systems Architecture

## Two Systems

There are two systems: **operational** and **analytical**.

- **Operational** — where transactions are created (reads & writes)
- **Analytical** — where data is used for analysis (reads)

We divide these into two separate systems based on their data read and write patterns.

**OLTP** — Online Transaction Processing
From a developer or end-user perspective, this involves reading and writing data to the database.

**OLAP** — Online Analytics Processing
Primarily used for analysis or dashboards. Systems have also been built to display real-time data dashboards to users, even when processing large amounts of data to surface real-time metrics.

A data warehouse is a database where data is extracted, transformed, and loaded from various source databases, allowing analysts to query it freely.
This separation ensures that analytical workloads do not disturb the actual OLTP databases, as heavy queries could degrade their performance and it would be difficult to query across several disparate databases.

<img src="assets/ddia_0101.png" width="500" />

For ML training, data must be transformed into vectors, which makes a traditional data warehouse unsuitable. Instead, raw data is stored in **data lakes**, where it can take any form — images, videos, genome sequences, etc. This is known as the **sushi principle**: "raw is better."

There are two categories: **systems of record** and **derived data systems**. A system of record is where user input is stored — typically normalized — and serves as the source of truth. Derived data systems, such as caches, are built on top of this data and must be kept in sync. The challenge lies in keeping derived data systems up to date, which is addressed through **data pipelines** for data integration.

**Cloud vs. Self-Hosted**
The core decision is what to build in-house versus what to outsource. For example, you could take MySQL and install it on your own hardware or on a virtual machine in the cloud (IaaS). Another option is taking an open-source database and running a customized version of it.

## Cloud Native System Architecture

| Category | Self-hosted Systems | Cloud Native Systems |
|---|---|---|
| Operational/OLTP | MySQL, PostgreSQL, MongoDB | AWS Aurora, Azure SQL DB Hyperscale, Google Cloud Spanner |
| Analytical/OLAP | Teradata, ClickHouse, Spark | Snowflake, Google BigQuery, Azure Synapse Analytics |

## Separation of Storage and Compute

Hard disks are mechanical devices (or even SSDs) that can fail at any time — due to wear, power surges, manufacturing defects, etc. When a disk fails, all data on it can be permanently lost. For critical systems (databases, servers, etc.), this is unacceptable.

### The Assumption of Durability

In traditional computing, we treat disk storage as durable — meaning:

- If you write a file and the program crashes, the file is still there
- If you turn off the computer and turn it back on, the data persists

This is in contrast to RAM (memory), which is volatile — it loses everything when power is cut. But "durable" doesn't mean "indestructible." Disks still fail, which is exactly the problem RAID solves.

**RAID** stands for Redundant Array of Independent Disks. The core idea is simple: instead of trusting a single disk, spread or copy your data across multiple physical disks so that if one fails, you don't lose anything.

> Think of it like making photocopies of an important document — if one copy burns, you still have the others.

### What "Operations" Traditionally Meant

Before cloud computing, running software meant owning physical machines. The operations team was responsible for keeping those machines healthy and the services running on them alive. That included concrete, hands-on tasks like:

- **Capacity planning** — if your disks are filling up, you physically add more before you run out
- **Provisioning** — setting up new machines when you need more compute power
- **Migration** — moving a service from one server to another (a painful, manual process)
- **Patching** — applying OS security updates without breaking anything

The mental model here is: the team manages machines, and services live on top of those machines.

### How Cloud Changed the Picture

Cloud providers introduced a key abstraction — they hide the machines behind an API. The passage uses storage as a concrete example:

| Traditional                       | Cloud                            |
| --------------------------------- | -------------------------------- |
| You buy a 2TB disk                | You just store data              |
| You plan capacity in advance      | You pay for what you use         |
| A disk fails → your service fails | Provider handles fault tolerance |

This is a profound shift. You're no longer thinking in terms of "I have 12 servers, server 7 is unhealthy" — you're thinking in terms of services and outcomes. The physical layer is someone else's problem.

### The DevOps/SRE Philosophy

With machines abstracted away, the operations role didn't disappear — it transformed. The passage lists five principles of this modern philosophy:

- **Automation over manual work** — instead of an engineer SSHing into a machine to fix something, you write a script or pipeline that handles it repeatably. One-off fixes are fragile and undocumented.
- **Ephemeral infrastructure** — rather than a server you carefully maintain for years (a "pet"), you spin up fresh VMs on demand and throw them away when done (treating them like "cattle"). This forces your configuration to be codified rather than tribal knowledge living on one machine.
- **Frequent deployments** — the ability to ship small updates often, rather than large risky releases infrequently. Smaller changes are easier to reason about and roll back.
- **Learning from incidents** — when something breaks, the team does a postmortem to understand why, not just what. The goal is systemic improvement, not blame.
- **Knowledge preservation** — systems should be documented and automated well enough that the organization doesn't lose critical know-how when an individual engineer leaves. The knowledge lives in code and docs, not in people's heads.

### Distributed Versus Single-Node Systems

- **Inherent distribution** — Multi-user apps are unavoidably distributed; devices communicate over a network.
- **Requests between cloud services** — Data living in one service but processed in another must travel over the network; microservices are inherently distributed.
- **Fault tolerance / high availability** — Spread across multiple machines so if one fails, another takes over.
- **Scalability** — Spread load across many machines when a single machine can't keep up.
- **Latency** — Place servers near users geographically to reduce round-trip time.
- **Elasticity** — Scale up during peak demand and down during idle periods; pay only for what you use.
- **Specialized hardware** — Different workloads (storage, analytics, ML) can run on hardware optimized for them (many disks, lots of RAM, GPUs, etc.).
- **Legal compliance** — Data residency laws in some countries require storing and processing user data within their borders.
- **Sustainability** — Schedule jobs where and when renewable energy is available to cut carbon emissions and cost.

## Microservices

The best way to distribute a system is to divide it into client and server — this is called **service-oriented architecture (SOA)**. This idea has evolved into **microservices**, where each team owns its own APIs and databases. The primary motivation is organizational: it solves the "people problem" by allowing large teams to deploy changes independently. The trade-off is operational complexity — maintaining, testing, and managing the infrastructure for each service can be challenging.

### Serverless / Function as a Service (FaaS)

Another approach to deploying services in which the management of the infrastructure is outsourced to a cloud vendor. You pay only for the time your application code is running, rather than having to provision resources in advance.
