---
layout: post
title: "Running Open-Source DocumentDB Locally with Docker"
date: 2026-03-13 10:00:00 +0000
categories: [documentdb, docker, database]
tags: [documentdb, docker, mongodb, postgresql, open-source, tutorial, getting-started]
author: TopShot99
excerpt: "A hands-on guide to running the open-source DocumentDB (Microsoft's MongoDB-compatible database, now under the Linux Foundation) locally using Docker, complete with CRUD operations and monitoring setup."
---

# Running Open-Source DocumentDB Locally with Docker

Microsoft recently open-sourced **DocumentDB** — a MongoDB-compatible document database built on top of PostgreSQL — and donated it to the **Linux Foundation**. This is the same engine that powers parts of Azure Cosmos DB's MongoDB API. In this post, I'll walk you through setting it up locally with Docker and running real database operations against it.

## What is DocumentDB?

DocumentDB is a document database that provides:

- ✅ **MongoDB wire protocol compatibility** — use any MongoDB driver (pymongo, mongosh, Compass, etc.)
- ✅ **PostgreSQL as the storage backend** — leveraging the battle-tested reliability of Postgres
- ✅ A Rust-based **gateway process** (`documentdb_gateway`) that translates MongoDB protocol to PostgreSQL
- ✅ **MIT licensed** and governed by the Linux Foundation

It reports itself as MongoDB server version **7.0.0**, so existing MongoDB tooling works out of the box.

