---
name: react-19-useactionstate
description: Use when building forms with React 19 useActionState, server actions, form validation, or migrating from useFormState.
---

# React 19 useActionState and Server Actions

## When This Skill Applies

- Building forms with server-side validation in React 19 or Next.js 15
- Managing form submission states (pending, success, error)
- Migrating from the deprecated `useFormState` hook to `useActionState`
- Implementing progressive enhancement for forms that work before JavaScript loads
- Handling async server actions with proper error and validation feedback
- Combining `useActionState` with `useFormStatus` for comprehensive form state management
- Creating forms that display field-level validation errors from server actions
- Building forms with optimistic updates using `useOptimistic` alongside server actions

## Key Patterns

### Hook Signature and Return Values

Call `useActionState` at the top level of your component to manage form state tied to server actions.

```javascript
import { useActionState } from 'react';

const [state, formAction, isPending] = useActionState(fn, initialState, permalink?);
```

**Parameters:**
- `fn`: Action function to call when form submits (receives `prevState` and `formData`)
- `initialState`: Initial state value before any action executes
- `permalink` (optional): URL string for progressive enhancement on dynamic content pages

**Returns:**
- `state`: Current state (initially `initialState`, then the return value of your action function)
- `formAction`: Function to pass to form's `action` prop or button's `formAction` prop
- `isPending`: Boolean indicating if the action is currently executing

### Server Action Function Signature

Define server actions with the correct parameter order. When using `useActionState`, the action receives `prevState` as the first parameter, followed by `formData`.

```javascript
'use server'

async function updateUser(prevState, formData) {
  // prevState is FIRST (injected by useActionState)
  // formData is SECOND (from the form submission)
  
  const name = formData.get('name');
  const email = formData.get('email');
  
  // Perform validation and return new state
  if (!email.includes('@')) {
    return {
      error: 'Invalid email address',
      fields: { name, email }
    };
  }
  
  // Success case
  await saveUser({ name, email });
  return { success: true, message: 'User updated successfully' };
}
```

### Client Component Form Setup

Create a client component using `useActionState` to wire up the server action and manage state.

```javascript
'use client'

import { useActionState } from 'react';
import { updateUser } from './actions';

export function UserForm() {
  const [state, formAction, isPending] = useActionState(updateUser, {
    error: null,
    fields: {}
  });
  
  return (
    <form action={formAction}>
      <input 
        name="name" 
        defaultValue={state.fields?.name}
        aria-describedby="name-error"
      />
      {state.error && <span id="name-error">{state.error}</span>}
      
      <button type="submit" disabled={isPending}>
        {isPending ? 'Saving...' : 'Save'}
      </button>
      
      {state.success && <p>{state.message}</p>}
    </form>
  );
}
```

### Server-Side Validation with Zod

Implement robust server-side validation using Zod and return field-specific errors.

```javascript
'use server'

import { z } from 'zod';

const schema = z.object({
  email: z.string().email('Invalid email address'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
  age: z.coerce.number().min(18, 'Must be 18 or older')
});

async function createAccount(prevState, formData) {
  const validatedFields = schema.safeParse({
    email: formData.get('email'),
    password: formData.get('password'),
    age: formData.get('age')
  });
  
  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
      message: 'Validation failed'
    };
  }
  
  // Process valid data
  await db.createUser(validatedFields.data);
  return { success: true };
}
```

Display field-specific errors in the form:

```javascript
'use client'

import { useActionState } from 'react';

export function SignupForm() {
  const [state, formAction, isPending] = useActionState(createAccount, {});
  
  return (
    <form action={formAction}>
      <div>
        <input name="email" type="email" />
        {state.errors?.email && (
          <p className="error">{state.errors.email[0]}</p>
        )}
      </div>
      
      <div>
        <input name="password" type="password" />
        {state.errors?.password && (
          <p className="error">{state.errors.password[0]}</p>
        )}
      </div>
      
      <div>
        <input name="age" type="number" />
        {state.errors?.age && (
          <p className="error">{state.errors.age[0]}</p>
        )}
      </div>
      
      <button disabled={isPending}>
        {isPending ? 'Creating account...' : 'Sign up'}
      </button>
    </form>
  );
}
```

### Using useFormStatus for Enhanced Pending States

Use `useFormStatus` in child components to access detailed submission status. `useFormStatus` must be called in a component that is a child of the form, not in the form component itself.

