---
title: "Workspace Detection Solutions"
domain: "chris-config"
bc_versions: "1.5.0+"
difficulty: "intermediate"
tags: ["mcp-configuration", "workspace-management", "vscode", "troubleshooting"]
related_topics:
  - "layer-system-fundamentals.md"
  - "configuration-troubleshooting.md"
  - "project-layer-setup.md"
applies_to:
  - "BC Code Intelligence MCP Server"
  - "VS Code MCP Extension"
last_updated: "2025-10-27"
---

# Workspace Detection Solutions

## Overview

The VS Code MCP extension does **not** set `process.cwd()` to the workspace root, causing the BC Code Intelligence MCP server to be unable to detect project-local configuration and knowledge layers. This document explains the problem, its impact, and the workspace management solution introduced in v1.5.0.

## The Problem

### **Root Cause**

When the MCP server runs in VS Code via the MCP extension:
- `process.cwd()` points to the MCP extension's installation directory
- **NOT** the user's workspace directory
- Example: `C:\Users\Username\.vscode\extensions\...` instead of `C:\Work\MyProject\`

### **Impact**

Without correct working directory:
1. **Configuration Discovery Fails**: Can't find `./bc-code-intel-config.json` in workspace
2. **Project Layer Won't Load**: Can't access `./bc-code-intel-overrides/` directory
3. **Relative Paths Broken**: Any workspace-relative path resolution fails
4. **User Confusion**: Project-specific knowledge appears missing

## The Solution: Workspace Management Tools (v1.5.0+)

### **Two New MCP Tools**

**1. set_workspace_root** - Configure workspace explicitly
```typescript
set_workspace_root({ 
  workspace_root: "/absolute/path/to/workspace" 
})
```

**What it does:**
1. Validates the provided path exists
2. Calls `process.chdir(workspace_root)` to change working directory
3. Triggers full service reinitialization
4. Reloads all knowledge layers with new workspace context
5. Discovers project-local configuration if present

**2. get_workspace_root** - Query current workspace
```typescript
get_workspace_root()
```

**Returns:**
```json
{
  "workspace_root": "/current/working/directory",
  "services_initialized": true
}
```

### **First-Call Interception Pattern**

The MCP server implements **lazy initialization** with **first-call interception**:

**At Startup:**
- Server loads embedded knowledge only (fast startup, ~100ms)
- `servicesInitialized = false`
- All tool classes initialized but not ready

**On First Tool Call (before workspace set):**
- Tool handler checks `servicesInitialized === false`
- Returns helpful message instead of executing tool:

```
🚀 BC Code Intelligence MCP Server - Workspace Configuration

The MCP server needs workspace context to load project-specific knowledge and configuration.

REQUIRED ACTION:
Call the set_workspace_root tool with your workspace path:

set_workspace_root({ 
  workspace_root: "/absolute/path/to/your/workspace" 
})

This enables:
- Project-local configuration (bc-code-intel-config.json)
- Project knowledge overrides (bc-code-intel-overrides/)
- Team and company knowledge layers
- Full workspace-aware functionality

After setting workspace root, all tools will be available.
```

**After Workspace Set:**
- Full initialization runs
- All layers loaded (embedded, git, project)
- `servicesInitialized = true`
- All tools execute normally

## Usage Patterns

### **VS Code Workflow**

**Step 1: Open workspace in VS Code**
```
File → Open Folder → Select your BC project
```

**Step 2: Start MCP conversation**
```
User: "Hey, can you help me optimize this code?"
```

**Step 3: Server prompts for workspace**
```
MCP: "I need workspace context. Call set_workspace_root first..."
```

**Step 4: Set workspace root** (agent or user)
```typescript
set_workspace_root({ 
  workspace_root: "C:\\Work\\MyBCProject" 
})
```

**Step 5: Full functionality available**
```
✅ Project config loaded
✅ Project knowledge layers available
✅ All tools ready
```

### **Claude Desktop Workflow** (Manual Setup)

**Option A: Environment Variable** (Recommended)
```bash
# Set before launching Claude Desktop
export BC_WORKSPACE_ROOT="/path/to/workspace"

# On Windows PowerShell:
$env:BC_WORKSPACE_ROOT = "C:\Work\MyBCProject"
```

**Option B: Manual Tool Call**
```typescript
// First interaction with MCP
set_workspace_root({ 
  workspace_root: "/path/to/workspace" 
})
```

### **Automated CI/CD Workflow**

```bash
# CI script sets workspace before running MCP tools
export BC_WORKSPACE_ROOT="$CI_PROJECT_DIR"

