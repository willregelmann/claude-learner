```typescript
// deploy.ts
import { $ } from "bun";
import { exit } from "process";

async function main() {
  console.log("🚀 Starting deployment process...\n");

  // Step 1: Check if git working directory is clean
  console.log("📋 Checking git status...");
  try {
    const gitStatus = await $`git status --porcelain`.text();
    if (gitStatus.trim() !== "") {
      console.error("❌ Error: Git working directory is not clean.");
      console.error("Please commit or stash your changes before deploying.\n");
      console.error("Uncommitted changes:");
      console.error(gitStatus);
      exit(1);
    }
    console.log("✅ Git working directory is clean\n");
  } catch (error) {
    console.error("❌ Failed to check git status:", error);
    exit(1);
  }

  // Step 2: Run test suite
  console.log("🧪 Running test suite...");
  try {
    await $`bun test`.quiet();
    console.log("✅ All tests passed\n");
  } catch (error) {
    console.error("❌ Tests failed. Please fix failing tests before deploying.");
    exit(1);
  }

  // Step 3: Build the project
  console.log("🔨 Building project...");
  try {
    await $`bun run build`.quiet();
    console.log("✅ Build successful\n");
  } catch (error) {
    console.error("❌ Build failed. Please fix build errors before deploying.");
    exit(1);
  }

  // Step 4: Read version from package.json and create git tag
  console.log("🏷️  Creating git tag...");
  try {
    const packageJsonPath = "./package.json";
    const packageJson = await Bun.file(packageJsonPath).json();
    const version = packageJson.version;

    if (!version) {
      console.error("❌ No version found in package.json");
      exit(1);
    }

    const tag = `v${version}`;
    console.log(`Creating tag: ${tag}`);

    // Check if tag already exists
    try {
      const existingTags = await $`git tag -l ${tag}`.text();
      if (existingTags.trim() === tag) {
        console.error(`❌ Tag ${tag} already exists. Please update the version in package.json.`);
        exit(1);
      }
    } catch (error) {
      // Tag doesn't exist, continue
    }

    await $`git tag ${tag}`.quiet();
    console.log(`✅ Created tag ${tag}\n`);
  } catch (error) {
    console.error("❌ Failed to create git tag:", error);
    exit(1);
  }

  // Step 5: Push commit and tag to remote origin
  console.log("📤 Pushing to remote...");
  try {
    const packageJson = await Bun.file("./package.json").json();
    const tag = `v${packageJson.version}`;

    await $`git push origin`.quiet();
    console.log("✅ Pushed commits to remote");

    await $`git push origin ${tag}`.quiet();
    console.log(`✅ Pushed tag ${tag} to remote\n`);
  } catch (error) {
    console.error("❌ Failed to push to remote:", error);
    exit(1);
  }

  console.log("🎉 Deployment completed successfully!");
}

main();
```