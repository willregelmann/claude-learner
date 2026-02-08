Now I'll create the composable implementation and comprehensive test file following the patterns from the reference guide.

---

## useFetch.ts

```typescript
import { ref, onUnmounted, type Ref } from 'vue'

interface UseFetchReturn<T> {
  data: Ref<T | null>
  error: Ref<Error | null>
  loading: Ref<boolean>
  refetch: () => Promise<void>
}

export function useFetch<T = any>(url: string): UseFetchReturn<T> {
  const data = ref<T | null>(null) as Ref<T | null>
  const error = ref<Error | null>(null)
  const loading = ref(false)
  const controller = new AbortController()

  async function fetchData() {
    loading.value = true
    error.value = null

    try {
      const response = await fetch(url, {
        signal: controller.signal
      })

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`)
      }

      const result = await response.json()
      data.value = result
    } catch (err: any) {
      if (err.name !== 'AbortError') {
        error.value = err
      }
    } finally {
      loading.value = false
    }
  }

  onUnmounted(() => {
    controller.abort()
  })

  return {
    data,
    error,
    loading,
    refetch: fetchData
  }
}
```

---

## useFetch.test.ts

```typescript
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest'
import { flushPromises } from '@vue/test-utils'
import { createApp } from 'vue'
import { useFetch } from './useFetch'

function withSetup<T>(composable: () => T): [T, any] {
  let result: T
  const app = createApp({
    setup() {
      result = composable()
      return () => {}
    }
  })
  app.mount(document.createElement('div'))
  // @ts-ignore: result will be set during setup
  return [result, app]
}

describe('useFetch', () => {
  let app: any

  beforeEach(() => {
    global.fetch = vi.fn()
  })

  afterEach(() => {
    if (app) {
      app.unmount()
      app = null
    }
    vi.restoreAllMocks()
  })

  it('returns data on successful fetch', async () => {
    const mockData = { id: 1, name: 'Test User' }
    
    global.fetch = vi.fn().mockResolvedValue({
      ok: true,
      status: 200,
      json: async () => mockData
    })

    const [result, testApp] = withSetup(() => useFetch('/api/user'))
    app = testApp

    expect(result.data.value).toBeNull()
    expect(result.loading.value).toBe(false)

    await result.refetch()
    await flushPromises()

    expect(global.fetch).toHaveBeenCalledWith(
      '/api/user',
      expect.objectContaining({ signal: expect.any(AbortSignal) })
    )
    expect(result.data.value).toEqual(mockData)
    expect(result.error.value).toBeNull()
    expect(result.loading.value).toBe(false)
  })

  it('sets error ref on failed fetch', async () => {
    global.fetch = vi.fn().mockRejectedValue(new Error('Network error'))

    const [result, testApp] = withSetup(() => useFetch('/api/user'))
    app = testApp

    await result.refetch()
    await flushPromises()

    expect(result.error.value).toBeInstanceOf(Error)
    expect(result.error.value?.message).toBe('Network error')
    expect(result.data.value).toBeNull()
    expect(result.loading.value).toBe(false)
  })

  it('sets error on HTTP error status', async () => {
    global.fetch = vi.fn().mockResolvedValue({
      ok: false,
      status: 404,
      json: async () => ({})
    })

    const [result, testApp] = withSetup(() => useFetch('/api/user'))
    app = testApp

    await result.refetch()
    await flushPromises()

    expect(result.error.value).toBeInstanceOf(Error)
    expect(result.error.value?.message).toContain('HTTP error! status: 404')
    expect(result.data.value).toBeNull()
  })

  it('loading is true during fetch and false after completion', async () => {
    let resolvePromise: (value: any) => void
    const fetchPromise = new Promise(resolve => {
      resolvePromise = resolve
    })

    global.fetch = vi.fn().mockReturnValue(fetchPromise)

    const [result, testApp] = withSetup(() => useFetch('/api/data'))
    app = testApp

    expect(result.loading.value).toBe(false)

    const refetchPromise = result.refetch()
    
    // Loading should be true immediately after refetch is called
    expect(result.loading.value).toBe(true)

    // Resolve the fetch
    resolvePromise!({
      ok: true,
      status: 200,
      json: async () => ({ data: 'test' })
    })

    await refetchPromise
    await flushPromises()

    // Loading should be false after completion
    expect(result.loading.value).toBe(false)
  })

  it('refetch triggers a new network request and updates data', async () => {
    const firstData = { id: 1, name: 'First' }
    const secondData = { id: 2, name: 'Second' }

    global.fetch = vi.fn()
      .mockResolvedValueOnce({
        ok: true,
        status: 200,
        json: async () => firstData
      })
      .mockResolvedValueOnce({
        ok: true,
        status: 200,
        json: async () => secondData
      })

    const [result, testApp] = withSetup(() => useFetch('/api/user'))
    app = testApp

    // First fetch
    await result.refetch()
    await flushPromises()

    expect(result.data.value).toEqual(firstData)
    expect(global.fetch).toHaveBeenCalledTimes(1)

    // Second fetch (refetch)
    await result.refetch()
    await flushPromises()

    expect(result.data.value).toEqual(secondData)
    expect(global.fetch).toHaveBeenCalledTimes(2)
    expect(global.fetch).toHaveBeenNthCalledWith(
      2,
      '/api/user',
      expect.objectContaining({ signal: expect.any(AbortSignal) })
    )
  })

  it('aborts the pending request when the component unmounts', async () => {
    const abortSpy = vi.spyOn(AbortController.prototype, 'abort')

    global.fetch = vi.fn().mockImplementation(() =>
      new Promise(resolve => setTimeout(resolve, 5000))
    )

    const [result, testApp] = withSetup(() => useFetch('/api/data'))

    // Start a fetch that won't complete
    result.refetch()

    // Unmount before fetch completes
    testApp.unmount()

    expect(abortSpy).toHaveBeenCalled()

    abortSpy.mockRestore()
    app = null // Already unmounted
  })

  it('does not set error when request is aborted', async () => {
    const abortError = new Error('Aborted')
    abortError.name = 'AbortError'

    global.fetch = vi.fn().mockRejectedValue(abortError)

    const [result, testApp] = withSetup(() => useFetch('/api/data'))
    app = testApp

    await result.refetch()
    await flushPromises()

    // AbortError should not be set as error
    expect(result.error.value).toBeNull()
  })

  it('clears error on successful refetch after failed fetch', async () => {
    const successData = { id: 1, name: 'Success' }

    global.fetch = vi.fn()
      .mockRejectedValueOnce(new Error('First fetch failed'))
      .mockResolvedValueOnce({
        ok: true,
        status: 200,
        json: async () => successData
      })

    const [result, testApp] = withSetup(() => useFetch('/api/user'))
    app = testApp

    // First fetch fails
    await result.refetch()
    await flushPromises()

    expect(result.error.value).toBeInstanceOf(Error)
    expect(result.data.value).toBeNull()

    // Second fetch succeeds
    await result.refetch()
    await flushPromises()

    expect(result.error.value).toBeNull()
    expect(result.data.value).toEqual(successData)
  })
})
```