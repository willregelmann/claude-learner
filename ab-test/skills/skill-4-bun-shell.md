---
name: bun-shell-api
description: Use when writing Bun scripts that use the $ shell API, running system commands, building CLI tools, or scripting with Bun.
---

# Bun Shell API (Bun.$)

## When This Skill Applies

- Writing shell scripts in JavaScript/TypeScript with Bun
- Running system commands and capturing their output
- Building CLI tools that execute external programs
- Automating tasks like file manipulation, git operations, or build processes
- Replacing bash/zsh scripts with cross-platform JavaScript alternatives
- Migrating from Node.js child_process, execa, or shelljs to Bun
- Processing command output with streams or buffers
- Chaining commands with pipes or command substitution
- Managing environment variables for subprocess execution
- Reading configuration files like package.json in Bun scripts

## Key Patterns

### Basic Command Execution

Import and use the `$` tagged template literal to execute shell commands. Commands run asynchronously and return promises.

```javascript
import { $ } from "bun";

// Simple command execution
await $`ls -la`;

// Capture output as text
const result = await $`echo "hello world"`;
const text = result.text(); // "hello world\n"

// Access stdout/stderr as Buffer
const output = result.stdout; // Buffer
const errors = result.stderr; // Buffer
```

The shell does not invoke a system shell like `/bin/sh`. Instead, it's a re-implementation of bash that runs directly in the Bun process, making it faster and more secure.

### Variable Interpolation and Security

All interpolated JavaScript values are treated as single, literal strings to prevent shell injection attacks. This automatic escaping makes the Bun shell inherently safer than traditional shell scripting.

```javascript
const filename = "file with spaces.txt";
await $`cat ${filename}`; // Safe: treated as single argument

const files = ["file1.txt", "file2.txt"];
await $`cat ${files}`; // Expands to: cat file1.txt file2.txt

// Interpolate command output
const branch = await $`git branch --show-current`.text();
await $`echo "Current branch: ${branch}"`;
```

### Error Handling and Exit Codes

By default, commands with non-zero exit codes throw a `ShellError`. Access `exitCode`, `stdout`, and `stderr` properties from the error object.

```javascript
// Default: throws on failure
try {
  await $`exit 1`;
} catch (err) {
  console.log(err.exitCode); // 1
  console.log(err.stdout.toString());
  console.log(err.stderr.toString());
}

// Disable throwing with .nothrow()
const result = await $`exit 1`.nothrow();
if (result.exitCode !== 0) {
  console.error("Command failed:", result.stderr.toString());
}

// Conditionally control throwing
const shouldThrow = process.env.STRICT === "true";
await $`some-command`.throws(shouldThrow);
```

Always use `.nothrow()` when you expect commands to potentially fail and want to handle the result programmatically rather than with try-catch.

### Piping and Command Chaining

Chain commands using pipes (`|`) just like in bash. The output of one command becomes the input of the next.

```javascript
// Pipe commands
await $`cat file.txt | grep "error" | wc -l`;

// Command substitution: use output of one command in another
const files = await $`ls *.txt`.text();
await $`echo ${files}`;

// Combine with JavaScript processing
const errors = await $`cat logs.txt | grep ERROR`.text();
const errorLines = errors.split('\n').filter(Boolean);
console.log(`Found ${errorLines.length} errors`);
```

### Redirection and Streams

Redirect stdin, stdout, and stderr using JavaScript objects. Bun supports `Response`, `ArrayBuffer`, `Blob`, `Bun.file()`, and more.

```javascript
// Redirect stdout to a file
await $`echo "log entry"`.stdout(Bun.file("output.log"));

// Read from file as stdin
await $`cat`.stdin(Bun.file("input.txt"));

// Use Response objects
const response = await fetch("https://api.example.com/data");
await $`jq .name`.stdin(response);

// Redirect stderr separately
await $`some-command`.stderr(Bun.file("errors.log"));
```

### Environment Variables

