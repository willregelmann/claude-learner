Write a deployment helper script using Bun's native $ shell API (the tagged template literal from Bun). The script should:

1. Check if the git working directory is clean (no uncommitted changes), abort if dirty.
2. Run the test suite (`bun test`) and abort if any tests fail.
3. Build the project (`bun run build`) and abort on failure.
4. Read the version from package.json and create a git tag (v{version}).
5. Push the commit and tag to the remote origin.

Use Bun.$ features throughout (not child_process, not execa). Handle errors at each step with clear error messages. Show the complete, runnable script file.