```javascript
'use client'

import { useFormStatus } from 'react-dom/hooks/useFormStatus';

function SubmitButton() {
  const { pending, data, method, action } = useFormStatus();
  
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Submitting...' : 'Submit'}
    </button>
  );
}

export function MyForm() {
  const [state, formAction] = useActionState(myAction, null);
  
  return (
    <form action={formAction}>
      <input name="username" />
      {/* useFormStatus must be in a child component */}
      <SubmitButton />
    </form>
  );
}
```

### Module-Level vs Inline "use server" Directives

Use module-level directive for files containing only server actions. Use inline directive when mixing client and server code.

**Module-level (recommended for dedicated action files):**

```javascript
// app/actions.js
'use server'

export async function createPost(prevState, formData) {
  // All functions in this file are server actions
}

export async function deletePost(prevState, formData) {
  // This is also a server action
}
```

**Inline (for client components that define server actions):**

```javascript
'use client'

import { useActionState } from 'react';

export function PostForm() {
  async function createPost(prevState, formData) {
    'use server'
    // This specific function is a server action
    const title = formData.get('title');
    await db.posts.create({ title });
    return { success: true };
  }
  
  const [state, formAction] = useActionState(createPost, {});
  return <form action={formAction}>...</form>;
}
```

**Important:** Client components can only import actions with module-level `'use server'` directives. They cannot import functions with inline `'use server'` from server components.

### Progressive Enhancement with Permalink

Use the optional permalink parameter to enable form submissions before JavaScript loads on pages with dynamic content.

```javascript
'use client'

import { useActionState } from 'react';

export function CommentForm({ postId }) {
  const [state, formAction] = useActionState(
    addComment,
    { message: null },
    `/posts/${postId}` // permalink for progressive enhancement
  );
  
  return (
    <form action={formAction}>
      <textarea name="comment" required />
      <button type="submit">Post Comment</button>
    </form>
  );
}
```

When the form is submitted before JavaScript loads, the browser navigates to the permalink URL. The same form component must be rendered at that URL for React to pass the state through. After hydration, the permalink has no effect.

### Preserving Form Input Values on Validation Errors

Preserve user input when validation fails by storing field values in state and using `defaultValue`.

```javascript
'use server'

async function submitForm(prevState, formData) {
  const email = formData.get('email');
  const name = formData.get('name');
  
  if (!email.includes('@')) {
    return {
      error: 'Invalid email',
      fields: { email, name } // Preserve entered values
    };
  }
  
  await saveData({ email, name });
  return { success: true, fields: {} };
}
```

```javascript
'use client'

export function MyForm() {
  const [state, formAction] = useActionState(submitForm, { fields: {} });
  
  return (
    <form action={formAction}>
      <input 
        name="name" 
        defaultValue={state.fields.name || ''}
      />
      <input 
        name="email" 
        defaultValue={state.fields.email || ''}
      />
      {state.error && <p>{state.error}</p>}
      <button>Submit</button>
    </form>
  );
}
```

### Passing Server Action Directly to Form

Pass the wrapped action directly to the form's `action` prop. Avoid creating wrapper functions that prevent the form from working without JavaScript.

```javascript
// ✅ CORRECT - Progressive enhancement works
const [state, formAction] = useActionState(myAction, initialState);
return <form action={formAction}>...</form>;

// ❌ WRONG - Form won't work without JavaScript
const [state, formAction] = useActionState(myAction, initialState);
const handleSubmit = (e) => {
  e.preventDefault();
  formAction(new FormData(e.target));
};
return <form onSubmit={handleSubmit}>...</form>;
```

### Security: Validate All Inputs

Always treat arguments to server functions as untrusted client input. Validate and sanitize all data.

```javascript
'use server'

import { z } from 'zod';

export async function updateProfile(prevState, formData) {
  // ✅ ALWAYS validate inputs
  const schema = z.object({
    bio: z.string().max(500),
    website: z.string().url().optional()
  });
  
  const result = schema.safeParse({
    bio: formData.get('bio'),
    website: formData.get('website')
  });
  
  if (!result.success) {
    return { error: 'Invalid input', errors: result.error.flatten() };
  }
  
  // ❌ NEVER trust raw formData without validation
  // const bio = formData.get('bio'); 
  // await db.update({ bio }); // Dangerous!
  
  // ✅ Use validated data
  await db.update(result.data);
  return { success: true };
}
```

