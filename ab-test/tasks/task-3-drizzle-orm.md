Using Drizzle ORM with PostgreSQL (drizzle-orm and drizzle-orm/pg-core), create:

1. A schema file defining: users (id, name, email, createdAt), posts (id, title, content, authorId, createdAt), and comments (id, text, postId, userId, createdAt) tables with proper foreign keys and relations using the relations() API.
2. A queries file demonstrating:
   a. Insert a user and a post by that user in a transaction
   b. Fetch a post with its author and all comments using the relational query API (db.query)
   c. Update a post's title by ID
   d. Delete all comments by a specific user

Show complete, runnable TypeScript code with all imports.