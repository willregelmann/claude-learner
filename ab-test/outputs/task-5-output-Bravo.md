# useFetch Composable Implementation

## File: `src/composables/useFetch.ts`

```typescript
import { ref, onUnmounted, type Ref } from 'vue';

export interface UseFetchReturn<T> {
  data: Ref<T | null>;
  error: Ref<Error | null>;
  loading: Ref<boolean>;
  refetch: () => Promise<void>;
}

export function useFetch<T = unknown>(url: string): UseFetchReturn<T> {
  const data = ref<T | null>(null);
  const error = ref<Error | null>(null);
  const loading = ref<boolean>(false);
  
  let abortController: AbortController | null = null;

  const fetchData = async () => {
    // Cancel previous request if any
    if (abortController) {
      abortController.abort();
    }

    // Create new abort controller
    abortController = new AbortController();
    
    loading.value = true;
    error.value = null;

    try {
      const response = await fetch(url, {
        signal: abortController.signal,
      });

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const json = await response.json();
      data.value = json;
    } catch (err) {
      // Don't set error if request was aborted
      if (err instanceof Error && err.name !== 'AbortError') {
        error.value = err;
        data.value = null;
      }
    } finally {
      // Only set loading to false if request wasn't aborted
      if (abortController && !abortController.signal.aborted) {
        loading.value = false;
      }
    }
  };

  const refetch = async () => {
    await fetchData();
  };

  // Cleanup on unmount
  onUnmounted(() => {
    if (abortController) {
      abortController.abort();
    }
  });

  // Initial fetch
  fetchData();

  return {
    data,
    error,
    loading,
    refetch,
  };
}
```

## File: `src/composables/useFetch.test.ts`

