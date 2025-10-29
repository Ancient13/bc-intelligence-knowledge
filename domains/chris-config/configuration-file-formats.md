---
title: "Configuration File Formats and Creation"
domain: "chris-config"
bc_versions: "1.5.0+"
difficulty: "beginner"
tags: ["mcp-configuration", "json", "yaml", "setup", "getting-started"]
related_topics:
  - "configuration-file-discovery.md"
  - "layer-system-fundamentals.md"
  - "git-layer-configuration.md"
applies_to:
  - "BC Code Intelligence MCP Server"
last_updated: "2025-10-29"
---

# Configuration File Formats and Creation

## Overview

The BC Code Intelligence MCP server supports flexible configuration through JSON or YAML files. This guide shows you how to create configuration files for different scenarios, from simple setups to complex multi-layer architectures.

## Supported Formats

The MCP server accepts configuration in **two formats**:

### **JSON Format** (Most Common)
- File names: `bc-code-intel-config.json`
- Syntax: Standard JSON with comments NOT supported
- Best for: Simple configurations, CI/CD environments
- Validation: Strict JSON parsing

### **YAML Format** (Human-Friendly)
- File names: `bc-code-intel-config.yaml` or `bc-code-intel-config.yml`
- Syntax: YAML with indentation-based structure
- Best for: Complex configurations, human readability
- Validation: YAML parser with schema validation

## Quick Start: Zero Configuration

**No config file needed!** The MCP server works immediately with embedded knowledge:

```bash
# Just install and run
npx bc-code-intelligence-mcp

# Embedded knowledge (87+ topics, 14 specialists) available automatically
```

**When to skip config files:**
- Individual developer using embedded BC knowledge only
- Trying out the MCP server for the first time
- No company-specific knowledge to add
- No team conventions to enforce

## Basic Configuration Examples

### **Example 1: Enable Diagnostic Tools**

**JSON:**
```json
{
  "enable_diagnostic_tools": true
}
```

**YAML:**
```yaml
enable_diagnostic_tools: true
```

**What this does:** Enables advanced MCP tools for layer diagnostics and troubleshooting.

### **Example 2: Add a Company Git Layer**

**JSON:**
```json
{
  "knowledge_layers": [
    {
      "name": "Company BC Standards",
      "type": "git",
      "priority": 20,
      "source": {
        "repository_url": "https://dev.azure.com/myorg/BC-Knowledge/_git/standards",
        "branch": "main",
        "auth": {
          "type": "az_cli"
        }
      },
      "enabled": true
    }
  ]
}
```

**YAML:**
```yaml
knowledge_layers:
  - name: "Company BC Standards"
    type: git
    priority: 20
    source:
      repository_url: "https://dev.azure.com/myorg/BC-Knowledge/_git/standards"
      branch: main
      auth:
        type: az_cli
    enabled: true
```

**What this does:** Adds a company knowledge layer from Azure DevOps with Azure CLI authentication (preferred method).

### **Example 3: Full Multi-Layer Setup**

**JSON:**
```json
{
  "knowledge_layers": [
    {
      "name": "Company Standards",
      "type": "git",
      "priority": 20,
      "source": {
        "repository_url": "https://dev.azure.com/myorg/BC-Knowledge/_git/standards",
        "branch": "main",
        "auth": {
          "type": "az_cli"
        }
      },
      "enabled": true
    },
    {
      "name": "Team Conventions",
      "type": "git",
      "priority": 50,
      "source": {
        "repository_url": "https://dev.azure.com/myorg/BC-Knowledge/_git/team-alpha",
        "branch": "main",
        "auth": {
          "type": "az_cli"
        }
      },
      "enabled": true
    },
    {
      "name": "Project Overrides",
      "type": "project",
      "priority": 100,
      "source": {
        "path": "./bc-code-intel-overrides"
      },
      "enabled": true
    }
  ],
  "enable_diagnostic_tools": true
}
```

**YAML:**
```yaml
knowledge_layers:
  - name: "Company Standards"
    type: git
    priority: 20
    source:
      repository_url: "https://dev.azure.com/myorg/BC-Knowledge/_git/standards"
      branch: main
      auth:
        type: az_cli
    enabled: true

  - name: "Team Conventions"
    type: git
    priority: 50
    source:
      repository_url: "https://dev.azure.com/myorg/BC-Knowledge/_git/team-alpha"
      branch: main
      auth:
        type: az_cli
    enabled: true

  - name: "Project Overrides"
    type: project
    priority: 100
    source:
      path: "./bc-code-intel-overrides"
    enabled: true

enable_diagnostic_tools: true
```

**What this does:** Complete organization setup with company, team, and project layers using Azure CLI authentication.

## Configuration Schema Reference

### **Root Configuration Object**

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `knowledge_layers` | Array | No | `[]` | Array of knowledge layer configurations |
| `enable_diagnostic_tools` | Boolean | No | `false` | Enable advanced diagnostic MCP tools |
| `bc_version` | String | No | Auto-detect | Target BC version for filtering (e.g., "22", "21") |