> 🔗 GitHub: [https://github.com/documentdb/documentdb](https://github.com/documentdb/documentdb)

## Prerequisites

Before we start, make sure you have:

- **Docker Desktop** installed and running (I used v29.2.0)
- **WSL2** enabled (required for Docker on Windows)
- **Python 3.x** with pip (for our demo script)

## Step 1: Pull and Run the DocumentDB Container

```bash
# Pull the official DocumentDB Local image
docker pull ghcr.io/microsoft/documentdb/documentdb-local:latest

# Run the container
docker run -dt --name documentdb-local \
  -p 10260:10260 \
  -e USERNAME=docdbuser \
  -e PASSWORD='DocDB@Learn2025' \
  ghcr.io/microsoft/documentdb/documentdb-local:latest
```

This starts DocumentDB on port **10260** with the credentials we specified.

## Step 2: Install pymongo

```bash
pip install pymongo
```

## Step 3: Connect and Run CRUD Operations

Here's the key gotcha I ran into — the DocumentDB gateway **always expects TLS connections**. Without the `tls=true` parameter, you'll get a cryptic `SSLError: [SSL: WRONG_VERSION_NUMBER]` error because the gateway receives plain TCP when it expects TLS.

The fix is to include `tls=true&tlsAllowInvalidCertificates=true` in your connection string (the container uses a self-signed certificate).

Here's the full demo script I wrote covering 7 different operations:

```python
"""
DocumentDB Local - Learning Demo
=================================
This script demonstrates basic MongoDB-compatible operations
against the open-source DocumentDB running in Docker.
"""

from pymongo import MongoClient
from urllib.parse import quote_plus

# --- Connect ---
password = quote_plus("DocDB@Learn2025")  # URL-encode the @ symbol
uri = f"mongodb://docdbuser:{password}@localhost:10260/?directConnection=true&tls=true&tlsAllowInvalidCertificates=true"
client = MongoClient(uri)

print("=" * 50)
print("Connected to DocumentDB Local!")
info = client.server_info()
print(f"Server version: {info.get('version', 'N/A')}")
print("=" * 50)

db = client["learningDB"]
col = db["users"]
col.drop()  # start fresh

# --- 1. INSERT ---
docs = [
    {"name": "Alice", "age": 30, "skills": ["Python", "MongoDB"], "city": "Seattle"},
    {"name": "Bob", "age": 25, "skills": ["JavaScript", "React"], "city": "Portland"},
    {"name": "Charlie", "age": 35, "skills": ["Python", "Java", "Docker"], "city": "Seattle"},
    {"name": "Diana", "age": 28, "skills": ["Go", "Kubernetes"], "city": "Austin"},
]
result = col.insert_many(docs)
print(f"\n1. INSERT: Added {len(result.inserted_ids)} documents")

# --- 2. FIND ---
print("\n2. FIND: Users in Seattle:")
for doc in col.find({"city": "Seattle"}):
    print(f"   - {doc['name']}, age {doc['age']}, skills: {doc['skills']}")

print("\n   Users with Python skill:")
for doc in col.find({"skills": "Python"}):
    print(f"   - {doc['name']}")

# --- 3. UPDATE ---
col.update_one({"name": "Bob"}, {"$set": {"age": 26, "city": "San Francisco"}})
bob = col.find_one({"name": "Bob"})
print(f"\n3. UPDATE: Bob -> age={bob['age']}, city={bob['city']}")

# --- 4. AGGREGATION ---
print("\n4. AGGREGATION: Average age by city:")
pipeline = [
    {"$group": {"_id": "$city", "avg_age": {"$avg": "$age"}, "count": {"$sum": 1}}},
    {"$sort": {"avg_age": -1}},
]
for doc in col.aggregate(pipeline):
    print(f"   {doc['_id']}: avg_age={doc['avg_age']:.1f}, count={doc['count']}")

# --- 5. INDEXING ---
col.create_index("name", unique=True)
col.create_index([("city", 1), ("age", -1)])
print("\n5. INDEXES created:")
for name, info in col.index_information().items():
    print(f"   {name}: {info['key']}")

# --- 6. DELETE ---
result = col.delete_one({"name": "Charlie"})
print(f"\n6. DELETE: Removed {result.deleted_count} document (Charlie)")
remaining = [d["name"] for d in col.find({}, {"_id": 0, "name": 1})]
print(f"   Remaining users: {remaining}")

# --- 7. COUNT & EXISTS ---
print(f"\n7. METADATA:")
print(f"   Document count: {col.count_documents({})}")
print(f"   Databases: {client.list_database_names()}")
print(f"   Collections in learningDB: {db.list_collection_names()}")

print("\n" + "=" * 50)
print("All operations successful! DocumentDB is working.")
print("=" * 50)

client.close()
```

All 7 operations completed successfully ✅ — INSERT, FIND, UPDATE, AGGREGATION, INDEXING, DELETE, and METADATA queries all worked just like regular MongoDB.

## Step 4: Monitoring with MongoDB Compass

Since DocumentDB speaks the MongoDB wire protocol, we can use **MongoDB Compass** for a visual dashboard.

```bash
# Install via winget
winget install --id MongoDB.Compass.Community
```

### Compass Connection — TLS Gotcha

When connecting Compass to DocumentDB, there's a quirk: putting `tlsAllowInvalidCertificates=true` directly in the URI causes an error:

> `rejectUnauthorized must be either "true" or "false"`

The fix is to use a **simplified connection string** and configure TLS through the Compass UI:

**Connection String:**
```
mongodb://docdbuser:DocDB%40Learn2025@localhost:10260/?directConnection=true
```

**Then in Compass UI:**
1. Click **Advanced Connection Options**
2. Go to the **TLS/SSL** tab
3. Set **SSL** to **ON**
4. Enable **Allow Invalid Certificates**
5. Enable **Allow Invalid Hostnames**

Once connected, you can browse your `learningDB` database, inspect the `users` collection, view indexes, and monitor performance metrics.

## Gotchas & Tips

| Issue | Solution |
|-------|----------|
| `SSLError: WRONG_VERSION_NUMBER` | Add `tls=true&tlsAllowInvalidCertificates=true` to your connection URI |
| `@` in password | URL-encode it as `%40` |
| PowerShell `$` interpolation | MongoDB operators like `$set` get mangled — write code to `.py` files instead of inline |
| Compass TLS error | Use simplified URI + configure TLS in Compass UI (see above) |

---

## What's Next? — Diving into DocumentDB Internals

Now that we have DocumentDB running locally and have confirmed basic CRUD operations work, it's time to go deeper. In upcoming posts, I plan to explore the **inner workings of DocumentDB** and understand what makes it tick under the hood. Here are some of the features and internals I'll be looking into:

- 🔍 **How MongoDB Wire Protocol Compatibility Works** — How does the Rust-based `documentdb_gateway` translate MongoDB protocol into PostgreSQL queries?
- 🐘 **PostgreSQL as the Storage Engine** — What does the DocumentDB PostgreSQL extension actually do? How are documents stored in relational tables?
- 🔎 **Vector Search with HNSW Indexes** — DocumentDB supports vector indexes using the HNSW algorithm — great for AI/embedding workloads
- ⏰ **TTL (Time-To-Live) Indexes** — Automatic document expiration, useful for session data, logs, and caches
- ✅ **Schema Validation** — Enforce document structure while still being "schemaless"
- 📊 **RUM Indexes** — Read-Update-Memory optimized indexes for different workload patterns
- 🔀 **Sharding** — How DocumentDB distributes data across shards
- 🔗 **Aggregation Pipeline** — Deep dive into the aggregation framework support and its PostgreSQL translation
- 🔑 **Unique and Compound Indexes** — How indexing strategies differ from vanilla MongoDB

Stay tuned — this is going to be a fun deep dive into one of the most interesting open-source database projects to come out recently! 🚀

---

*Happy learning! — TopShot99*
