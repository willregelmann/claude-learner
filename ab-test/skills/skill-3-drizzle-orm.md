---
name: drizzle-orm-postgresql
description: Use when working with Drizzle ORM and PostgreSQL, defining schemas, relations, writing queries, or performing CRUD operations.
---

# Drizzle ORM with PostgreSQL

## When This Skill Applies

- Defining PostgreSQL database schemas using TypeScript with `drizzle-orm/pg-core`
- Creating table relationships (one-to-one, one-to-many, many-to-many) using the `relations()` API
- Querying nested relational data with `db.query.tableName.findFirst()` or `findMany()`
- Performing insert, update, delete operations with type safety
- Managing database transactions and batch operations
- Migrating from other ORMs (Prisma, TypeORM) or raw SQL to Drizzle
- Troubleshooting common Drizzle ORM issues and anti-patterns
- Working with prepared statements and parameterized queries for performance

## Key Patterns

### Schema Definition with pg-core

Import table and column types from `drizzle-orm/pg-core`. Use `pgTable()` to define tables with TypeScript type inference.

```typescript
import { pgTable, serial, text, integer, timestamp, boolean, varchar } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  email: varchar('email', { length: 255 }).unique().notNull(),
  age: integer('age'),
  isActive: boolean('is_active').default(true),
  createdAt: timestamp('created_at').defaultNow(),
});

export const posts = pgTable('posts', {
  id: serial('id').primaryKey(),
  title: text('title').notNull(),
  content: text('content'),
  authorId: integer('author_id').notNull().references(() => users.id),
  publishedAt: timestamp('published_at'),
});
```

**Use identity columns over serial for new projects.** PostgreSQL recommends identity columns as the modern approach. Drizzle supports this with `integer('id').generatedAlwaysAsIdentity()`.

### Custom Schemas

Organize tables into PostgreSQL schemas using `pgSchema()`:

```typescript
import { pgSchema, serial, text } from 'drizzle-orm/pg-core';

export const authSchema = pgSchema('auth');
export const users = authSchema.table('users', {
  id: serial('id').primaryKey(),
  username: text('username').notNull(),
});
```

### Defining Relations with relations()

**Critical distinction:** Foreign keys are database-level constraints; relations are application-level abstractions for querying. You can use them together or separately. Relations enable the relational query API but do not create database constraints.

```typescript
import { relations } from 'drizzle-orm';
import { pgTable, serial, text, integer } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
});

export const posts = pgTable('posts', {
  id: serial('id').primaryKey(),
  title: text('title').notNull(),
  authorId: integer('author_id').notNull(),
});

// Define relations separately from table schemas
export const usersRelations = relations(users, ({ many }) => ({
  posts: many(posts),
}));

export const postsRelations = relations(posts, ({ one }) => ({
  author: one(users, {
    fields: [posts.authorId],
    references: [users.id],
  }),
}));
```

**One-to-Many:** Use `many()` on the "one" side, `one()` on the "many" side.

**One-to-One:** Use `one()` on both sides with appropriate foreign key placement.

**Many-to-Many:** Requires an explicit junction table:

```typescript
export const categories = pgTable('categories', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
});

export const articles = pgTable('articles', {
  id: serial('id').primaryKey(),
  title: text('title').notNull(),
});

// Junction table
export const articlesToCategories = pgTable('articles_to_categories', {
  articleId: integer('article_id').notNull().references(() => articles.id),
  categoryId: integer('category_id').notNull().references(() => categories.id),
});

export const categoriesRelations = relations(categories, ({ many }) => ({
  articlesToCategories: many(articlesToCategories),
}));

export const articlesRelations = relations(articles, ({ many }) => ({
  articlesToCategories: many(articlesToCategories),
}));

export const articlesToCategoriesRelations = relations(articlesToCategories, ({ one }) => ({
  article: one(articles, {
    fields: [articlesToCategories.articleId],
    references: [articles.id],
  }),
  category: one(categories, {
    fields: [articlesToCategories.categoryId],
    references: [categories.id],
  }),
}));
```