## Common Mistakes to Avoid

**Wrong parameter order in action function** → Action functions wrapped by `useActionState` receive `prevState` as the first parameter, `formData` as the second. Don't confuse this with standalone server actions that receive only `formData`.

```javascript
// ❌ WRONG
async function myAction(formData, prevState) { }

// ✅ CORRECT
async function myAction(prevState, formData) { }
```

**Missing "use server" directive** → Server actions must have `'use server'` at module or function level. Without it, the function executes on the client.

```javascript
// ❌ WRONG - No directive
async function saveData(prevState, formData) {
  await db.save(formData);
}

// ✅ CORRECT - Module level
'use server'
async function saveData(prevState, formData) {
  await db.save(formData);
}

// ✅ CORRECT - Inline
async function saveData(prevState, formData) {
  'use server'
  await db.save(formData);
}
```

**Using useFormStatus in the same component as useActionState** → `useFormStatus` must be called in a child component of the form, not in the component that renders the form.

```javascript
// ❌ WRONG
function MyForm() {
  const [state, formAction] = useActionState(myAction, {});
  const { pending } = useFormStatus(); // Error: not in form context
  return <form action={formAction}>...</form>;
}

// ✅ CORRECT
function SubmitButton() {
  const { pending } = useFormStatus(); // Works: inside form as child
  return <button disabled={pending}>Submit</button>;
}

function MyForm() {
  const [state, formAction] = useActionState(myAction, {});
  return (
    <form action={formAction}>
      <SubmitButton />
    </form>
  );
}
```

**Forgetting async on server actions** → Server actions must be async functions because network calls are asynchronous.

```javascript
// ❌ WRONG
function myAction(prevState, formData) {
  'use server'
  db.save(formData);
}

// ✅ CORRECT
async function myAction(prevState, formData) {
  'use server'
  await db.save(formData);
}
```

**Not validating server-side inputs** → Always validate form data on the server. Client-side validation can be bypassed.

```javascript
// ❌ WRONG - Trusting client input
async function updateEmail(prevState, formData) {
  'use server'
  const email = formData.get('email');
  await db.updateEmail(email); // No validation!
}

// ✅ CORRECT - Server-side validation
async function updateEmail(prevState, formData) {
  'use server'
  const email = formData.get('email');
  if (!email || !email.includes('@')) {
    return { error: 'Invalid email' };
  }
  await db.updateEmail(email);
  return { success: true };
}
```

**Using useFormState instead of useActionState** → `useFormState` is deprecated. Migrate to `useActionState` (imported from `react`, not `react-dom`).

```javascript
// ❌ DEPRECATED
import { useFormState } from 'react-dom';
const [state, formAction] = useFormState(myAction, initialState);

// ✅ CORRECT
import { useActionState } from 'react';
const [state, formAction] = useActionState(myAction, initialState);
```

**Creating wrapper functions that break progressive enhancement** → Pass the action directly to the form. Wrapping it in event handlers prevents the form from working without JavaScript.

```javascript
// ❌ WRONG - Breaks progressive enhancement
const [state, formAction] = useActionState(myAction, {});
return (
  <form onSubmit={(e) => {
    e.preventDefault();
    formAction(new FormData(e.target));
  }}>...</form>
);

// ✅ CORRECT - Progressive enhancement works
const [state, formAction] = useActionState(myAction, {});
return <form action={formAction}>...</form>;
```

**Client components using inline "use server" from imports** → Client components can only import module-level server actions, not functions with inline `'use server'` directives from other modules.

```javascript
// ❌ WRONG - Client component trying to import inline server action
'use client'
import { inlineServerAction } from './serverComponent'; // Won't work

// ✅ CORRECT - Import from dedicated actions file
'use client'
import { myAction } from './actions'; // actions.js has 'use server' at top
```

## Examples

### Example 1: Login Form with Validation

**Server action with validation:**

```javascript
// app/actions.js
'use server'

import { z } from 'zod';
import { signIn } from '@/lib/auth';

const loginSchema = z.object({
  email: z.string().email('Invalid email address'),
  password: z.string().min(6, 'Password must be at least 6 characters')
});

export async function login(prevState, formData) {
  const validatedFields = loginSchema.safeParse({
    email: formData.get('email'),
    password: formData.get('password')
  });
  
  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors
    };
  }
  
  try {
    await signIn(validatedFields.data);
    return { success: true };
  } catch (error) {
    return { 
      message: 'Invalid credentials. Please try again.' 
    };
  }
}
```

