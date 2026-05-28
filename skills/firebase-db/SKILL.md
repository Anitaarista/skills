---
name: firebase-db
description: >
  Firebase Firestore database integration for this workspace. Use this skill whenever a project
  needs persistent database storage, real-time data, user management, or any server-side data
  operations — including when building web apps, APIs, dashboards, or any feature that requires
  reading/writing structured data. Also use when migrating from file-based storage (JSON, SQLite,
  localStorage) to Firebase. Automatically initializes Firebase Admin SDK with the project
  credentials and provides ready-to-use CRUD patterns, query builders, and migration utilities.
  Trigger on: "database", "Firebase", "Firestore", "data storage", "save data", "read data",
  "query data", "migrate database", "real-time database", "collections", "documents",
  "persistent storage", "cloud database", or any request involving structured data persistence.
---

# Firebase DB Skill

Integrate Google Firebase Firestore as the primary database for any project in this workspace. This skill handles initialization, CRUD operations, queries, migrations, and real-time listeners using the Firebase Admin SDK.

## Why Firebase?

This workspace runs in an ephemeral Kubernetes pod — local files (SQLite, JSON) are lost on restart. Firebase Firestore provides persistent, cloud-hosted, NoSQL document storage that survives pod restarts and scales automatically. The Admin SDK is already installed and credentials are pre-configured.

## Architecture Overview

```
Your App  →  Firebase Admin SDK (server-side)  →  Firestore (cloud)
                ↕
         Credential File (pre-configured)
```

- **Firestore** stores data as *documents* inside *collections* (like folders and files)
- **Admin SDK** provides full read/write access bypassing security rules (server-side trust)
- **Credential** is stored at `/home/z/my-project/upload/gen-lang-client-0173847591-firebase-adminsdk-fbsvc-0c9f6c5c70.json`

## Project Configuration

| Setting | Value |
|---------|-------|
| **Project ID** | `gen-lang-client-0173847591` |
| **Credential Path** | `/home/z/my-project/upload/gen-lang-client-0173847591-firebase-adminsdk-fbsvc-0c9f6c5c70.json` |
| **SDK Version** | `firebase-admin@13.10.0` |
| **Database Type** | Firestore (Native mode) |
| **Database URL** | `https://gen-lang-client-0173847591.firebaseio.com` |

## Quick Start

### 1. Initialize Firebase in Your Project

Always use this initialization pattern in any backend code that needs Firebase:

```javascript
const admin = require('firebase-admin');
const path = require('path');

// Prevent double initialization
if (admin.apps.length === 0) {
  const serviceAccount = require('/home/z/my-project/upload/gen-lang-client-0173847591-firebase-adminsdk-fbsvc-0c9f6c5c70.json');
  
  admin.initializeApp({
    credential: admin.credential.cert(serviceAccount),
    databaseURL: 'https://gen-lang-client-0173847591.firebaseio.com'
  });
}

const db = admin.firestore();
```

### 2. CRUD Operations

Read `references/firestore-crud.md` for the full CRUD reference with examples. Here's the essential pattern:

```javascript
// CREATE - Add a document to a collection
const docRef = await db.collection('users').add({
  name: 'Ahmad',
  email: 'ahmad@example.com',
  role: 'admin',
  createdAt: admin.firestore.FieldValue.serverTimestamp()
});

// READ - Get a document by ID
const doc = await db.collection('users').doc(docRef.id).get();
if (doc.exists) {
  console.log(doc.id, '=>', doc.data());
}

// UPDATE - Update specific fields
await db.collection('users').doc(docRef.id).update({
  role: 'editor',
  updatedAt: admin.firestore.FieldValue.serverTimestamp()
});

// DELETE - Remove a document
await db.collection('users').doc(docRef.id).delete();
```

### 3. Querying Data

Read `references/firestore-queries.md` for advanced query patterns.

```javascript
// Simple where query
const snapshot = await db.collection('users')
  .where('role', '==', 'admin')
  .get();

snapshot.forEach(doc => {
  console.log(doc.id, '=>', doc.data());
});

// Ordered and limited
const recent = await db.collection('orders')
  .where('status', '==', 'active')
  .orderBy('createdAt', 'desc')
  .limit(10)
  .get();
```

## Data Modeling Patterns

Firestore is NoSQL — design your collections around access patterns, not relationships.

### Pattern 1: Root Collections (most common)
```
users/{userId}        → { name, email, role, createdAt }
orders/{orderId}      → { customerId, total, status, items, createdAt }
attendance/{id}       → { nama, tanggal, status, waktu, keterangan }
```

### Pattern 2: Sub-collections (for one-to-many)
```
users/{userId}/orders/{orderId}  → orders belonging to a specific user
projects/{projectId}/tasks/{taskId}  → tasks within a project
```

### Pattern 3: Composite Keys (alternative to sub-collections)
```
attendance/{nama_tanggal}  → key like "ahmad-fauzi_2026-05-28"
```

### Important Rules
- Avoid deeply nested sub-collections (max 100 levels, but keep under 3)
- Denormalize data for read performance — Firestore charges per document read
- Use `serverTimestamp()` for all time fields to ensure consistency
- Store IDs as document names for O(1) lookups
- Maximum document size: 1 MB