### **Knowledge Layer Object**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | String | Yes | Human-readable layer name |
| `type` | String | Yes | Layer type: `"git"` or `"project"` |
| `priority` | Number | Yes | Layer priority (0-100, higher wins) |
| `source` | Object | Yes | Source configuration (varies by type) |
| `enabled` | Boolean | No (default: `true`) | Whether layer is active |

### **Git Source Configuration**

```typescript
{
  "repository_url": string,     // Git repository URL
  "branch": string,             // Git branch (default: "main")
  "subdirectory"?: string,      // Optional subdirectory path
  "auth": {
    // Option 1: Azure CLI (recommended for Azure DevOps)
    "type": "az_cli"
    
    // Option 2: PAT token (fallback or for GitHub)
    // "type": "pat",
    // "token_env_var": string   // Environment variable containing PAT token
  }
}
```

### **Project Source Configuration**

```typescript
{
  "path": string                // Relative or absolute path (default: "./bc-code-intel-overrides")
}
```

## Layer Priority Guidelines

**Standard Priority Ranges:**
- **0**: Embedded knowledge (reserved - always present)
- **10-30**: Company-wide standards
- **40-60**: Team or department conventions
- **70-90**: Sub-team or project-type specific
- **100**: Project-local overrides (highest priority)

**Why priority matters:** When the same topic/specialist exists in multiple layers, only the highest-priority version is used.

## Authentication Configuration

### **Azure DevOps Authentication** ⭐ **Recommended**

#### **Option 1: Azure CLI (Preferred)**

**Why prefer Azure CLI:**
- ✅ No token expiration management
- ✅ Works with MFA and conditional access policies
- ✅ Single sign-on via `az login`
- ✅ Simpler than PAT management

**Setup:**

1. **Install Azure CLI:** https://aka.ms/install-az-cli

2. **Login to Azure:**
   ```bash
   az login
   ```

3. **Configure in JSON:**
   ```json
   "auth": {
     "type": "az_cli"
   }
   ```

4. **Configure in YAML:**
   ```yaml
   auth:
     type: az_cli
   ```

**Full example:**
```json
{
  "knowledge_layers": [
    {
      "name": "Company Standards",
      "type": "git",
      "priority": 20,
      "source": {
        "repository_url": "https://dev.azure.com/myorg/BC-Knowledge/_git/standards",
        "branch": "main",
        "auth": {
          "type": "az_cli"
        }
      },
      "enabled": true
    }
  ]
}
```

**How it works:** Git credential manager automatically uses your Azure CLI session for authentication. No manual token handling required!

#### **Option 2: Azure DevOps PAT Token (Fallback)**

Use when Azure CLI is not available or when automation requires static credentials.

1. **Generate token:** Azure DevOps → User Settings → Personal Access Tokens
   - Scopes: `Code (Read)`

2. **Set environment variable:**
   ```bash
   # Windows PowerShell
   $env:AZURE_DEVOPS_PAT = "your_pat_token_here"
   
   # Linux/macOS
   export AZURE_DEVOPS_PAT="your_pat_token_here"
   ```

3. **Reference in config:**
   ```json
   "auth": {
     "type": "pat",
     "token_env_var": "AZURE_DEVOPS_PAT"
   }
   ```

### **GitHub Authentication**

#### **GitHub PAT Token**

1. **Generate token:** https://github.com/settings/tokens
   - Permissions: `repo` (for private repos) or no permissions (public repos)

2. **Set environment variable:**
   ```bash
   # Windows PowerShell
   $env:GITHUB_TOKEN = "ghp_your_token_here"
   
   # Linux/macOS
   export GITHUB_TOKEN="ghp_your_token_here"
   ```

3. **Reference in config:**
   ```json
   "auth": {
     "type": "pat",
     "token_env_var": "GITHUB_TOKEN"
   }
   ```

## Creating Configuration Files

### **Method 1: Manual Creation**

**Windows:**
```powershell
# Create in workspace
New-Item -Path "bc-code-intel-config.json" -ItemType File

# Open in editor
notepad bc-code-intel-config.json
```

**Linux/macOS:**
```bash
# Create in workspace
touch bc-code-intel-config.json

# Open in editor
code bc-code-intel-config.json
```

### **Method 2: From Template**

Copy from the MCP repo examples:

```bash
# Find example configs in MCP repo
ls bc-code-intelligence-mcp/bc-code-intel-config.example.*

# Copy and customize
cp bc-code-intel-config.example.json bc-code-intel-config.json
```

### **Method 3: Programmatic Generation**

**PowerShell:**
```powershell
$config = @{
    knowledge_layers = @(
        @{
            name = "Company Standards"
            type = "git"
            priority = 20
            source = @{
                repository_url = "https://dev.azure.com/myorg/BC-Knowledge/_git/standards"
                branch = "main"
                auth = @{
                    type = "az_cli"
                }
            }
            enabled = $true
        }
    )
    enable_diagnostic_tools = $false
}

$config | ConvertTo-Json -Depth 10 | Out-File "bc-code-intel-config.json"
```

