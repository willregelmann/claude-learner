Now I'll create the deployment helper script using Bun's native $ shell API:

**deploy.js**

```javascript
#!/usr/bin/env bun
import { $ } from "bun";

console.log("🚀 Starting deployment process...\n");

// Step 1: Check if git working directory is clean
console.log("Step 1: Checking git status...");
const status = await $`git status --porcelain`.nothrow();
const hasChanges = status.text().trim().length > 0;

if (hasChanges) {
  console.error("❌ Error: Git working directory is dirty. Please commit or stash your changes.");
  console.error("\nUncommitted changes:");
  console.error(status.text());
  process.exit(1);
}
console.log("✅ Git working directory is clean\n");

// Step 2: Run test suite
console.log("Step 2: Running test suite...");
const testResult = await $`bun test`.nothrow();

if (testResult.exitCode !== 0) {
  console.error("❌ Error: Tests failed");
  console.error(testResult.stderr.toString());
  process.exit(1);
}
console.log("✅ All tests passed\n");

// Step 3: Build the project
console.log("Step 3: Building project...");
const buildResult = await $`bun run build`.nothrow();

if (buildResult.exitCode !== 0) {
  console.error("❌ Error: Build failed");
  console.error(buildResult.stderr.toString());
  process.exit(1);
}
console.log("✅ Build completed successfully\n");

// Step 4: Read version from package.json and create git tag
console.log("Step 4: Creating git tag...");
const pkg = await Bun.file("package.json").json();
const version = pkg.version;

if (!version) {
  console.error("❌ Error: No version found in package.json");
  process.exit(1);
}

const tag = `v${version}`;
console.log(`Creating tag: ${tag}`);

const tagResult = await $`git tag ${tag}`.nothrow();

if (tagResult.exitCode !== 0) {
  const errorMsg = tagResult.stderr.toString();
  if (errorMsg.includes("already exists")) {
    console.error(`❌ Error: Tag ${tag} already exists`);
  } else {
    console.error("❌ Error: Failed to create git tag");
    console.error(errorMsg);
  }
  process.exit(1);
}
console.log(`✅ Tag ${tag} created\n`);

// Step 5: Push commit and tag to remote origin
console.log("Step 5: Pushing to remote origin...");

const pushResult = await $`git push origin`.nothrow();
if (pushResult.exitCode !== 0) {
  console.error("❌ Error: Failed to push commits");
  console.error(pushResult.stderr.toString());
  process.exit(1);
}
console.log("✅ Pushed commits to origin");

const pushTagResult = await $`git push origin ${tag}`.nothrow();
if (pushTagResult.exitCode !== 0) {
  console.error(`❌ Error: Failed to push tag ${tag}`);
  console.error(pushTagResult.stderr.toString());
  process.exit(1);
}
console.log(`✅ Pushed tag ${tag} to origin\n`);

console.log(`🎉 Deployment successful! Version ${version} has been tagged and pushed.`);
```