# 🚀 NewsoftDS Nx Monorepo Buildpack

**Simple, clean Heroku buildpack for Nx monorepos with automatic dependency pruning.**

Perfect for deploying individual services from your Nx monorepo to Heroku with minimal slug size and only the dependencies each service actually needs.

## ✨ What it does

1. **🔍 Detects** your Nx workspace 
2. **📦 Installs** Node.js and Bun
3. **🏗️ Builds** your specified Nx project with `generatePackageJson: true`
4. **🎯 Installs** only the production dependencies your app needs (resolves `workspace:*`)
5. **🧹 Cleans** up unnecessary files to minimize slug size
6. **✅ Ready** for deployment with clean logs throughout

## 🎯 Key Features

- ✅ **Two required environment variables**: `BP_BUILD` + `BP_START`
- ✅ **Bun-first approach** - Node.js is optional and disabled by default
- ✅ **Automatic dependency resolution** - `workspace:*` becomes real versions
- ✅ **Minimal deployments** - only installs what your app actually uses
- ✅ **Beautiful logs** - clear, colorful progress indicators
- ✅ **Nx native** - leverages `generatePackageJson` feature
- ✅ **Zero configuration** - works out of the box with sensible defaults

## 🚀 Why Bun-first?

**Bun is faster and more efficient than Node.js for most use cases:**
- ⚡ **3x faster** startup times
- 🎯 **Built-in bundler** - no webpack needed
- 📦 **Native TypeScript** support
- 🔧 **Drop-in Node.js replacement** for most packages

**When you might need Node.js:**
- Legacy dependencies that don't work with Bun yet
- Specific Node.js native modules
- Docker containers that expect Node.js
- Just set `BP_NODE=true` if needed

## 📝 Bun Version Control

The buildpack supports multiple ways to specify the Bun version (in priority order):

1. **`runtime.bun.txt`** - Create this file in your repo root with version (e.g., `1.1.38`)
2. **`runtime.txt`** - Alternative file with version (e.g., `1.1.38`) 
3. **`BP_BUN_VERSION`** - Environment variable (e.g., `heroku config:set BP_BUN_VERSION=1.1.38`)
4. **Default** - Uses latest Bun version if none specified

```bash
# Method 1: Create file in repo
echo "1.1.38" > runtime.bun.txt

# Method 2: Environment variable  
heroku config:set BP_BUN_VERSION=1.1.38

# Method 3: Use default (latest)
# No action needed
```

## 🔧 Environment Variables

### Required

| Variable | Description | Example |
|----------|-------------|---------|
| `BP_BUILD` | Nx project name to build | `@newsoftds/api-gateway-server` |
| `BP_START` | Start command for your app | `bun ./index.js` |

### Optional

| Variable | Default | Description |
|----------|---------|-------------|
| `BP_NODE` | `false` | Install Node.js alongside Bun (set to `true` if needed) |
| `BP_NODE_VERSION` | `22.11.0` | Node.js version to install (must be specific version, not "latest") |
| `BP_BUN_VERSION` | `latest` | Bun version to install (or use `runtime.bun.txt`/`runtime.txt` files) |
| `BP_INSTALL` | `bun install` | Command to install monorepo dependencies |
| `BP_CLEAN` | `true` | Clean up source files after build |

## 📋 Setup Guide

### 1. Configure your Nx project

Make sure your `project.json` has `generatePackageJson: true`:

```json
{
  "targets": {
    "build": {
      "executor": "@nx/js:tsc",
      "options": {
        "generatePackageJson": true,
        "outputPath": "dist/apis/servers/api-gateway-server"
      }
    }
  }
}
```

### 2. Set Heroku environment variables

```bash
# Required: Which project to build and how to start it
heroku config:set BP_BUILD=@newsoftds/api-gateway-server
heroku config:set BP_START="bun ./index.js"

# Optional: Enable Node.js if your project needs it
heroku config:set BP_NODE=true
heroku config:set BP_NODE_VERSION=22.11.0

# Optional: Customize Bun version
heroku config:set BP_BUN_VERSION=latest
```

### 3. Deploy (No Procfile needed!)

```bash
git push heroku main
```

## 🎯 Examples

### API Gateway Server

```bash
heroku config:set BP_BUILD=@newsoftds/api-gateway-server
heroku config:set BP_START="bun ./index.js"
```

### University API

```bash
heroku config:set BP_BUILD=@newsoftds/api-university-server
heroku config:set BP_START="bun start"
```

### Web Application

