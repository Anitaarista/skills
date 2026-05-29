---
name: firebase-db
description: "Firebase Admin SDK database integration for Next.js projects. Replaces Prisma/SQLite with Firebase Firestore and Realtime Database. Use this skill whenever the user mentions Firebase, Firestore, Realtime Database, firebase-admin, or wants to replace the default database system with Firebase. Also trigger when the user asks about cloud database, NoSQL database, real-time data sync, or any task involving Firebase authentication, Firestore CRUD operations, or migrating from Prisma/SQLite to Firebase."
argument-hint: "Describe what you want to do with Firebase (e.g., setup, CRUD, migrate from Prisma)"
---

# Firebase Database Skill

## Overview

This skill replaces the default Prisma ORM + SQLite database system in the workspace with **Firebase Admin SDK**, enabling you to use **Cloud Firestore** and **Firebase Realtime Database** as the primary data layer for Next.js applications.

The workspace currently uses:
- **Prisma ORM** with SQLite (`DATABASE_URL=file:/home/z/my-project/db/custom.db`)
- **Prisma Client** via `import { db } from '@/lib/db'`

After applying this skill, the database layer will use:
- **Firebase Admin SDK** (`firebase-admin`) for server-side operations
- **Cloud Firestore** as the primary NoSQL document database
- **Firebase Realtime Database** for real-time sync scenarios (optional)
- A new `@/lib/firebase` module replacing `@/lib/db`

---

## Setup Instructions

### Step 1: Install Dependencies

Run the following command in the project root:

```bash
cd /home/z/my-project && bun add firebase-admin
```

### Step 2: Configure Firebase Credentials

The Firebase service account key is already available at:

```
/home/z/my-project/upload/smakda-73d57-firebase-adminsdk-fbsvc-845336727d.json
```

Add the following environment variables to `.env`:

```env
FIREBASE_SERVICE_ACCOUNT_PATH=/home/z/my-project/upload/smakda-73d57-firebase-adminsdk-fbsvc-845336727d.json
FIREBASE_PROJECT_ID=smakda-73d57
FIREBASE_DATABASE_URL=https://smakda-73d57-default-rtdb.firebaseio.com
```

### Step 3: Create Firebase Admin Module

Create `src/lib/firebase.ts` in the Next.js project:

```typescript
import * as admin from 'firebase-admin';

// Singleton pattern to prevent multiple initializations
let firebaseApp: admin.app.App;

function getFirebaseApp(): admin.app.App {
  if (firebaseApp) {
    return firebaseApp;
  }

  const serviceAccountPath = process.env.FIREBASE_SERVICE_ACCOUNT_PATH;

  if (!serviceAccountPath) {
    throw new Error('FIREBASE_SERVICE_ACCOUNT_PATH environment variable is not set');
  }

  const serviceAccount = require(serviceAccountPath);

  firebaseApp = admin.initializeApp({
    credential: admin.credential.cert(serviceAccount),
    projectId: process.env.FIREBASE_PROJECT_ID,
    databaseURL: process.env.FIREBASE_DATABASE_URL,
  });

  return firebaseApp;
}

// Firestore instance
export function getFirestore(): admin.firestore.Firestore {
  return getFirebaseApp().firestore();
}

// Realtime Database instance
export function getRealtimeDb(): admin.database.Database {
  return getFirebaseApp().database();
}

// Auth instance
export function getAuth(): admin.auth.Auth {
  return getFirebaseApp().auth();
}

// Default export for convenience
export default getFirebaseApp;
```

### Step 4: Create Database Utility Module

Create `src/lib/firebase-helpers.ts` for common CRUD operations:

```typescript
import {
  getFirestore,
  getRealtimeDb,
} from '@/lib/firebase';
import {
  DocumentData,
  Query,
  WhereFilterOp,
  OrderByDirection,
} from 'firebase-admin/firestore';

// ==========================================
// Firestore CRUD Operations
// ==========================================

/**
 * Create or overwrite a document in a collection
 */
export async function createDocument(
  collection: string,
  data: DocumentData,
  docId?: string
): Promise<string> {
  const db = getFirestore();
  if (docId) {
    await db.collection(collection).doc(docId).set(data);
    return docId;
  }
  const ref = await db.collection(collection).add(data);
  return ref.id;
}

/**
 * Read a single document by ID
 */
export async function getDocument<T = DocumentData>(
  collection: string,
  docId: string
): Promise<(T & { id: string }) | null> {
  const db = getFirestore();
  const doc = await db.collection(collection).doc(docId).get();
  if (!doc.exists) return null;
  return { id: doc.id, ...doc.data() } as T & { id: string };
}

/**
 * Update specific fields of a document (merge)
 */
export async function updateDocument(
  collection: string,
  docId: string,
  data: Partial<DocumentData>
): Promise<void> {
  const db = getFirestore();
  await db.collection(collection).doc(docId).update(data);
}

/**
 * Delete a document by ID
 */
export async function deleteDocument(
  collection: string,
  docId: string
): Promise<void> {
  const db = getFirestore();
  await db.collection(collection).doc(docId).delete();
}

/**
 * Query documents with filters
 */
export async function queryDocuments<T = DocumentData>(
  collection: string,
  filters: Array<{ field: string; operator: WhereFilterOp; value: unknown }>,
  orderBy?: { field: string; direction: OrderByDirection },
  limit?: number
): Promise<(T & { id: string })[]> {
  const db = getFirestore();
  let query: Query = db.collection(collection);

  for (const filter of filters) {
    query = query.where(filter.field, filter.operator, filter.value);
  }

  if (orderBy) {
    query = query.orderBy(orderBy.field, orderBy.direction);
  }

  if (limit) {
    query = query.limit(limit);
  }

  const snapshot = await query.get();
  return snapshot.docs.map(
    (doc) => ({ id: doc.id, ...doc.data() } as T & { id: string })
  );
}

/**
 * Get all documents in a collection
 */
export async function getAllDocuments<T = DocumentData>(
  collection: string
): Promise<(T & { id: string })[]> {
  const db = getFirestore();
  const snapshot = await db.collection(collection).get();
  return snapshot.docs.map(
    (doc) => ({ id: doc.id, ...doc.data() } as T & { id: string })
  );
}

/**
 * Batch write multiple documents
 */
export async function batchWrite(
  operations: Array<{
    type: 'set' | 'update' | 'delete';
    collection: string;
    docId: string;
    data?: DocumentData;
  }>
): Promise<void> {
  const db = getFirestore();
  const batch = db.batch();

  for (const op of operations) {
    const ref = db.collection(op.collection).doc(op.docId);
    switch (op.type) {
      case 'set':
        batch.set(ref, op.data || {});
        break;
      case 'update':
        batch.update(ref, op.data || {});
        break;
      case 'delete':
        batch.delete(ref);
        break;
    }
  }

  await batch.commit();
}

// ==========================================
// Realtime Database Operations
// ==========================================

/**
 * Set data at a path in Realtime Database
 */
export async function rtdbSet(
  path: string,
  data: unknown
): Promise<void> {
  const db = getRealtimeDb();
  await db.ref(path).set(data);
}

/**
 * Read data from a path in Realtime Database
 */
export async function rtdbGet<T = unknown>(path: string): Promise<T | null> {
  const db = getRealtimeDb();
  const snapshot = await db.ref(path).once('value');
  return snapshot.val() as T | null;
}

/**
 * Update specific fields at a path in Realtime Database
 */
export async function rtdbUpdate(
  path: string,
  data: Record<string, unknown>
): Promise<void> {
  const db = getRealtimeDb();
  await db.ref(path).update(data);
}

/**
 * Delete data at a path in Realtime Database
 */
export async function rtdbDelete(path: string): Promise<void> {
  const db = getRealtimeDb();
  await db.ref(path).remove();
}

/**
 * Push new data under a path (auto-generated key)
 */
export async function rtdbPush(
  path: string,
  data: unknown
): Promise<string> {
  const db = getRealtimeDb();
  const ref = await db.ref(path).push(data);
  return ref.key!;
}
```

---

## Migrating from Prisma to Firebase

When the user asks to migrate existing Prisma models to Firebase, follow this pattern:

### 1. Read the Prisma Schema

Read `prisma/schema.prisma` to identify all models and their relations.

### 2. Design Firestore Collections

Prisma models map to Firestore collections. Key differences:
- Firestore is NoSQL — no JOINs; use denormalization or sub-collections
- Relations in Prisma become either:
  - **Sub-collections** (for one-to-many): `users/{userId}/posts`
  - **Reference fields** (for many-to-one): store the parent doc ID as a field
  - **Denormalized data** (for frequently accessed related data)

### 3. Create Migration Script

Generate a migration script at `scripts/migrate-to-firebase.ts`:

```typescript
import { getFirestore } from '../src/lib/firebase';
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();
const db = getFirestore();

async function migrate() {
  // Example: migrate a "User" model
  const users = await prisma.user.findMany();
  for (const user of users) {
    await db.collection('users').doc(user.id).set({
      name: user.name,
      email: user.email,
      createdAt: user.createdAt,
      updatedAt: user.updatedAt,
    });
  }
  console.log(`Migrated ${users.length} users`);
}

migrate()
  .then(() => console.log('Migration complete'))
  .catch(console.error)
  .finally(() => prisma.$disconnect());
```

### 4. Update API Routes

Replace Prisma imports with Firebase imports in all API route files:

**Before (Prisma):**
```typescript
import { db } from '@/lib/db';

// Read
const users = await db.user.findMany();

// Create
const user = await db.user.create({ data: { name: 'Alice' } });

// Update
await db.user.update({ where: { id: '1' }, data: { name: 'Bob' } });

// Delete
await db.user.delete({ where: { id: '1' } });
```