# MCP server auto-detects from environment
# No set_workspace_root call needed
```

## Configuration Discovery After Workspace Set

Once workspace is set, configuration discovery searches:

1. **Workspace Root**
   - `./bc-code-intel-config.json`
   - `./bc-code-intel-config.yaml`
   - `./bc-code-intel-config.yml`

2. **User Home Directory** (expanded paths)
   - `~/.bc-code-intel/config.json`
   - `~/.bc-code-intel/config.yaml`
   - `~/.bc-code-intel/config.yml`

3. **Environment Variable**
   - `BC_CODE_INTEL_CONFIG=/path/to/config.json`

4. **Default Embedded Config**
   - Embedded fallback if no config found

**Priority:** Workspace config > Environment variable > Home directory > Embedded

## Project Layer After Workspace Set

With workspace configured, project layer loads from:

```
{workspace_root}/bc-code-intel-overrides/
├── domains/              # Project-specific topics
├── specialists/          # Project-specific specialists (future)
└── methodologies/        # Project-specific workflows (future)
```

**Layer Priority:** 100 (highest - always wins)

## Troubleshooting

### **"Project config not found"**

**Problem:** Configuration exists but not being loaded

**Solution:**
```typescript
// Verify current workspace
get_workspace_root()

// If wrong, set correct path
set_workspace_root({ workspace_root: "/correct/path" })
```

### **"Project layer showing 0 topics"**

**Problem:** `bc-code-intel-overrides/` directory structure incorrect

**Check:**
1. Directory exists: `{workspace}/bc-code-intel-overrides/`
2. Content in subdirectories: `domains/`, `specialists/`, `methodologies/`
3. Proper frontmatter in markdown files

**Solution:**
```bash
# Create proper structure
mkdir -p bc-code-intel-overrides/domains/my-domain
mkdir -p bc-code-intel-overrides/specialists
```

### **"Services not initializing after set_workspace_root"**

**Problem:** Path doesn't exist or insufficient permissions

**Verify:**
```bash
# Check path exists
ls -la /path/to/workspace

# Check permissions
# Should be readable by MCP server process
```

**Solution:**
- Use absolute paths (not relative)
- Ensure directory exists before calling set_workspace_root
- Check file permissions

## Implementation Details

### **Lazy Initialization Benefits**

1. **Fast Startup**: Embedded knowledge only (~100ms)
2. **User-Friendly**: Clear guidance when workspace needed
3. **Zero-Config Default**: Works without workspace for embedded-only usage
4. **Explicit Configuration**: User/agent explicitly opts into workspace features

### **Service Reinitialization on Workspace Change**

When `set_workspace_root` is called:
```typescript
// 1. Change working directory
process.chdir(workspace_root)

// 2. Reload configuration
configService.loadConfiguration()

// 3. Reinitialize all services
await initializeServices()

// 4. Reload all knowledge layers
await layerService.loadAllLayers()
```

**Performance:** Full reinitialization takes 1-3 seconds including git layer cloning (if not cached).

## Best Practices

1. **Set Workspace Early**: Call `set_workspace_root` at the start of MCP conversations in VS Code
2. **Absolute Paths**: Always use absolute paths, not relative
3. **Verify Before Setting**: Ensure workspace directory exists and is accessible
4. **Check After Setting**: Use `get_workspace_root` to confirm workspace configured correctly
5. **Environment Variables**: For CLI/CI scenarios, use `BC_WORKSPACE_ROOT` env var

## Advanced: Environment-Based Auto-Configuration

### **Setting Default Workspace**

Create a shell alias or launcher script:

```bash
# ~/.bashrc or ~/.zshrc
export BC_WORKSPACE_ROOT="$HOME/BCProjects/current"

# Windows PowerShell profile
$env:BC_WORKSPACE_ROOT = "C:\Work\CurrentProject"
```

### **Per-Project Launch Scripts**

```bash
#!/bin/bash
# launch-mcp.sh in project root

export BC_WORKSPACE_ROOT="$(pwd)"
code .  # Launch VS Code with workspace set
```

## Migration from Pre-1.5.0

**Before v1.5.0:**
- Project layer only worked in environments where `process.cwd()` was correct
- VS Code users couldn't use project-local features
- No workaround available

**After v1.5.0:**
- Explicit workspace configuration via `set_workspace_root`
- Works in all environments (VS Code, Claude Desktop, CLI)
- Backward compatible - embedded knowledge still works without workspace

**No Breaking Changes:** Existing configurations continue to work. Workspace management is optional but recommended for project features.

## See Also

- [Layer System Fundamentals](layer-system-fundamentals.md) - Understanding layer architecture
- [Project Layer Setup](project-layer-setup.md) - Setting up project-local knowledge
- [Configuration File Discovery](configuration-file-discovery.md) - Where configs are found
- [Configuration Troubleshooting](configuration-troubleshooting.md) - Common issues and solutions