```typescript
import { describe, it, expect, beforeEach, afterEach, vi } from 'vitest';
import { useFetch } from './useFetch';
import { nextTick } from 'vue';
import { mount } from '@vue/test-utils';
import { defineComponent, h } from 'vue';

describe('useFetch', () => {
  let originalFetch: typeof global.fetch;

  beforeEach(() => {
    // Store original fetch
    originalFetch = global.fetch;
    
    // Mock fetch
    global.fetch = vi.fn();
  });

  afterEach(() => {
    // Restore original fetch
    global.fetch = originalFetch;
    
    // Clear all mocks
    vi.clearAllMocks();
  });

  it('returns data on successful fetch', async () => {
    const mockData = { id: 1, name: 'Test User' };
    
    (global.fetch as any).mockResolvedValueOnce({
      ok: true,
      json: async () => mockData,
    });

    const wrapper = mount(
      defineComponent({
        setup() {
          return useFetch('https://api.example.com/users/1');
        },
        render() {
          return h('div');
        },
      })
    );

    const { data, error, loading } = wrapper.vm as any;

    // Initially loading
    expect(loading.value).toBe(true);
    expect(data.value).toBe(null);

    // Wait for fetch to complete
    await vi.waitFor(() => {
      expect(loading.value).toBe(false);
    });

    expect(data.value).toEqual(mockData);
    expect(error.value).toBe(null);
    expect(global.fetch).toHaveBeenCalledWith(
      'https://api.example.com/users/1',
      expect.objectContaining({
        signal: expect.any(AbortSignal),
      })
    );

    wrapper.unmount();
  });

  it('sets error ref on failed fetch', async () => {
    (global.fetch as any).mockResolvedValueOnce({
      ok: false,
      status: 404,
    });

    const wrapper = mount(
      defineComponent({
        setup() {
          return useFetch('https://api.example.com/users/999');
        },
        render() {
          return h('div');
        },
      })
    );

    const { data, error, loading } = wrapper.vm as any;

    // Wait for fetch to complete
    await vi.waitFor(() => {
      expect(loading.value).toBe(false);
    });

    expect(data.value).toBe(null);
    expect(error.value).toBeInstanceOf(Error);
    expect(error.value?.message).toContain('HTTP error! status: 404');

    wrapper.unmount();
  });

  it('loading is true during fetch and false after completion', async () => {
    const mockData = { id: 1, name: 'Test User' };
    
    // Create a promise we can control
    let resolvePromise: (value: any) => void;
    const fetchPromise = new Promise((resolve) => {
      resolvePromise = resolve;
    });

    (global.fetch as any).mockReturnValueOnce(fetchPromise);

    const wrapper = mount(
      defineComponent({
        setup() {
          return useFetch('https://api.example.com/users/1');
        },
        render() {
          return h('div');
        },
      })
    );

    const { loading } = wrapper.vm as any;

    // Should be loading initially
    await nextTick();
    expect(loading.value).toBe(true);

    // Resolve the fetch
    resolvePromise!({
      ok: true,
      json: async () => mockData,
    });

    // Wait for loading to become false
    await vi.waitFor(() => {
      expect(loading.value).toBe(false);
    });

    wrapper.unmount();
  });

  it('refetch() triggers a new network request and updates data', async () => {
    const mockData1 = { id: 1, name: 'User 1' };
    const mockData2 = { id: 1, name: 'User 1 Updated' };

    (global.fetch as any)
      .mockResolvedValueOnce({
        ok: true,
        json: async () => mockData1,
      })
      .mockResolvedValueOnce({
        ok: true,
        json: async () => mockData2,
      });

    const wrapper = mount(
      defineComponent({
        setup() {
          return useFetch('https://api.example.com/users/1');
        },
        render() {
          return h('div');
        },
      })
    );

    const { data, refetch, loading } = wrapper.vm as any;

    // Wait for initial fetch
    await vi.waitFor(() => {
      expect(loading.value).toBe(false);
    });

    expect(data.value).toEqual(mockData1);
    expect(global.fetch).toHaveBeenCalledTimes(1);

    // Call refetch
    await refetch();

    expect(data.value).toEqual(mockData2);
    expect(global.fetch).toHaveBeenCalledTimes(2);
    expect(loading.value).toBe(false);

    wrapper.unmount();
  });

  it('aborts the pending request when the component unmounts', async () => {
    let abortSignal: AbortSignal | undefined;
    
    (global.fetch as any).mockImplementation((url: string, options: any) => {
      abortSignal = options.signal;
      
      return new Promise((resolve) => {
        // Never resolve - simulate slow request
        setTimeout(() => {
          resolve({
            ok: true,
            json: async () => ({ data: 'test' }),
          });
        }, 10000);
      });
    });

    const wrapper = mount(
      defineComponent({
        setup() {
          return useFetch('https://api.example.com/users/1');
        },
        render() {
          return h('div');
        },
      })
    );

    const { loading } = wrapper.vm as any;

    await nextTick();
    
    // Should be loading
    expect(loading.value).toBe(true);
    expect(abortSignal).toBeDefined();
    expect(abortSignal?.aborted).toBe(false);

    // Unmount component
    wrapper.unmount();

    // Abort signal should be triggered
    expect(abortSignal?.aborted).toBe(true);
  });
});
```

## File: `vitest.config.ts`

```typescript
import { defineConfig } from 'vitest/config';
import vue from '@vitejs/plugin-vue';
import { fileURLToPath } from 'node:url';

export default defineConfig({
  plugins: [vue()],
  test: {
    globals: true,
    environment: 'jsdom',
  },
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url)),
    },
  },
});
```

## File: `package.json`

```json
{
  "name": "vue-use-fetch",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:run": "vitest run"
  },
  "devDependencies": {
    "@vitejs/plugin-vue": "^5.0.0",
    "@vue/test-utils": "^2.4.0",
    "jsdom": "^24.0.0",
    "vitest": "^1.3.0",
    "vue": "^3.4.0"
  },
  "dependencies": {
    "vue": "^3.4.0"
  }
}
```