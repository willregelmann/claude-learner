Now I'll create the complete, runnable TypeScript code with Drizzle ORM based on the reference guide.

## schema.ts

```typescript
import { pgTable, serial, text, integer, timestamp } from 'drizzle-orm/pg-core';
import { relations } from 'drizzle-orm';

// Table definitions
export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  email: text('email').notNull().unique(),
  createdAt: timestamp('created_at').defaultNow(),
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
```

## queries.ts

```typescript
import { drizzle } from 'drizzle-orm/node-postgres';
import { Pool } from 'pg';
import { eq } from 'drizzle-orm';
import { users, posts, comments, usersRelations, postsRelations, commentsRelations } from './schema';

// Initialize database connection
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
});

const db = drizzle(pool, {
  schema: {
    users,
    posts,
    comments,
    usersRelations,
    postsRelations,
    commentsRelations,
  },
});

// a. Insert a user and a post by that user in a transaction
export async function insertUserAndPost(userName: string, userEmail: string, postTitle: string, postContent: string) {
  await db.transaction(async (tx) => {
    const [user] = await tx.insert(users).values({
      name: userName,
      email: userEmail,
    }).returning();

    await tx.insert(posts).values({
      title: postTitle,
      content: postContent,
      authorId: user.id,
    });
  });
}

// b. Fetch a post with its author and all comments using the relational query API
export async function fetchPostWithAuthorAndComments(postId: number) {
  const post = await db.query.posts.findFirst({
    where: (posts, { eq }) => eq(posts.id, postId),
    with: {
      author: true,
      comments: {
        with: {
          user: true,
        },
      },
    },
  });

  return post;
}

// c. Update a post's title by ID
export async function updatePostTitle(postId: number, newTitle: string) {
  await db.update(posts)
    .set({ title: newTitle })
    .where(eq(posts.id, postId));
}

// d. Delete all comments by a specific user
export async function deleteCommentsByUser(userId: number) {
  await db.delete(comments)
    .where(eq(comments.userId, userId));
}

// Example usage
async function main() {
  // a. Insert a user and a post in a transaction
  await insertUserAndPost('Alice Johnson', 'alice@example.com', 'My First Post', 'This is the content of my first post.');

  // b. Fetch a post with author and comments
  const postWithData = await fetchPostWithAuthorAndComments(1);
  console.log('Post with author and comments:', JSON.stringify(postWithData, null, 2));

  // c. Update a post's title
  await updatePostTitle(1, 'My Updated Post Title');

  // d. Delete all comments by a specific user
  await deleteCommentsByUser(2);

  await pool.end();
}

main().catch(console.error);
```

## package.json

```json
{
  "name": "drizzle-orm-example",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start": "tsx queries.ts"
  },
  "dependencies": {
    "drizzle-orm": "^0.38.0",
    "pg": "^8.13.0"
  },
  "devDependencies": {
    "@types/node": "^22.0.0",
    "@types/pg": "^8.11.0",
    "tsx": "^4.19.0",
    "typescript": "^5.7.0"
  }
}
```

## tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "outDir": "./dist"
  },
  "include": ["*.ts"],
  "exclude": ["node_modules"]
}
```