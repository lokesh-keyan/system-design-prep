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