**Bash/Node.js:**
```bash
cat > bc-code-intel-config.json << 'EOF'
{
  "knowledge_layers": [
    {
      "name": "Company Standards",
      "type": "git",
      "priority": 20,
      "source": {
        "repository_url": "https://dev.azure.com/myorg/BC-Knowledge/_git/standards",
        "branch": "main",
        "auth": {
          "type": "az_cli"
        }
      },
      "enabled": true
    }
  ]
}
EOF
```

## Configuration File Locations

See [Configuration File Discovery](configuration-file-discovery.md) for details on where to place config files.

**Quick reference:**
1. **Workspace:** `./bc-code-intel-config.{json|yaml|yml}` (highest priority)
2. **Home Directory:** `~/.bc-code-intel/config.{json|yaml|yml}`
3. **Environment Variable:** `BC_CODE_INTEL_CONFIG=/path/to/config.json`

## Validation and Testing

### **Validate JSON Syntax**

**Online:** Use https://jsonlint.com/

**Command Line:**
```bash
# Node.js
node -e "JSON.parse(require('fs').readFileSync('bc-code-intel-config.json', 'utf8'))"

# Python
python -m json.tool bc-code-intel-config.json

# jq
jq empty bc-code-intel-config.json
```

### **Validate YAML Syntax**

```bash
# Python
python -c "import yaml; yaml.safe_load(open('bc-code-intel-config.yaml'))"

# yamllint (install via pip)
yamllint bc-code-intel-config.yaml
```

### **Test Configuration Loading**

Enable diagnostic tools and use layer diagnostic MCP tools:

```json
{
  "enable_diagnostic_tools": true,
  "knowledge_layers": [...]
}
```

Then query layer status via MCP tools to verify layers loaded correctly.

## Common Configuration Patterns

### **Pattern 1: Individual Developer (Zero Config)**
```json
{}
```
Uses embedded knowledge only.

### **Pattern 2: Company Standards Only**
```json
{
  "knowledge_layers": [
    {
      "name": "Company",
      "type": "git",
      "priority": 20,
      "source": {
        "repository_url": "https://github.com/company/bc-standards",
        "branch": "main"
      },
      "enabled": true
    }
  ]
}
```

### **Pattern 3: Multiple Teams with Project Overrides**
```json
{
  "knowledge_layers": [
    {
      "name": "Company",
      "type": "git",
      "priority": 20,
      "source": { "repository_url": "..." },
      "enabled": true
    },
    {
      "name": "Team Alpha",
      "type": "git",
      "priority": 50,
      "source": { "repository_url": "..." },
      "enabled": true
    },
    {
      "name": "Project",
      "type": "project",
      "priority": 100,
      "source": { "path": "./bc-code-intel-overrides" },
      "enabled": true
    }
  ]
}
```

### **Pattern 4: Temporary Layer Disable**
```json
{
  "knowledge_layers": [
    {
      "name": "Experimental Team Layer",
      "type": "git",
      "priority": 50,
      "source": { "repository_url": "..." },
      "enabled": false  // <-- Temporarily disabled
    }
  ]
}
```

## Troubleshooting

### **"Configuration file not found"**
- Check file name exactly: `bc-code-intel-config.json` (not `.config.json`)
- Verify file in correct location (workspace root or home directory)
- Use `get_workspace_root` to check current working directory

### **"Invalid JSON/YAML syntax"**
- Validate with tools mentioned above
- Common issues: trailing commas (JSON), incorrect indentation (YAML)
- Use example configs as templates

### **"Git layer not loading"**
- Verify repository URL is accessible
- Check authentication token is set in environment variable
- Ensure `token_env_var` name matches actual environment variable
- Test git clone manually: `git clone <repository_url>`

### **"Project layer showing 0 topics"**
- Verify directory structure: `bc-code-intel-overrides/domains/`
- Check YAML frontmatter in topic files
- Ensure `path` in config matches actual directory

## Best Practices

1. **Start Simple**: Begin with zero config or single git layer
2. **Use YAML for Complex Setups**: Better readability for multi-layer configs
3. **Version Control**: Commit config files to project repos (except tokens!)
4. **Environment Variables**: Never hardcode tokens in config files
5. **Test Incrementally**: Add one layer at a time, validate before adding next
6. **Document Decisions**: Add comments in YAML or separate README explaining layer purposes

## See Also

- [Configuration File Discovery](configuration-file-discovery.md) - Where configs are found
- [Layer System Fundamentals](layer-system-fundamentals.md) - Understanding layer architecture
- [Git Layer Configuration](git-layer-configuration.md) - Git-specific setup details
- [Multi-Team Layer Configuration](multi-team-layer-configuration.md) - Organization-wide setups