### Relational Query API with db.query

Use `db.query.tableName.findMany()` or `findFirst()` to query with nested relations. The `with` clause loads related data in a single SQL query.

```typescript
// Find all users with their posts
const usersWithPosts = await db.query.users.findMany({
  with: {
    posts: true,
  },
});

// Find first user by email with nested relations
const user = await db.query.users.findFirst({
  where: (users, { eq }) => eq(users.email, 'user@example.com'),
  with: {
    posts: {
      columns: {
        title: true,
        publishedAt: true,
      },
      with: {
        comments: true,
      },
    },
  },
});

// Partial column selection
const usersPartial = await db.query.users.findMany({
  columns: {
    id: true,
    name: true,
  },
  with: {
    posts: {
      columns: {
        title: true,
      },
    },
  },
});
```

**Performance note:** Drizzle generates exactly one SQL query regardless of nesting depth. No N+1 query problem.

### Select API (Alternative to Relational Queries)

For complex queries or when you need full SQL control, use the select API with explicit joins:

```typescript
import { eq } from 'drizzle-orm';

const result = await db
  .select({
    userId: users.id,
    userName: users.name,
    postTitle: posts.title,
  })
  .from(users)
  .leftJoin(posts, eq(posts.authorId, users.id))
  .where(eq(users.isActive, true));
```

**When to use select vs query:** Use `db.query` for straightforward relational data fetching. Use `db.select()` for complex aggregations, custom joins, or when you need explicit SQL control.

### Insert Operations

```typescript
// Single insert
const newUser = await db.insert(users).values({
  name: 'John Doe',
  email: 'john@example.com',
}).returning();

// Multiple inserts
await db.insert(posts).values([
  { title: 'Post 1', authorId: 1 },
  { title: 'Post 2', authorId: 1 },
]);

// Insert with onConflictDoUpdate (upsert)
await db.insert(users)
  .values({ id: 1, name: 'John', email: 'john@example.com' })
  .onConflictDoUpdate({
    target: users.email,
    set: { name: 'John Updated' },
  });
```

### Update Operations

```typescript
import { eq } from 'drizzle-orm';

// Update with WHERE clause
await db.update(users)
  .set({ name: 'Jane Doe', age: 30 })
  .where(eq(users.id, 1));

// Update with returning
const updated = await db.update(posts)
  .set({ publishedAt: new Date() })
  .where(eq(posts.id, 5))
  .returning();

// Undefined values are ignored; use null explicitly
await db.update(users)
  .set({ age: null }) // Sets to NULL
  .where(eq(users.id, 2));
```

### Delete Operations

```typescript
import { eq, lt } from 'drizzle-orm';

// Delete with WHERE clause
await db.delete(posts).where(eq(posts.id, 10));

// Delete with multiple conditions
await db.delete(posts)
  .where(lt(posts.publishedAt, new Date('2020-01-01')));

// Delete with returning
const deleted = await db.delete(users)
  .where(eq(users.id, 5))
  .returning();
```

### Transactions

Wrap multiple operations in `db.transaction()`. All operations execute on the transaction object (`tx`), not `db`.

```typescript
await db.transaction(async (tx) => {
  const user = await tx.insert(users).values({
    name: 'Alice',
    email: 'alice@example.com',
  }).returning();

  await tx.insert(posts).values({
    title: 'First Post',
    authorId: user[0].id,
  });

  // Rollback by throwing an error
  if (someCondition) {
    throw new Error('Rollback transaction');
  }
});
```

**Nested transactions with savepoints:**

```typescript
await db.transaction(async (tx) => {
  await tx.insert(users).values({ name: 'Bob', email: 'bob@example.com' });

  await tx.transaction(async (tx2) => {
    // This is a savepoint
    await tx2.insert(posts).values({ title: 'Nested', authorId: 1 });
  });
});
```

### Prepared Statements for Performance

Use `.prepare()` with `sql.placeholder()` for queries executed multiple times. Database reuses the compiled query plan.

