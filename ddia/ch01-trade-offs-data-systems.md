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