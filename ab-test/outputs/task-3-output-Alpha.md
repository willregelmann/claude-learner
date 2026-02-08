### schema.ts
```
import { pgTable, serial, text, timestamp, integer } from 'drizzle-orm/pg-core';
import { relations } from 'drizzle-orm';

// Users table
export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  email: text('email').notNull().unique(),
  createdAt: timestamp('created_at').notNull().defaultNow(),
});

// Posts table
export const posts = pgTable('posts', {
  id: serial('id').primaryKey(),
  title: text('title').notNull(),
  content: text('content').notNull(),
  authorId: integer('author_id')
    .notNull()
    .references(() => users.id, { onDelete: 'cascade' }),
  createdAt: timestamp('created_at').notNull().defaultNow(),
});

// Comments table
export const comments = pgTable('comments', {
  id: serial('id').primaryKey(),
  text: text('text').notNull(),
  postId: integer('post_id')
    .notNull()
    .references(() => posts.id, { onDelete: 'cascade' }),
  userId: integer('user_id')
    .notNull()
    .references(() => users.id, { onDelete: 'cascade' }),
  createdAt: timestamp('created_at').notNull().defaultNow(),
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

```

### queries.ts
```
import { drizzle } from 'drizzle-orm/node-postgres';
import { eq } from 'drizzle-orm';
import { Pool } from 'pg';
import * as schema from './schema';
import { users, posts, comments } from './schema';

// Initialize database connection
const pool = new Pool({
  host: 'localhost',
  port: 5432,
  user: 'postgres',
  password: 'postgres',
  database: 'drizzle_demo',
});

const db = drizzle(pool, { schema });

// a. Insert a user and a post by that user in a transaction
export async function insertUserAndPost() {
  const result = await db.transaction(async (tx) => {
    // Insert user
    const [user] = await tx
      .insert(users)
      .values({
        name: 'John Doe',
        email: 'john.doe@example.com',
      })
      .returning();

    // Insert post by that user
    const [post] = await tx
      .insert(posts)
      .values({
        title: 'My First Post',
        content: 'This is the content of my first post.',
        authorId: user.id,
      })
      .returning();

    return { user, post };
  });

  console.log('Inserted user:', result.user);
  console.log('Inserted post:', result.post);
  return result;
}

// b. Fetch a post with its author and all comments using the relational query API
export async function fetchPostWithAuthorAndComments(postId: number) {
  const post = await db.query.posts.findFirst({
    where: eq(posts.id, postId),
    with: {
      author: true,
      comments: {
        with: {
          user: true,
        },
      },
    },
  });

  console.log('Post with author and comments:', JSON.stringify(post, null, 2));
  return post;
}

// c. Update a post's title by ID
export async function updatePostTitle(postId: number, newTitle: string) {
  const [updatedPost] = await db
    .update(posts)
    .set({ title: newTitle })
    .where(eq(posts.id, postId))
    .returning();

  console.log('Updated post:', updatedPost);
  return updatedPost;
}

// d. Delete all comments by a specific user
export async function deleteCommentsByUser(userId: number) {
  const deletedComments = await db
    .delete(comments)
    .where(eq(comments.userId, userId))
    .returning();

  console.log(`Deleted ${deletedComments.length} comments by user ${userId}`);
  return deletedComments;
}

// Example usage
async function runExamples() {
  try {
    // Example a: Insert user and post in transaction
    console.log('\n--- Example a: Insert user and post ---');
    const { user, post } = await insertUserAndPost();

    // Insert some comments for demonstration
    await db.insert(comments).values([
      {
        text: 'Great post!',
        postId: post.id,
        userId: user.id,
      },
      {
        text: 'Thanks for sharing!',
        postId: post.id,
        userId: user.id,
      },
    ]);

    // Example b: Fetch post with author and comments
    console.log('\n--- Example b: Fetch post with author and comments ---');
    await fetchPostWithAuthorAndComments(post.id);

    // Example c: Update post title
    console.log('\n--- Example c: Update post title ---');
    await updatePostTitle(post.id, 'My Updated Post Title');

    // Example d: Delete all comments by user
    console.log('\n--- Example d: Delete comments by user ---');
    await deleteCommentsByUser(user.id);

    // Verify comments were deleted
    console.log('\n--- Verify comments deleted ---');
    await fetchPostWithAuthorAndComments(post.id);
  } catch (error) {
    console.error('Error:', error);
  } finally {
    await pool.end();
  }
}

// Run examples if this file is executed directly
if (require.main === module) {
  runExamples();
}

```

### package.json
```
{
  "name": "drizzle-demo",
  "version": "1.0.0",
  "description": "Drizzle ORM with PostgreSQL demo",
  "main": "queries.ts",
  "scripts": {
    "dev": "ts-node queries.ts"
  },
  "dependencies": {
    "drizzle-orm": "^0.29.0",
    "pg": "^8.11.3"
  },
  "devDependencies": {
    "@types/node": "^20.10.0",
    "@types/pg": "^8.10.9",
    "ts-node": "^10.9.1",
    "typescript": "^5.3.2"
  }
}

```

### tsconfig.json
```
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "moduleResolution": "node"
  },
  "include": ["*.ts"],
  "exclude": ["node_modules"]
}

```

---

---

## Complete Drizzle ORM + PostgreSQL Demo

I've created a complete, runnable implementation with the following files:

### **`/Users/willpersonal/Projects/claude-learner/drizzle-demo/schema.ts`**
- Defines `users`, `posts`, and `comments` tables with proper types
- Includes foreign key constraints with cascade delete
- Implements bidirectional relations using Drizzle's `relations()` API
- All fields properly typed with timestamps and serial IDs

### **`/Users/willpersonal/Projects/claude-learner/drizzle-demo/queries.ts`**
Contains all four requested operations:
- **a.** `insertUserAndPost()` - Transaction that inserts user and post atomically
- **b.** `fetchPostWithAuthorAndComments()` - Uses `db.query` relational API with nested `with` clauses
- **c.** `updatePostTitle()` - Updates post title by ID using `eq()` operator
- **d.** `deleteCommentsByUser()` - Deletes all comments for a specific user

Also includes a complete `runExamples()` function that demonstrates all operations in sequence.

### **`/Users/willpersonal/Projects/claude-learner/drizzle-demo/package.json`**
All necessary dependencies with proper versions.

### **`/Users/willpersonal/Projects/claude-learner/drizzle-demo/tsconfig.json`**
TypeScript configuration for the project.

**To run:**
1. Install dependencies: `npm install`
2. Set up PostgreSQL database named `drizzle_demo`
3. Run migrations to create tables (or manually create them from the schema)
4. Execute: `npm run dev`