```typescript
import { sql, placeholder } from 'drizzle-orm';

const prepared = db.select()
  .from(users)
  .where(eq(users.id, placeholder('id')))
  .prepare('get_user_by_id');

const user1 = await prepared.execute({ id: 1 });
const user2 = await prepared.execute({ id: 2 });
```

**PostgreSQL note:** All values in filter operators are parameterized automatically (e.g., `$1`, `$2`) to prevent SQL injection.

### Batch Operations

For supported drivers (Neon, LibSQL, D1), execute multiple statements in a single network round-trip:

```typescript
const batchResult = await db.batch([
  db.insert(users).values({ name: 'User 1', email: 'user1@example.com' }),
  db.insert(users).values({ name: 'User 2', email: 'user2@example.com' }),
  db.select().from(users),
]);
```

### Dynamic SQL with sql`` Operator

For raw SQL or dynamic expressions:

```typescript
import { sql } from 'drizzle-orm';

// Raw SQL in WHERE clause
const result = await db.select()
  .from(users)
  .where(sql`${users.createdAt} > NOW() - INTERVAL '7 days'`);

// Dynamic ORDER BY
const orderByField = users.name;
const ordered = await db.select()
  .from(users)
  .orderBy(sql`${orderByField} ASC`);
```

## Common Mistakes to Avoid

**Mistake:** Defining foreign keys without relations (or vice versa) and expecting relational queries to work.  
**Fix:** Foreign keys enforce database integrity; `relations()` enable `db.query` API. Define both if you want constraints and relational queries.

**Mistake:** Using `db` inside a transaction callback instead of `tx`.  
**Fix:** Always use the transaction object: `await tx.insert(users)...` not `await db.insert(users)...`.

**Mistake:** Modifying migration files manually after they've been applied.  
**Fix:** Never edit migration history. Generate new migrations with `drizzle-kit generate` for schema changes.

**Mistake:** Forgetting `.returning()` after insert/update/delete and expecting to get the affected rows.  
**Fix:** Add `.returning()` to get inserted/updated/deleted data: `await db.insert(users).values({...}).returning()`.

**Mistake:** Using `leftJoin()` without understanding null handling, leading to unexpected `null` values.  
**Fix:** Use `innerJoin()` when you expect matching rows on both sides. Use `leftJoin()` only when nulls are acceptable.

**Mistake:** Selecting all columns in large tables when only a few are needed.  
**Fix:** Use the `columns` parameter in `db.query` or specify fields in `db.select()` to reduce data transfer.

**Mistake:** Treating Drizzle relations as SQL relationships (like Prisma models).  
**Fix:** Understand that Drizzle uses TypeScript schema definitions and generates SQL joins behind the scenes, not ORM-style abstractions.

**Mistake:** Passing `undefined` to `.set()` in updates expecting it to set NULL.  
**Fix:** `undefined` values are ignored. Use `null` explicitly: `.set({ age: null })`.

**Mistake:** Confusing the relational query API (`db.query`) with the select API (`db.select()`).  
**Fix:** Use `db.query` for nested relations with `with`. Use `db.select()` for custom joins, aggregations, or full SQL control.

**Mistake:** Not using prepared statements for frequently executed queries with parameters.  
**Fix:** Use `.prepare()` with `placeholder()` to improve performance by reusing query plans.

## Examples

### Example 1: Blog System with Comments