```bash
heroku config:set BP_BUILD=@newsoftds/portal-paciente
heroku config:set BP_START="bunx serve -s . -l $PORT"
```

### With Node.js (if your project needs it)

```bash
# Some projects might need Node.js for specific dependencies
heroku config:set BP_BUILD=@newsoftds/api-university-server
heroku config:set BP_START="bun start"
heroku config:set BP_NODE=true
```

### Custom Start Command with Full Path

```bash
# If you need full control, you can specify the complete command:
heroku config:set BP_START="cd dist/apis/servers/api-gateway-server && NODE_ENV=production bun ./index.js"
```

## 📊 Build Output Example

```
🚀 NewsoftDS Nx Monorepo Buildpack v2.0

   ℹ️  Project to build: @newsoftds/api-gateway-server
   ℹ️  Start command: bun ./index.js
   ℹ️  Node.js: disabled (Bun only)
   ℹ️  Bun version: latest
   
   📦 Setting up build environment
   📦 Skipping Node.js installation (BP_NODE=false)
   ℹ️  Using Bun only - set BP_NODE=true if you need Node.js
   
   📦 Installing Bun
   ℹ️  Using latest Bun version (no specific version specified)
   ✅ Bun installed: 1.2.20
   
   📦 Installing monorepo dependencies
   ✅ Dependencies installed
   
   📦 Validating Nx project: @newsoftds/api-gateway-server
   ✅ Found project: @newsoftds/api-gateway-server
   
   📦 Building @newsoftds/api-gateway-server with Nx (generatePackageJson enabled)
   ✅ Build completed - output in: dist/apis/servers/api-gateway-server
   ℹ️  Generated package.json found: 62 lines
   
   📦 Installing production dependencies in build output
   ℹ️  Installing 12 production dependencies (workspace:* resolved to actual versions)
   ✅ Production dependencies installed: 15M in node_modules
   
   📦 Cleaning up to reduce slug size
   ℹ️  Removed root node_modules
   ℹ️  Removed build caches  
   ℹ️  Removed source directories
   ✅ Cleanup complete - slug size: 45M

🚀 Build Summary

   ✅ Project: @newsoftds/api-gateway-server
   ✅ Built to: dist/apis/servers/api-gateway-server
   ✅ Dependencies: 12 packages installed
   ✅ Runtime: Bun 1.1.38 (Node.js disabled)
   
   ℹ️  Your app will start with: bun ./index.js
   ℹ️  To change start command: heroku config:set BP_START='bun ./index.js'
   ℹ️  No Procfile needed - start command is configured via BP_START variable

✅ Build Complete - Ready for deployment!
```

## 🔍 Troubleshooting

### Missing required environment variables
```
❌ Build Failed - Missing Required Environment Variables

   ❌ BP_BUILD environment variable is required!
   ℹ️  Set it with: heroku config:set BP_BUILD=@newsoftds/your-project-name
   ℹ️  Available projects: @newsoftds/api-gateway-server @newsoftds/api-clinic-server

   ❌ BP_START environment variable is required!
   ℹ️  Set it with: heroku config:set BP_START='bun ./index.js'
   ℹ️  Common examples: 'bun start', 'bun ./index.js', 'bunx serve -s . -l $PORT'
```
**Solution**: Set both required environment variables in Heroku.

### Workspace dependency resolution errors
```
error: Workspace dependency "@types/jsonwebtoken" not found
error: Workspace dependency "hono" not found
error: @types/jsonwebtoken@workspace:* failed to resolve
```
**Solution**: This happens when external NPM packages are incorrectly marked as `workspace:*` dependencies. Only internal packages should use `workspace:*`. External packages should use actual version numbers:
```json
// ✅ Correct - internal packages use workspace:*
"@newsoftds/graphql-core": "workspace:*",

// ✅ Correct - external packages use versions  
"@pothos/core": "^4.8.1",
"hono": "^4.6.10"

// ❌ Wrong - external package marked as workspace
"hono": "workspace:*"
```

### Nx resolution errors
```
❌ Project '@newsoftds/api-gateway-server' not found in Nx workspace!
error: ENOENT while resolving package 'nx/bin/nx.js'
```
**Solution**: This usually happens when dependencies aren't installed yet. The buildpack now installs dependencies first, then validates the project. If you still see this, check:
```bash
# Make sure your project name is correct
heroku config:get BP_BUILD

# Check available projects locally
bunx nx show projects --type=application
```

### Project not found
```
❌ Project '@newsoftds/my-app' not found in Nx workspace!
ℹ️  Available projects:
```
**Solution**: Check your `BP_BUILD` variable matches an actual Nx project name.