Set environment variables globally for all commands or locally for specific commands using `.env()`.

```javascript
// Global environment variables (uses process.env by default)
$.env({ NODE_ENV: "production", DEBUG: "true" });
await $`echo $NODE_ENV`; // "production"

// Local environment variables for a single command
await $`echo $API_KEY`.env({ API_KEY: "secret123" });

// Combine with existing environment
await $`printenv`.env({ ...process.env, CUSTOM: "value" });

// Access environment in command output
const nodeEnv = await $`echo $NODE_ENV`.text();
```

Global settings via `$.env()` persist across all subsequent `$` calls in the script, while `.env()` on individual commands only affects that specific invocation.

### Working Directory

Change the working directory for command execution using `.cwd()`.

```javascript
// Execute in specific directory
await $`pwd`.cwd("/tmp"); // runs in /tmp

// Relative to current directory
await $`ls`.cwd("./subdirectory");

// Combine with other methods
const result = await $`npm install`
  .cwd("/path/to/project")
  .env({ NODE_ENV: "development" })
  .nothrow();
```

The working directory change only affects the specific command, not the entire Node process.

### Reading Files with Bun APIs

Use `Bun.file()` for efficient file reading. The API is lazy-loading and uses modern async patterns.

```javascript
// Read JSON files
const pkg = await Bun.file("package.json").json();
console.log(pkg.name, pkg.version);

// Read text files
const content = await Bun.file("README.md").text();

// Read as Buffer
const buffer = await Bun.file("image.png").arrayBuffer();

// Check if file exists
const file = Bun.file("config.json");
const exists = await file.exists();

// Combine with shell commands
const pkg = await Bun.file("package.json").json();
await $`echo "Building ${pkg.name} v${pkg.version}"`;
```

Unlike Node.js `fs.readFile`, `Bun.file()` returns a `BunFile` object that extends `Blob`, providing a lazy reference to the file without immediately loading contents into memory.

### Built-in Commands

Bun shell implements common commands natively for cross-platform compatibility: `ls`, `cd`, `rm`, `mv`, `cp`, `mkdir`, `echo`, `cat`, `pwd`, and more.

```javascript
// These work identically on Windows, macOS, and Linux
await $`ls -la`;
await $`rm -rf node_modules`;
await $`mkdir -p dist/assets`;
await $`cp package.json dist/`;
```

Using built-in commands ensures your scripts run consistently across platforms without depending on system utilities.

### Streaming Output

Process command output as a stream for large outputs or real-time processing.

```javascript
// Get output as ReadableStream
const proc = $`tail -f logfile.txt`.nothrow();
const stream = proc.stdout; // ReadableStream

// Process chunks as they arrive
for await (const chunk of proc.stdout) {
  console.log("Received:", chunk.toString());
}

// Stream to another command
const result = await $`cat large-file.txt | gzip > output.gz`;
```

Unlike Node.js which returns strings by default, Bun returns `ReadableStream` objects for stdout and stderr, enabling efficient streaming of large outputs.

### Checking Process Status

Access process information and wait for completion.

```javascript
// Get exit code
const result = await $`ls`.nothrow();
console.log(result.exitCode); // 0 for success

// Check if command succeeded
const success = result.exitCode === 0;

// Get process information
console.log(result.stdout.toString());
console.log(result.stderr.toString());
```

## Common Mistakes to Avoid

**Mistake**: Forgetting to await the `$` command
```javascript
$`echo "hello"`; // ❌ Returns a promise, doesn't execute
```
**Correction**: Always await shell commands
```javascript
await $`echo "hello"`; // ✅ Properly executes
```

**Mistake**: Using string concatenation for commands instead of template literals
```javascript
await $("echo " + message); // ❌ Wrong syntax
```
**Correction**: Use tagged template literals with interpolation
```javascript
await $`echo ${message}`; // ✅ Correct syntax
```