## Migration from File-Based Storage

When a project currently uses JSON files or SQLite and needs to migrate to Firebase, use the bundled migration script:

```bash
# Migrate a JSON file to a Firestore collection
node scripts/migrate-json.js --file /path/to/data.json --collection attendance

# Migrate with custom ID field
node scripts/migrate-json.js --file /path/to/users.json --collection users --id-field email

# Migrate with transform function
node scripts/migrate-json.js --file /path/to/data.json --collection orders --transform scripts/transforms/orders.js
```

Read `references/migration-guide.md` for step-by-step migration instructions.

## Integration with Express.js Backend

When building a backend API that uses Firebase, follow this pattern:

```javascript
const express = require('express');
const admin = require('firebase-admin');

// Initialize (see Quick Start)
const db = admin.firestore();
const app = express();
app.use(express.json());

// GET all items from a collection
app.get('/api/items', async (req, res) => {
  try {
    const snapshot = await db.collection('items').orderBy('createdAt', 'desc').get();
    const items = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
    res.json({ success: true, data: items });
  } catch (error) {
    res.status(500).json({ success: false, error: { message: error.message } });
  }
});

// POST create a new item
app.post('/api/items', async (req, res) => {
  try {
    const docRef = await db.collection('items').add({
      ...req.body,
      createdAt: admin.firestore.FieldValue.serverTimestamp(),
    });
    const doc = await docRef.get();
    res.status(201).json({ success: true, data: { id: doc.id, ...doc.data() } });
  } catch (error) {
    res.status(500).json({ success: false, error: { message: error.message } });
  }
});
```

## Integration with Next.js Fullstack

For Next.js projects using the fullstack-dev skill, create API routes that use Firebase Admin:

```javascript
// app/api/items/route.js
import { getFirebaseDb } from '@/lib/firebase-admin';
import { NextResponse } from 'next/server';

export async function GET() {
  const db = getFirebaseDb();
  const snapshot = await db.collection('items').get();
  const items = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
  return NextResponse.json({ success: true, data: items });
}
```

## Real-Time Listeners (for live dashboards)

```javascript
// Server-side real-time listener (in Node.js backend)
db.collection('attendance')
  .where('tanggal', '==', '2026-05-28')
  .onSnapshot(snapshot => {
    const records = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
    // Push to WebSocket clients, emit via Socket.io, etc.
    broadcastToClients(records);
  }, error => {
    console.error('Listener error:', error);
  });
```

## Environment & Security Notes

- The Admin SDK bypasses Firestore Security Rules — it has full read/write access to all data
- Never expose the credential JSON file to client-side code
- For client-side access (browser), use Firebase Client SDK with proper security rules instead
- The credential file path is fixed: `/home/z/my-project/upload/gen-lang-client-0173847591-firebase-adminsdk-fbsvc-0c9f6c5c70.json`
- Firestore is in the `gen-lang-client-0173847591` GCP project — all data lives there

## Bundled Scripts

| Script | Purpose | Usage |
|--------|---------|-------|
| `scripts/init-firebase.js` | Verify Firebase connection & list collections | `node scripts/init-firebase.js` |
| `scripts/migrate-json.js` | Migrate JSON data file to Firestore collection | `node scripts/migrate-json.js --file data.json --collection name` |
| `scripts/backup-collection.js` | Export a Firestore collection to JSON | `node scripts/backup-collection.js --collection name --output backup.json` |
| `scripts/seed-data.js` | Seed sample data for testing | `node scripts/seed-data.js --collection name --count 10` |

## Reference Files

| File | When to Read |
|------|-------------|
| `references/firestore-crud.md` | When you need full CRUD patterns with error handling, batch writes, and transactions |
| `references/firestore-queries.md` | When you need advanced queries, pagination, aggregation, or compound filters |
| `references/migration-guide.md` | When migrating existing projects from JSON/SQLite/other DBs to Firestore |
| `references/firebase-client.md` | When adding Firebase Client SDK for browser-side real-time features |
| `references/schema-design.md` | When designing collection structure, choosing between root/sub-collections, or modeling relationships |

## Workflow for New Projects

When a new project needs a database:

1. **Initialize** — Run `node scripts/init-firebase.js` to verify the connection
2. **Design schema** — Read `references/schema-design.md` and plan collections
3. **Create API routes** — Use the Express.js integration pattern above
4. **Test** — Use `scripts/seed-data.js` for test data, verify with `scripts/backup-collection.js`
5. **Deploy** — Your backend already runs in this workspace with Firebase as the persistence layer

## Common Pitfalls

- **Missing serverTimestamp()** — Always use `admin.firestore.FieldValue.serverTimestamp()` for time fields, not `new Date()`. Server timestamps are resolved server-side and guaranteed consistent.
- **Double initialization** — Always check `admin.apps.length === 0` before calling `initializeApp()`. The SDK throws if initialized twice.
- **Missing indexes** — Compound queries (multiple `where` clauses + `orderBy`) require composite indexes. Firestore will log an error with a direct link to create the index — follow that link.
- **Document size limit** — Each document max 1 MB. Use sub-collections for large arrays instead of embedding.
- **Not handling non-existent docs** — Always check `doc.exists` before calling `doc.data()`.