**Client form component:**

```javascript
// app/login/page.js
'use client'

import { useActionState } from 'react';
import { login } from '@/app/actions';

export default function LoginPage() {
  const [state, formAction, isPending] = useActionState(login, {});
  
  return (
    <form action={formAction} className="login-form">
      <h1>Login</h1>
      
      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          name="email"
          type="email"
          required
          aria-describedby="email-error"
        />
        {state.errors?.email && (
          <p id="email-error" className="error">
            {state.errors.email[0]}
          </p>
        )}
      </div>
      
      <div>
        <label htmlFor="password">Password</label>
        <input
          id="password"
          name="password"
          type="password"
          required
          aria-describedby="password-error"
        />
        {state.errors?.password && (
          <p id="password-error" className="error">
            {state.errors.password[0]}
          </p>
        )}
      </div>
      
      {state.message && (
        <p className="error" role="alert">{state.message}</p>
      )}
      
      <button type="submit" disabled={isPending}>
        {isPending ? 'Logging in...' : 'Log in'}
      </button>
    </form>
  );
}
```

### Example 2: Post Creation Form with useFormStatus

**Server action:**

```javascript
// app/actions.js
'use server'

import { revalidatePath } from 'next/cache';
import { z } from 'zod';

const postSchema = z.object({
  title: z.string().min(1, 'Title is required').max(100),
  content: z.string().min(10, 'Content must be at least 10 characters')
});

export async function createPost(prevState, formData) {
  const validatedFields = postSchema.safeParse({
    title: formData.get('title'),
    content: formData.get('content')
  });
  
  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
      fields: {
        title: formData.get('title'),
        content: formData.get('content')
      }
    };
  }
  
  try {
    const post = await db.posts.create({
      data: validatedFields.data
    });
    
    revalidatePath('/posts');
    return { 
      success: true, 
      postId: post.id,
      fields: {} 
    };
  } catch (error) {
    return { 
      message: 'Failed to create post. Please try again.',
      fields: validatedFields.data
    };
  }
}
```

**Client form with useFormStatus:**

```javascript
// app/posts/new/page.js
'use client'

import { useActionState } from 'react';
import { useFormStatus } from 'react-dom/hooks/useFormStatus';
import { createPost } from '@/app/actions';

function SubmitButton() {
  const { pending } = useFormStatus();
  
  return (
    <button type="submit" disabled={pending}>
      {pending ? (
        <>
          <span className="spinner" />
          Creating post...
        </>
      ) : (
        'Create Post'
      )}
    </button>
  );
}

export default function NewPostPage() {
  const [state, formAction] = useActionState(createPost, { fields: {} });
  
  if (state.success) {
    return (
      <div className="success">
        <h2>Post created successfully!</h2>
        <a href={`/posts/${state.postId}`}>View your post</a>
      </div>
    );
  }
  
  return (
    <form action={formAction}>
      <h1>Create New Post</h1>
      
      <div>
        <label htmlFor="title">Title</label>
        <input
          id="title"
          name="title"
          type="text"
          defaultValue={state.fields.title || ''}
          aria-describedby="title-error"
        />
        {state.errors?.title && (
          <p id="title-error" className="error">
            {state.errors.title[0]}
          </p>
        )}
      </div>
      
      <div>
        <label htmlFor="content">Content</label>
        <textarea
          id="content"
          name="content"
          rows={10}
          defaultValue={state.fields.content || ''}
          aria-describedby="content-error"
        />
        {state.errors?.content && (
          <p id="content-error" className="error">
            {state.errors.content[0]}
          </p>
        )}
      </div>
      
      {state.message && (
        <p className="error" role="alert">{state.message}</p>
      )}
      
      <SubmitButton />
    </form>
  );
}
```

## Sources

- [React useActionState Documentation](https://react.dev/reference/react/useActionState)
- [React v19 Release Blog](https://react.dev/blog/2024/12/05/react-19)
- [React useFormStatus Documentation](https://react.dev/reference/react-dom/hooks/useFormStatus)
- [React Server Functions](https://react.dev/reference/rsc/server-functions)
- [React "use server" Directive](https://react.dev/reference/rsc/use-server)
- [Next.js Forms Guide](https://nextjs.org/docs/app/guides/forms)
- [Next.js Server Actions and Mutations](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations)