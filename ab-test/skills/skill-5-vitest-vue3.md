---
name: vitest-vue3-composable-testing
description: Use when writing Vitest tests for Vue 3 composables, mocking APIs, testing async composition functions, or ensuring test isolation.
---

# Testing Vue 3 Composables with Vitest

## When This Skill Applies

- Writing unit tests for Vue 3 composition API functions (composables)
- Testing composables that use lifecycle hooks (onMounted, onUnmounted, onBeforeUnmount)
- Testing composables with async operations, timers, or fetch calls
- Mocking global APIs (fetch, setTimeout, setInterval) in composable tests
- Preventing state leakage and ensuring proper test isolation
- Testing composables that use Vue's reactivity system (ref, reactive, computed, watch)
- Debugging composable test failures related to lifecycle, cleanup, or timing
- Setting up Vitest configuration for Vue 3 projects

## Key Patterns

### Initial Setup and Configuration

Install the required dependencies for testing Vue 3 composables with Vitest:

```bash
npm install -D vitest @vue/test-utils happy-dom
npm install -D @types/node  # For TypeScript projects
```

Configure Vitest in `vitest.config.ts` or `vite.config.ts`:

```typescript
import { defineConfig } from 'vitest/config'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  test: {
    globals: true,
    environment: 'happy-dom', // or 'jsdom'
  },
})
```

For TypeScript projects, update `tsconfig.json` to include Vitest types:

```json
{
  "compilerOptions": {
    "types": ["vitest/globals"]
  }
}
```

### Testing Independent Composables (No Lifecycle Hooks)

Composables that only use reactivity APIs (ref, reactive, computed, watch) without lifecycle hooks can be tested directly without mounting components:

```typescript
import { describe, it, expect } from 'vitest'
import { useCounter } from '@/composables/useCounter'

describe('useCounter', () => {
  it('increments count', () => {
    const { count, increment } = useCounter()
    
    expect(count.value).toBe(0)
    increment()
    expect(count.value).toBe(1)
  })
  
  it('decrements count', () => {
    const { count, decrement } = useCounter(10)
    
    expect(count.value).toBe(10)
    decrement()
    expect(count.value).toBe(9)
  })
})
```

### Testing Lifecycle-Dependent Composables with withSetup Helper

Composables using lifecycle hooks (onMounted, onUnmounted, etc.) require a Vue component context. Create a `withSetup` helper to mount a temporary component:

```typescript
import { createApp } from 'vue'

function withSetup<T>(composable: () => T): [T, any] {
  let result: T
  const app = createApp({
    setup() {
      result = composable()
      // Suppress missing template warning
      return () => {}
    }
  })
  app.mount(document.createElement('div'))
  // @ts-ignore: result will be set during setup
  return [result, app]
}

// Usage in tests
describe('useFetchData', () => {
  it('fetches data on mount', async () => {
    const [result, app] = withSetup(() => useFetchData('/api/users'))
    
    await flushPromises()
    
    expect(result.data.value).toBeDefined()
    expect(result.loading.value).toBe(false)
    
    // Always cleanup
    app.unmount()
  })
})
```

### Handling Async Operations with flushPromises

Use `flushPromises` from `@vue/test-utils` to wait for all pending promises to resolve before assertions:

```typescript
import { flushPromises } from '@vue/test-utils'
import { describe, it, expect, beforeEach } from 'vitest'

describe('useAsyncData', () => {
  it('loads data asynchronously', async () => {
    const [result, app] = withSetup(() => useAsyncData())
    
    // Initial state
    expect(result.loading.value).toBe(true)
    expect(result.data.value).toBeNull()
    
    // Wait for all promises to resolve
    await flushPromises()
    
    expect(result.loading.value).toBe(false)
    expect(result.data.value).toEqual({ name: 'Test' })
    
    app.unmount()
  })
})
```

Do not rely on `nextTick()` alone for async operations. Vue's `nextTick()` only waits for the next DOM update cycle, not for async callbacks or promises to complete. Always use `flushPromises()` when testing async composables.

### Mocking Global Fetch API

Mock the global fetch function using Vitest's `vi.fn()`:

```typescript
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest'

describe('useFetch', () => {
  beforeEach(() => {
    // Mock global fetch
    global.fetch = vi.fn()
  })
  
  afterEach(() => {
    // Restore original implementation
    vi.restoreAllMocks()
  })
  
  it('fetches data successfully', async () => {
    const mockResponse = {
      ok: true,
      json: async () => ({ id: 1, name: 'Test User' }),
    }
    
    global.fetch.mockResolvedValue(mockResponse)
    
    const [result, app] = withSetup(() => useFetch('/api/user'))
    
    await flushPromises()
    
    expect(global.fetch).toHaveBeenCalledWith('/api/user')
    expect(result.data.value).toEqual({ id: 1, name: 'Test User' })
    
    app.unmount()
  })
  
  it('handles fetch errors', async () => {
    global.fetch.mockRejectedValue(new Error('Network error'))
    
    const [result, app] = withSetup(() => useFetch('/api/user'))
    
    await flushPromises()
    
    expect(result.error.value).toBe('Network error')
    expect(result.data.value).toBeNull()
    
    app.unmount()
  })
})
```

### Mocking Timers (setTimeout, setInterval)

Use Vitest's fake timers to control time-dependent code:

```typescript
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest'

describe('useDebounce', () => {
  beforeEach(() => {
    vi.useFakeTimers()
  })
  
  afterEach(() => {
    vi.restoreAllMocks()
  })
  
  it('debounces function calls', () => {
    const callback = vi.fn()
    const [result, app] = withSetup(() => useDebounce(callback, 300))
    
    result.debouncedFn()
    result.debouncedFn()
    result.debouncedFn()
    
    expect(callback).not.toHaveBeenCalled()
    
    // Advance timers by 300ms
    vi.advanceTimersByTime(300)
    
    expect(callback).toHaveBeenCalledTimes(1)
    
    app.unmount()
  })
  
  it('runs all pending timers', () => {
    const callback = vi.fn()
    const [result, app] = withSetup(() => useInterval(callback, 1000))
    
    vi.advanceTimersByTime(3500)
    
    expect(callback).toHaveBeenCalledTimes(3)
    
    app.unmount()
    vi.runOnlyPendingTimers() // Alternative to advanceTimersByTime
  })
})
```

Available timer control methods:
- `vi.useFakeTimers()`: Enable fake timers
- `vi.advanceTimersByTime(ms)`: Advance timers by specified milliseconds
- `vi.runAllTimers()`: Run all pending timers immediately
- `vi.advanceTimersToNextTimer()`: Advance to the next scheduled timer
- `vi.runOnlyPendingTimers()`: Run only currently pending timers
- `vi.restoreAllMocks()`: Restore real timers

### Testing Cleanup and AbortController Patterns

Composables that create side effects (event listeners, intervals, fetch requests) must clean up properly. Test that cleanup happens correctly:

```typescript
describe('useEventListener', () => {
  it('adds event listener on mount and removes on unmount', () => {
    const handler = vi.fn()
    const addEventListener = vi.spyOn(window, 'addEventListener')
    const removeEventListener = vi.spyOn(window, 'removeEventListener')
    
    const [result, app] = withSetup(() => 
      useEventListener('resize', handler)
    )
    
    expect(addEventListener).toHaveBeenCalledWith('resize', expect.any(Function))
    
    window.dispatchEvent(new Event('resize'))
    expect(handler).toHaveBeenCalledTimes(1)
    
    // Test cleanup
    app.unmount()
    expect(removeEventListener).toHaveBeenCalledWith('resize', expect.any(Function))
    
    addEventListener.mockRestore()
    removeEventListener.mockRestore()
  })
})

describe('useFetchWithAbort', () => {
  beforeEach(() => {
    global.fetch = vi.fn()
  })
  
  it('aborts fetch on unmount', async () => {
    const abortSpy = vi.fn()
    const mockAbort = vi.spyOn(AbortController.prototype, 'abort')
      .mockImplementation(abortSpy)
    
    global.fetch.mockImplementation(() => 
      new Promise(resolve => setTimeout(resolve, 1000))
    )
    
    const [result, app] = withSetup(() => useFetchWithAbort('/api/data'))
    
    // Unmount before fetch completes
    app.unmount()
    
    expect(abortSpy).toHaveBeenCalled()
    
    mockAbort.mockRestore()
  })
})
```

### Ensuring Test Isolation and Preventing State Leakage

Always unmount components after each test to prevent memory leaks and state pollution:

```typescript
describe('useSharedState', () => {
  let app: any
  
  afterEach(() => {
    // Always cleanup after each test
    if (app) {
      app.unmount()
      app = null
    }
  })
  
  it('maintains independent state between tests', () => {
    const [result1, app1] = withSetup(() => useSharedState())
    result1.count.value = 5
    app1.unmount()
    
    const [result2, app2] = withSetup(() => useSharedState())
    
    // State should be fresh, not leaked from previous test
    expect(result2.count.value).toBe(0)
    
    app = app2
  })
})
```

For composables with module-level state, reset state between tests:

```typescript
// In composable
let sharedState = ref(0)

export function useSharedCounter() {
  return {
    count: sharedState,
    reset: () => { sharedState.value = 0 }
  }
}

// In test
describe('useSharedCounter', () => {
  afterEach(() => {
    const [{ reset }, app] = withSetup(() => useSharedCounter())
    reset()
    app.unmount()
  })
  
  it('test 1', () => {
    const [result, app] = withSetup(() => useSharedCounter())
    result.count.value = 10
    expect(result.count.value).toBe(10)
    app.unmount()
  })
  
  it('test 2 has clean state', () => {
    const [result, app] = withSetup(() => useSharedCounter())
    expect(result.count.value).toBe(0) // Reset was called in afterEach
    app.unmount()
  })
})
```

### Testing Watchers and Reactive Dependencies

Test that composables properly react to changes in dependencies:

```typescript
describe('useFilteredList', () => {
  it('updates filtered list when filter changes', async () => {
    const items = ref([
      { id: 1, name: 'Apple' },
      { id: 2, name: 'Banana' },
      { id: 3, name: 'Avocado' }
    ])
    
    const [result, app] = withSetup(() => useFilteredList(items))
    
    expect(result.filtered.value).toHaveLength(3)
    
    result.filter.value = 'A'
    await nextTick() // Wait for watchers to run
    
    expect(result.filtered.value).toHaveLength(2)
    expect(result.filtered.value[0].name).toBe('Apple')
    
    app.unmount()
  })
})
```

### Mocking Composables in Component Tests

When testing components that use composables, mock the composable to isolate component logic:

```typescript
import { vi } from 'vitest'

vi.mock('@/composables/useAuth', () => ({
  useAuth: vi.fn(() => ({
    user: ref({ id: 1, name: 'Test User' }),
    isAuthenticated: ref(true),
    login: vi.fn(),
    logout: vi.fn()
  }))
}))

describe('UserProfile', () => {
  it('displays user name when authenticated', () => {
    const wrapper = mount(UserProfile)
    expect(wrapper.text()).toContain('Test User')
  })
})
```

## Common Mistakes to Avoid

**Mistake:** Testing lifecycle-dependent composables without a component context.
```typescript
// WRONG: This will fail if composable uses onMounted
const result = useDataFetcher()
```
**Correction:** Use the withSetup helper to create a proper Vue component context.
```typescript
const [result, app] = withSetup(() => useDataFetcher())
// ... assertions
app.unmount()
```

**Mistake:** Not awaiting async operations before making assertions.
```typescript
// WRONG: Assertion runs before promise resolves
const [result, app] = withSetup(() => useAsyncData())
expect(result.data.value).toBeDefined() // Likely still null
```
**Correction:** Always use flushPromises() for async composables.
```typescript
const [result, app] = withSetup(() => useAsyncData())
await flushPromises()
expect(result.data.value).toBeDefined()
```

**Mistake:** Using nextTick() instead of flushPromises() for async operations.
```typescript
// WRONG: nextTick only waits for DOM updates, not promises
await nextTick()
expect(result.data.value).toBeDefined()
```
**Correction:** Use flushPromises() for promise-based async operations.
```typescript
await flushPromises()
expect(result.data.value).toBeDefined()
```

**Mistake:** Forgetting to unmount the test app after each test.
```typescript
// WRONG: Memory leak and state pollution
it('test 1', () => {
  const [result, app] = withSetup(() => useCounter())
  expect(result.count.value).toBe(0)
  // Missing app.unmount()
})
```
**Correction:** Always unmount in afterEach or at the end of each test.
```typescript
let app: any

afterEach(() => {
  if (app) app.unmount()
})

it('test 1', () => {
  const [result, _app] = withSetup(() => useCounter())
  app = _app
  expect(result.count.value).toBe(0)
})
```

**Mistake:** Not restoring mocks after tests, causing interference between tests.
```typescript
// WRONG: Mock persists across tests
beforeEach(() => {
  global.fetch = vi.fn()
})
```
**Correction:** Always restore mocks in afterEach.
```typescript
beforeEach(() => {
  global.fetch = vi.fn()
})

afterEach(() => {
  vi.restoreAllMocks()
})
```

**Mistake:** Not testing cleanup behavior for composables with side effects.
```typescript
// WRONG: Only testing happy path, not cleanup
it('sets up interval', () => {
  const [result, app] = withSetup(() => useInterval(callback, 1000))
  expect(callback).toHaveBeenCalled()
})
```
**Correction:** Explicitly test that cleanup happens on unmount.
```typescript
it('clears interval on unmount', () => {
  const clearIntervalSpy = vi.spyOn(global, 'clearInterval')
  const [result, app] = withSetup(() => useInterval(callback, 1000))
  
  app.unmount()
  
  expect(clearIntervalSpy).toHaveBeenCalled()
})
```

**Mistake:** Mocking fetch without providing complete Response interface.
```typescript
// WRONG: Missing required Response properties
global.fetch = vi.fn().mockResolvedValue({ 
  json: async () => ({ data: 'test' })
})
```
**Correction:** Include ok, status, and other Response properties.
```typescript
global.fetch = vi.fn().mockResolvedValue({
  ok: true,
  status: 200,
  json: async () => ({ data: 'test' }),
  text: async () => JSON.stringify({ data: 'test' })
})
```

**Mistake:** Not handling module-level state reset between tests.
```typescript
// In composable: const cache = ref({})
// WRONG: Cache persists across tests
it('test 1', () => {
  const [result, app] = withSetup(() => useCache())
  result.set('key', 'value1')
})

it('test 2', () => {
  const [result, app] = withSetup(() => useCache())
  // Cache still contains 'key' from test 1
})
```
**Correction:** Provide reset method and call it in afterEach.
```typescript
export function useCache() {
  return {
    get, set,
    reset: () => { cache.value = {} }
  }
}

afterEach(() => {
  const [{ reset }, app] = withSetup(() => useCache())
  reset()
  app.unmount()
})
```

## Examples

### Example 1: Testing a Composable with Fetch and Cleanup

```typescript
// composables/useUserData.ts
import { ref, onUnmounted } from 'vue'

export function useUserData(userId: string) {
  const data = ref(null)
  const loading = ref(true)
  const error = ref(null)
  const controller = new AbortController()
  
  async function fetchUser() {
    try {
      loading.value = true
      const response = await fetch(`/api/users/${userId}`, {
        signal: controller.signal
      })
      if (!response.ok) throw new Error('Failed to fetch')
      data.value = await response.json()
    } catch (err: any) {
      if (err.name !== 'AbortError') {
        error.value = err.message
      }
    } finally {
      loading.value = false
    }
  }
  
  onMounted(() => {
    fetchUser()
  })
  
  onUnmounted(() => {
    controller.abort()
  })
  
  return { data, loading, error, refetch: fetchUser }
}

// __tests__/useUserData.test.ts
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest'
import { flushPromises } from '@vue/test-utils'
import { useUserData } from '@/composables/useUserData'

function withSetup(composable) {
  let result
  const app = createApp({
    setup() {
      result = composable()
      return () => {}
    }
  })
  app.mount(document.createElement('div'))
  return [result, app]
}

describe('useUserData', () => {
  beforeEach(() => {
    global.fetch = vi.fn()
  })
  
  afterEach(() => {
    vi.restoreAllMocks()
  })
  
  it('fetches user data on mount', async () => {
    global.fetch.mockResolvedValue({
      ok: true,
      json: async () => ({ id: '123', name: 'John Doe' })
    })
    
    const [result, app] = withSetup(() => useUserData('123'))
    
    expect(result.loading.value).toBe(true)
    
    await flushPromises()
    
    expect(global.fetch).toHaveBeenCalledWith(
      '/api/users/123',
      expect.objectContaining({ signal: expect.any(AbortSignal) })
    )
    expect(result.data.value).toEqual({ id: '123', name: 'John Doe' })
    expect(result.loading.value).toBe(false)
    
    app.unmount()
  })
  
  it('aborts fetch on unmount', async () => {
    const abortSpy = vi.spyOn(AbortController.prototype, 'abort')
    
    global.fetch.mockImplementation(() => 
      new Promise(resolve => setTimeout(resolve, 5000))
    )
    
    const [result, app] = withSetup(() => useUserData('123'))
    
    app.unmount()
    
    expect(abortSpy).toHaveBeenCalled()
    abortSpy.mockRestore()
  })
})
```

### Example 2: Testing a Composable with Timers and State Management

```typescript
// composables/usePolling.ts
import { ref, onUnmounted } from 'vue'

export function usePolling(callback: () => void, interval: number) {
  const isActive = ref(false)
  let timerId: number | null = null
  
  function start() {
    if (isActive.value) return
    isActive.value = true
    timerId = setInterval(callback, interval)
  }
  
  function stop() {
    if (timerId !== null) {
      clearInterval(timerId)
      timerId = null
    }
    isActive.value = false
  }
  
  onUnmounted(() => {
    stop()
  })
  
  return { isActive, start, stop }
}

// __tests__/usePolling.test.ts
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest'
import { usePolling } from '@/composables/usePolling'

describe('usePolling', () => {
  beforeEach(() => {
    vi.useFakeTimers()
  })
  
  afterEach(() => {
    vi.restoreAllMocks()
  })
  
  it('calls callback at specified interval', () => {
    const callback = vi.fn()
    const [result, app] = withSetup(() => usePolling(callback, 1000))
    
    result.start()
    expect(result.isActive.value).toBe(true)
    
    vi.advanceTimersByTime(3500)
    
    expect(callback).toHaveBeenCalledTimes(3)
    
    result.stop()
    app.unmount()
  })
  
  it('stops polling when stopped', () => {
    const callback = vi.fn()
    const [result, app] = withSetup(() => usePolling(callback, 1000))
    
    result.start()
    vi.advanceTimersByTime(2000)
    
    result.stop()
    expect(result.isActive.value).toBe(false)
    
    vi.advanceTimersByTime(2000)
    
    // Should still be 2, not 4
    expect(callback).toHaveBeenCalledTimes(2)
    
    app.unmount()
  })
  
  it('cleans up interval on unmount', () => {
    const clearIntervalSpy = vi.spyOn(global, 'clearInterval')
    const callback = vi.fn()
    const [result, app] = withSetup(() => usePolling(callback, 1000))
    
    result.start()
    app.unmount()
    
    expect(clearIntervalSpy).toHaveBeenCalled()
    clearIntervalSpy.mockRestore()
  })
})
```

Sources:
- [Mastering Vue 3 Composables Testing with Vitest](https://dylanbritz.dev/writing/testing-vue-composables-lifecycle/)
- [How to Test Vue Composables: A Comprehensive Guide with Vitest](https://alexop.dev/posts/how-to-test-vue-composables/)
- [Testing Vue.js | Official Vue Documentation](https://vuejs.org/guide/scaling-up/testing)
- [Mocking | Vitest Guide](https://vitest.dev/guide/mocking)
- [Vitest Timers Documentation](https://vitest.dev/guide/mocking/timers)
- [Vue Test Utils - Reusability & Composition](https://test-utils.vuejs.org/guide/advanced/reusability-composition)
```
Shell cwd was reset to /Users/willpersonal/Projects/claude-learner