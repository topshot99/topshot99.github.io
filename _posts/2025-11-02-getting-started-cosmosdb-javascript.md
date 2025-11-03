---
layout: post
title: "Getting Started with Azure Cosmos DB JavaScript SDK"
date: 2025-11-02 10:00:00 +0000
categories: [azure, cosmosdb, javascript]
tags: [cosmos-db, javascript, sdk, tutorial, getting-started]
author: TopShot99
excerpt: "A comprehensive guide to getting started with Azure Cosmos DB using the JavaScript SDK, covering setup, basic operations, and best practices."
---

# Getting Started with Azure Cosmos DB JavaScript SDK

Azure Cosmos DB is Microsoft's globally distributed, multi-model database service. In this post, I'll walk you through getting started with the JavaScript SDK and share some insights from my hands-on experience.

## Installation and Setup

First, install the Azure Cosmos DB SDK:

```bash
npm install @azure/cosmos
```

## Basic Connection Setup

```typescript
import { CosmosClient, ConnectionMode } from "@azure/cosmos";
import dotenv from "dotenv";

dotenv.config();

const client = new CosmosClient({
    endpoint: process.env.COSMOS_ENDPOINT,
    key: process.env.COSMOS_KEY,
    connectionPolicy: {
        connectionMode: ConnectionMode.Gateway,
        enableEndpointDiscovery: false,
    }
});
```

## Working with the Emulator

For development, the Cosmos DB Emulator is invaluable:

```typescript
const endpoint = 'https://127.0.0.1:8081/';
const key = 'C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==';

const client = new CosmosClient({
    endpoint,
    key,
    connectionPolicy: {
        connectionMode: ConnectionMode.Gateway,
        enableEndpointDiscovery: false,
    }
});
```

## Creating Database and Container

```typescript
// Create database
const { database } = await client.databases.createIfNotExists({
    id: 'MyDatabase'
});

// Create container with partition key
const { container } = await database.containers.createIfNotExists({
    id: 'MyContainer',
    partitionKey: { paths: ['/partitionKey'] }
});
```

## Basic CRUD Operations

### Create
```typescript
const newItem = {
    id: 'unique-id',
    partitionKey: 'partition-value',
    name: 'Sample Item',
    description: 'This is a sample item'
};

const { resource: createdItem } = await container.items.create(newItem);
```

### Read
```typescript
const { resource: item } = await container
    .item('unique-id', 'partition-value')
    .read();
```

### Update
```typescript
item.description = 'Updated description';
const { resource: updatedItem } = await container
    .item('unique-id', 'partition-value')
    .replace(item);
```

### Delete
```typescript
await container.item('unique-id', 'partition-value').delete();
```

## Querying Data

```typescript
const querySpec = {
    query: "SELECT * FROM c WHERE c.partitionKey = @partitionKey",
    parameters: [
        {
            name: "@partitionKey",
            value: "partition-value"
        }
    ]
};

const { resources: items } = await container.items
    .query(querySpec)
    .fetchAll();
```

## Key Insights and Best Practices

1. **Partition Key Selection**: Choose partition keys that distribute data evenly
2. **Connection Mode**: Gateway mode is simpler for development, Direct mode for production
3. **Error Handling**: Always wrap operations in try-catch blocks
4. **Resource Management**: Properly dispose of clients when done

## Common Gotchas

- **Partition Key Requirements**: Most operations require specifying the partition key
- **Case Sensitivity**: Property names in queries are case-sensitive
- **Rate Limiting**: Be prepared to handle 429 responses
- **Connection Management**: Reuse the client instance across operations

## What's Next?

In upcoming posts, I'll dive deeper into:
- Advanced querying techniques
- Performance optimization strategies
- Change feed processing
- Bulk operations and transactions

Stay tuned for more Cosmos DB insights!

---

*Have questions about Cosmos DB? Feel free to reach out or check out my other posts on database optimization!*