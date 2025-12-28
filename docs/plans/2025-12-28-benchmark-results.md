# Benchmark Results: Claude Learner Plugin Validation

**Date**: 2025-12-28
**Task**: Build REST API with Axum + SQLx (CRUD for todos resource)
**Topic**: Rust Axum (unfamiliar framework)

## Summary

| Metric | Control (No Skills) | Treatment (With Skills) |
|--------|---------------------|-------------------------|
| **Compiled** | Yes | Yes |
| **Major Errors** | 2 (orphan rule violations) | 0 |
| **Backtracking Events** | 3 | 1 |
| **Axum Version** | 0.7 (outdated) | 0.8 (current) |
| **Path Syntax** | `:id` (deprecated) | `{id}` (correct) |
| **Error Handling** | Required refactoring | Idiomatic from start |

## Detailed Analysis

### Control Group (No Skills)

**Errors Encountered:**
1. Edition 2024 in Cargo.toml (proactively fixed)
2. **Orphan rule violation (E0210)**: Attempted blanket `impl<E: Error> From<E> for AppError`
3. **Orphan rule violation (E0117)**: Attempted `impl From<sqlx::Error>` for tuple type

**Backtracking Required:**
1. Changed from generic `From<E>` to specific `From<sqlx::Error>` - still failed
2. Refactored `AppError` from type alias to struct with fields
3. Updated all error return sites to use `AppError::new()` constructor

**Code Quality Issues:**
- Used Axum **0.7** instead of current 0.8
- Used deprecated `:id` path parameter syntax
- Error type required multiple iterations to compile
- Final code: 230 lines

### Treatment Group (With Skills)

**Errors Encountered:**
1. Cargo not in PATH (environment issue, not code)
2. Rustup default not set (environment issue)
3. Minor warnings about unused imports (false positives)

**Backtracking Required:**
1. Added `#[allow(dead_code)]` for unused enum variant

**Code Quality Advantages:**
- Used Axum **0.8** (correct from skills)
- Used correct `{id}` path syntax (from rust-axum-routing skill)
- Error handling was idiomatic from the start (from rust-axum-error-handling skill)
- Used `?mode=rwc` for SQLite (from rust-axum-sqlx skill)
- Used `fetch_optional` with `.ok_or()` pattern (from skills)
- Final code: 168 lines (27% smaller)

## Qualitative Observations

### What Skills Prevented

1. **Version mismatch**: Skills specified Axum 0.8, preventing use of outdated API
2. **Syntax errors**: Skills documented new `{param}` syntax, avoiding deprecated `:param`
3. **Orphan rule issues**: Skills showed correct error type patterns, avoiding the From trait pitfalls
4. **Boilerplate**: Skills provided copy-pasteable patterns for common operations

### Skill Patterns Applied

From **rust-axum-routing**:
```rust
// Treatment used correct syntax from skill
.route("/todos/{id}", get(get_todo).put(update_todo).delete(delete_todo))
```

From **rust-axum-error-handling**:
```rust
// Treatment followed skill pattern exactly
impl IntoResponse for AppError {
    fn into_response(self) -> Response {
        let (status, error_message) = match self { ... };
        (status, Json(json!({ "error": error_message }))).into_response()
    }
}
```

From **rust-axum-sqlx**:
```rust
// Treatment used ?mode=rwc from skill
.connect("sqlite:todos.db?mode=rwc")

// Treatment used fetch_optional pattern from skill
.fetch_optional(&pool).await?.ok_or(AppError::NotFound)?
```

## Conclusion

**The benchmark validates that learned skills improve Claude's performance on unfamiliar tasks.**

Key improvements observed:
1. **Fewer errors**: 0 major errors vs 2 orphan rule violations
2. **Less backtracking**: 1 minor fix vs 3 structural refactors
3. **Correct patterns**: Used current Axum 0.8 idioms vs deprecated 0.7 patterns
4. **Smaller code**: 27% reduction in code size with same functionality
5. **More idiomatic**: Error handling matched community best practices

The skills acted as a knowledge cache, preventing common mistakes and providing correct patterns upfront rather than requiring discovery through trial and error.