### Missing package.json in build output
```
❌ Build output not found or missing package.json!
```
**Solution**: Make sure your `project.json` has `"generatePackageJson": true` in the build target options.

### Missing environment variables in release
```
ERROR: Missing required environment variables
Please set: heroku config:set BP_BUILD=@newsoftds/api-gateway-server BP_START='bun ./index.js'
```
**Solution**: Ensure both required environment variables are set:
```bash
heroku config:set BP_BUILD=@newsoftds/api-gateway-server
heroku config:set BP_START='bun ./index.js'
```

### Workspace dependencies not found
```
error: Workspace dependency "@newsoftds/graphql-core" not found
Searched in "./*"
Workspace documentation: https://bun.com/docs/install/workspaces
```
**Solution**: This indicates workspace resolution issues. The buildpack now includes comprehensive debugging:
- ✅ Checks for workspace packages in correct directories  
- ✅ Validates `package.json` workspace configuration
- ✅ Retries installation with verbose logging if first attempt fails
- ✅ Regenerates lockfile if needed
- ✅ Removes conflicting lockfiles from dist directory
- ✅ Validates generated package.json has no workspace references

The new debugging output will show:
```bash
📦 Installing production dependencies in build output
ℹ️  Files in dist directory: [lists all files]
ℹ️  Installing 17 production dependencies (from generated package.json)
ℹ️  Found lockfile in dist directory - removing to avoid workspace conflicts
ℹ️  No workspace: references found in generated package.json ✅
ℹ️  Running: bun install --production --no-save --no-optional
```

Common causes:
1. **Missing workspace packages**: Ensure all referenced packages exist in the monorepo
2. **Incorrect `workspace:*` usage**: Use `workspace:*` only for internal monorepo packages, not external NPM dependencies  
3. **Lockfile conflicts**: The buildpack automatically removes conflicting lockfiles from the dist directory
4. **Nx generatePackageJson issues**: Ensure your `project.json` has `"generatePackageJson": true`

### Bun installation fails (404 error)
```
📦 Installing Bun latest
curl: (22) The requested URL returned error: 404
error: Failed to download bun from "https://github.com/oven-sh/bun/releases/download/bun-latest/bun-linux-x64.zip"
```
**Solution**: Use a specific Bun version or let it auto-detect latest:
```bash
# Option 1: Use specific version
heroku config:set BP_BUN_VERSION=1.1.38

# Option 2: Use default (auto-detects latest)
heroku config:unset BP_BUN_VERSION
```

### Node.js installation hangs or times out
```
📦 Installing Node.js v22.11.0
(stuck for several minutes)
```
**Solution**: The Node.js download might be slow or timing out. Since Bun/Nx compatibility issues are fixed, you likely don't need Node.js:
```bash
# Disable Node.js for faster builds (recommended)
heroku config:unset BP_NODE

# OR if you really need Node.js, try a different version
heroku config:set BP_NODE_VERSION=20.18.0
```

### Node.js installation fails
```
📦 Installing Node.js vlatest
xz: (stdin): File format not recognized  
tar: Child returned status 1
```
**Solution**: Use a specific Node.js version number, not "latest":
```bash
heroku config:set BP_NODE_VERSION=22.11.0
```

### Build fails
Check that your project builds locally first:
```bash
bunx nx build @newsoftds/your-project --configuration=production
```

## 🚀 Migration from old buildpack

**Old way** (complex):
- Multiple environment variables: `APP_DIR`, `RUNTIME_DIR`, `BUILD_COMMAND`, etc.
- Custom package.json generation logic
- Complex path resolution
- Required Procfile management

**New way** (simple):
- Two variables: `BP_BUILD` + `BP_START`
- Nx handles package.json generation
- Smart auto-detection of build output
- No Procfile needed

Just update your Heroku config:
```bash
# Remove old variables
heroku config:unset APP_DIR RUNTIME_DIR BUILD_COMMAND START_COMMAND WEB_START_COMMAND

# Set new variables  
heroku config:set BP_BUILD=@newsoftds/api-gateway-server
heroku config:set BP_START="bun ./index.js"

# Remove Procfile (no longer needed)
rm Procfile
```

## 📝 Requirements

- Nx workspace with projects configured for `generatePackageJson`
- `package.json` and `bun.lock` in repository root
- Heroku app with this buildpack configured

---

**Built for NewsoftDS monorepo** 🏗️ **Powered by Nx + Bun** ⚡