**Mistake**: Not handling errors when commands might fail
```javascript
await $`some-flaky-command`; // ❌ Throws on non-zero exit
```
**Correction**: Use `.nothrow()` and check exit codes
```javascript
const result = await $`some-flaky-command`.nothrow();
if (result.exitCode !== 0) {
  // handle error
}
```

**Mistake**: Assuming you can use complex bash syntax
```javascript
await $`if [ -f file ]; then echo "exists"; fi`; // ❌ Complex bash constructs may not work
```
**Correction**: Use JavaScript control flow
```javascript
const exists = await Bun.file("file").exists();
if (exists) {
  await $`echo "exists"`;
}
```

**Mistake**: Treating stdout/stderr as strings directly
```javascript
const result = await $`ls`.nothrow();
console.log(result.stdout.includes("file")); // ❌ stdout is a Buffer/Stream
```
**Correction**: Convert to string or use `.text()`
```javascript
const result = await $`ls`;
const output = result.text(); // ✅ Get as string
console.log(output.includes("file"));
```

**Mistake**: Using Node.js child_process patterns
```javascript
const { exec } = require("child_process"); // ❌ Node.js API
exec("ls", (err, stdout) => { /* ... */ });
```
**Correction**: Use Bun's native `$` shell API
```javascript
import { $ } from "bun";
const result = await $`ls`; // ✅ Bun-native
const output = result.text();
```

**Mistake**: Expecting synchronous execution
```javascript
const output = $`echo "hello"`.text(); // ❌ Need to await twice
```
**Correction**: Await the command execution first
```javascript
const result = await $`echo "hello"`;
const output = result.text(); // ✅ Or: await $`echo "hello"`.text()
```

## Examples

### Example 1: Cross-Platform Build Script

```javascript
#!/usr/bin/env bun
import { $ } from "bun";

// Read package.json
const pkg = await Bun.file("package.json").json();
console.log(`Building ${pkg.name} v${pkg.version}`);

// Clean previous build
await $`rm -rf dist`;
await $`mkdir -p dist`;

// Run build with environment variables
const result = await $`tsc --outDir dist`
  .env({ NODE_ENV: "production" })
  .nothrow();

if (result.exitCode !== 0) {
  console.error("Build failed:", result.stderr.toString());
  process.exit(1);
}

// Copy static assets
await $`cp -r public dist/`;

// Create archive
await $`tar -czf dist.tar.gz dist`;

console.log("Build complete!");
```

### Example 2: Git Helper Script

```javascript
#!/usr/bin/env bun
import { $ } from "bun";

// Check if on main branch
const branch = (await $`git branch --show-current`.text()).trim();

if (branch === "main") {
  console.error("Cannot run on main branch!");
  process.exit(1);
}

// Check for uncommitted changes
const status = await $`git status --porcelain`.nothrow();
const hasChanges = status.text().trim().length > 0;

if (hasChanges) {
  console.log("Uncommitted changes detected, stashing...");
  await $`git stash`;
}

// Pull latest changes
await $`git pull origin main`;

// Run tests in specific directory
const testResult = await $`bun test`
  .cwd("./tests")
  .env({ CI: "true" })
  .nothrow();

if (testResult.exitCode !== 0) {
  console.error("Tests failed!");
  if (hasChanges) {
    await $`git stash pop`;
  }
  process.exit(1);
}

// Restore stashed changes
if (hasChanges) {
  await $`git stash pop`;
}

console.log("All checks passed!");
```

Sources:
- [Bun Shell Documentation](https://bun.com/docs/runtime/shell)
- [Bun.$ API Reference](https://bun.com/reference/bun/$)
- [The Bun Shell Blog Post](https://bun.com/blog/the-bun-shell)
- [Read a JSON file - Bun](https://bun.com/docs/guides/read-file/json)
- [Run shell command and collect output comparison](https://medium.com/deno-the-complete-reference/run-shell-command-and-collect-the-output-in-node-js-deno-and-bun-f23977bf1d68)