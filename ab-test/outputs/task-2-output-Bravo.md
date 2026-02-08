```typescript
// app/actions/register.ts
'use server'

type FormState = {
  errors?: {
    name?: string[]
    email?: string[]
    password?: string[]
  }
  message?: string
  success?: boolean
}

function validateEmail(email: string): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  return emailRegex.test(email)
}

export async function registerUser(
  prevState: FormState,
  formData: FormData
): Promise<FormState> {
  // Simulate network delay
  await new Promise(resolve => setTimeout(resolve, 1000))

  const name = formData.get('name') as string
  const email = formData.get('email') as string
  const password = formData.get('password') as string

  const errors: FormState['errors'] = {}

  // Validation
  if (!name || name.trim().length === 0) {
    errors.name = ['Name is required']
  }

  if (!email || email.trim().length === 0) {
    errors.email = ['Email is required']
  } else if (!validateEmail(email)) {
    errors.email = ['Please enter a valid email address']
  }

  if (!password || password.length === 0) {
    errors.password = ['Password is required']
  } else if (password.length < 8) {
    errors.password = ['Password must be at least 8 characters long']
  }

  // Return errors if validation fails
  if (Object.keys(errors).length > 0) {
    return {
      errors,
      success: false
    }
  }

  // Simulate successful registration
  // In a real app, you would save to database here
  console.log('Registering user:', { name, email })

  return {
    message: 'Registration successful! Welcome aboard.',
    success: true
  }
}
```

```typescript
// app/components/RegisterForm.tsx
'use client'

import { useActionState } from 'react'
import { registerUser } from '../actions/register'

const initialState = {
  errors: {},
  message: '',
  success: false
}

export default function RegisterForm() {
  const [state, formAction, isPending] = useActionState(registerUser, initialState)

  return (
    <div className="max-w-md mx-auto mt-8 p-6 bg-white rounded-lg shadow-md">
      <h2 className="text-2xl font-bold mb-6 text-gray-800">Register</h2>
      
      {state.success && state.message && (
        <div className="mb-4 p-4 bg-green-100 border border-green-400 text-green-700 rounded">
          {state.message}
        </div>
      )}

      <form action={formAction} className="space-y-4">
        <div>
          <label htmlFor="name" className="block text-sm font-medium text-gray-700 mb-1">
            Name
          </label>
          <input
            type="text"
            id="name"
            name="name"
            disabled={isPending}
            className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 disabled:bg-gray-100 disabled:cursor-not-allowed"
            aria-describedby={state.errors?.name ? 'name-error' : undefined}
          />
          {state.errors?.name && (
            <p id="name-error" className="mt-1 text-sm text-red-600">
              {state.errors.name[0]}
            </p>
          )}
        </div>

        <div>
          <label htmlFor="email" className="block text-sm font-medium text-gray-700 mb-1">
            Email
          </label>
          <input
            type="email"
            id="email"
            name="email"
            disabled={isPending}
            className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 disabled:bg-gray-100 disabled:cursor-not-allowed"
            aria-describedby={state.errors?.email ? 'email-error' : undefined}
          />
          {state.errors?.email && (
            <p id="email-error" className="mt-1 text-sm text-red-600">
              {state.errors.email[0]}
            </p>
          )}
        </div>

        <div>
          <label htmlFor="password" className="block text-sm font-medium text-gray-700 mb-1">
            Password
          </label>
          <input
            type="password"
            id="password"
            name="password"
            disabled={isPending}
            className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 disabled:bg-gray-100 disabled:cursor-not-allowed"
            aria-describedby={state.errors?.password ? 'password-error' : undefined}
          />
          {state.errors?.password && (
            <p id="password-error" className="mt-1 text-sm text-red-600">
              {state.errors.password[0]}
            </p>
          )}
        </div>

        <button
          type="submit"
          disabled={isPending}
          className="w-full bg-blue-600 text-white py-2 px-4 rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 disabled:bg-blue-400 disabled:cursor-not-allowed transition-colors"
        >
          {isPending ? 'Registering...' : 'Register'}
        </button>
      </form>
    </div>
  )
}
```

```typescript
// app/page.tsx
import RegisterForm from './components/RegisterForm'

export default function Page() {
  return (
    <main className="min-h-screen bg-gray-50 py-12">
      <RegisterForm />
    </main>
  )
}
```