**After (Firebase):**
```typescript
import { getAllDocuments, createDocument, updateDocument, deleteDocument } from '@/lib/firebase-helpers';

// Read
const users = await getAllDocuments('users');

// Create
const userId = await createDocument('users', { name: 'Alice' });

// Update
await updateDocument('users', '1', { name: 'Bob' });

// Delete
await deleteDocument('users', '1');
```

### 5. Update the fullstack-dev Skill Reference

When using the fullstack-dev skill alongside this one, note that:
- The `import { db } from '@/lib/db'` pattern is replaced with Firebase imports
- The Prisma schema steps (`prisma/schema.prisma`, `bun run db:push`) are no longer needed
- Firestore collections are schemaless; define TypeScript interfaces instead of Prisma models

---

## TypeScript Type Definitions

Since Firestore is schemaless, define TypeScript interfaces for type safety. Place them in `src/types/firebase.ts`:

```typescript
// Example type definitions for Firestore collections
export interface User {
  id: string;
  name: string;
  email: string;
  avatarUrl?: string;
  createdAt: admin.firestore.Timestamp;
  updatedAt: admin.firestore.Timestamp;
}

export interface Post {
  id: string;
  title: string;
  content: string;
  authorId: string; // Reference to User
  tags: string[];
  createdAt: admin.firestore.Timestamp;
  updatedAt: admin.firestore.Timestamp;
}
```

---

## API Route Patterns

### Using Firebase in Next.js API Routes

All Firebase Admin SDK operations are server-side only. Use them in API routes (`src/app/api/`) or Server Actions.

**Example API Route — `src/app/api/users/route.ts`:**

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { getAllDocuments, createDocument } from '@/lib/firebase-helpers';

export async function GET() {
  try {
    const users = await getAllDocuments('users');
    return NextResponse.json(users);
  } catch (error) {
    return NextResponse.json({ error: 'Failed to fetch users' }, { status: 500 });
  }
}

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const userId = await createDocument('users', {
      ...body,
      createdAt: new Date(),
      updatedAt: new Date(),
    });
    return NextResponse.json({ id: userId }, { status: 201 });
  } catch (error) {
    return NextResponse.json({ error: 'Failed to create user' }, { status: 500 });
  }
}
```

**Example API Route — `src/app/api/users/[id]/route.ts`:**

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { getDocument, updateDocument, deleteDocument } from '@/lib/firebase-helpers';

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const user = await getDocument('users', params.id);
    if (!user) {
      return NextResponse.json({ error: 'User not found' }, { status: 404 });
    }
    return NextResponse.json(user);
  } catch (error) {
    return NextResponse.json({ error: 'Failed to fetch user' }, { status: 500 });
  }
}

export async function PUT(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const body = await request.json();
    await updateDocument('users', params.id, {
      ...body,
      updatedAt: new Date(),
    });
    return NextResponse.json({ success: true });
  } catch (error) {
    return NextResponse.json({ error: 'Failed to update user' }, { status: 500 });
  }
}

export async function DELETE(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    await deleteDocument('users', params.id);
    return NextResponse.json({ success: true });
  } catch (error) {
    return NextResponse.json({ error: 'Failed to delete user' }, { status: 500 });
  }
}
```

---

## Firestore Indexes

For complex queries with multiple filters and ordering, Firestore requires composite indexes. If a query fails with an index error, create the index:

1. The error message will include a direct link to create the index in the Firebase Console
2. Or define indexes in `firestore.indexes.json`:

```json
{
  "indexes": [
    {
      "collectionGroup": "posts",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "authorId", "order": "ASCENDING" },
        { "fieldPath": "createdAt", "order": "DESCENDING" }
      ]
    }
  ],
  "fieldOverrides": []
}
```

---

## Common Patterns Reference

Read `references/firestore-patterns.md` for advanced patterns including:
- Sub-collections for nested data
- Transactions for atomic operations
- Paginated queries with cursors
- Aggregation queries
- Real-time listeners (client-side with Firebase Client SDK)
- Data denormalization strategies

Read `references/rtdb-patterns.md` for Realtime Database patterns including:
- Real-time data sync
- Presence detection
- Atomic multi-path updates
- Security rules design

---

## Troubleshooting

### "FIREBASE_SERVICE_ACCOUNT_PATH is not set"
Make sure `.env` contains the `FIREBASE_SERVICE_ACCOUNT_PATH` variable and the file exists at that path.

### "The default Firebase app already exists"
The singleton pattern in `src/lib/firebase.ts` prevents this. If it still occurs, ensure the module is not being imported from multiple conflicting paths.

### Firestore query requires an index
Follow the link in the error message to create the composite index, or add it to `firestore.indexes.json`.

### Permission denied on Firestore
Ensure the service account has the correct IAM permissions (Cloud Datastore User or Firebase Admin) in the Firebase Console.

### Realtime Database URL not set
Add `FIREBASE_DATABASE_URL` to `.env`. The URL format is `https://{project-id}-default-rtdb.firebaseio.com`.