```typescript
import { pgTable, serial, text, integer, timestamp } from 'drizzle-orm/pg-core';
import { relations } from 'drizzle-orm';
import { drizzle } from 'drizzle-orm/node-postgres';
import { Pool } from 'pg';

// Schema
export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  username: text('username').notNull().unique(),
});

export const posts = pgTable('posts', {
  id: serial('id').primaryKey(),
  title: text('title').notNull(),
  content: text('content'),
  authorId: integer('author_id').notNull().references(() => users.id),
  createdAt: timestamp('created_at').defaultNow(),
});

export const comments = pgTable('comments', {
  id: serial('id').primaryKey(),
  text: text('text').notNull(),
  postId: integer('post_id').notNull().references(() => posts.id),
  userId: integer('user_id').notNull().references(() => users.id),
  createdAt: timestamp('created_at').defaultNow(),
});

// Relations
export const usersRelations = relations(users, ({ many }) => ({
  posts: many(posts),
  comments: many(comments),
}));

export const postsRelations = relations(posts, ({ one, many }) => ({
  author: one(users, {
    fields: [posts.authorId],
    references: [users.id],
  }),
  comments: many(comments),
}));

export const commentsRelations = relations(comments, ({ one }) => ({
  post: one(posts, {
    fields: [comments.postId],
    references: [posts.id],
  }),
  user: one(users, {
    fields: [comments.userId],
    references: [users.id],
  }),
}));

// Query
const pool = new Pool({ connectionString: process.env.DATABASE_URL });
const db = drizzle(pool);

// Get all posts with author and comments
const postsWithData = await db.query.posts.findMany({
  with: {
    author: {
      columns: { username: true },
    },
    comments: {
      with: {
        user: {
          columns: { username: true },
        },
      },
    },
  },
  orderBy: (posts, { desc }) => [desc(posts.createdAt)],
  limit: 10,
});
```

### Example 2: E-commerce with Many-to-Many

```typescript
import { pgTable, serial, text, integer, numeric } from 'drizzle-orm/pg-core';
import { relations } from 'drizzle-orm';
import { eq } from 'drizzle-orm';

// Schema
export const products = pgTable('products', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  price: numeric('price', { precision: 10, scale: 2 }).notNull(),
});

export const orders = pgTable('orders', {
  id: serial('id').primaryKey(),
  userId: integer('user_id').notNull(),
  status: text('status').notNull().default('pending'),
});

export const orderItems = pgTable('order_items', {
  id: serial('id').primaryKey(),
  orderId: integer('order_id').notNull().references(() => orders.id),
  productId: integer('product_id').notNull().references(() => products.id),
  quantity: integer('quantity').notNull().default(1),
});

// Relations
export const ordersRelations = relations(orders, ({ many }) => ({
  items: many(orderItems),
}));

export const productsRelations = relations(products, ({ many }) => ({
  orderItems: many(orderItems),
}));

export const orderItemsRelations = relations(orderItems, ({ one }) => ({
  order: one(orders, {
    fields: [orderItems.orderId],
    references: [orders.id],
  }),
  product: one(products, {
    fields: [orderItems.productId],
    references: [products.id],
  }),
}));

// Transaction: Create order with items
await db.transaction(async (tx) => {
  const [order] = await tx.insert(orders).values({
    userId: 123,
    status: 'pending',
  }).returning();

  await tx.insert(orderItems).values([
    { orderId: order.id, productId: 1, quantity: 2 },
    { orderId: order.id, productId: 3, quantity: 1 },
  ]);
});

// Query order with products
const orderWithProducts = await db.query.orders.findFirst({
  where: eq(orders.id, 1),
  with: {
    items: {
      with: {
        product: true,
      },
    },
  },
});
```

Sources:
- [Drizzle ORM - Schema](https://orm.drizzle.team/docs/sql-schema-declaration)
- [Drizzle ORM - Drizzle Relations](https://orm.drizzle.team/docs/relations-v2)
- [Drizzle ORM - Query](https://orm.drizzle.team/docs/rqb-v2)
- [Drizzle ORM - Transactions](https://orm.drizzle.team/docs/transactions)
- [Drizzle ORM - Insert](https://orm.drizzle.team/docs/insert)
- [3 Biggest Mistakes with Drizzle ORM](https://medium.com/@lior_amsalem/3-biggest-mistakes-with-drizzle-orm-1327e2531aff)
- [API with NestJS #181. Prepared statements in PostgreSQL with Drizzle ORM](https://wanago.io/2024/12/30/nestjs-api-prepared-statements-drizzle-postgresql/)