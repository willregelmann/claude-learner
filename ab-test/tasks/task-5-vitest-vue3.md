Write comprehensive Vitest tests for a Vue 3 composable called `useFetch(url: string)` that returns `{ data: Ref<T | null>, error: Ref<Error | null>, loading: Ref<boolean>, refetch: () => Promise<void> }`.

Test cases needed:
1. Returns data on successful fetch
2. Sets error ref on failed fetch
3. loading is true during fetch and false after completion
4. refetch() triggers a new network request and updates data
5. Aborts the pending request when the component unmounts

Mock the global fetch API properly. Include both the composable implementation and the complete test file with all imports. Ensure proper test isolation (no leaking state between tests).