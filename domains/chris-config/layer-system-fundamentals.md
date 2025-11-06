---
title: "Layer System Fundamentals"
domain: "chris-config"
bc_versions: "1.5.0+"
difficulty: "intermediate"
tags: ["mcp-configuration", "layer-architecture", "knowledge-management", "override-system"]
related_topics:
  - "content-types-structure.md"
  - "layer-override-strategies.md"
  - "configuration-file-discovery.md"
applies_to:
  - "BC Code Intelligence MCP Server"
  - "Knowledge Layer Architecture"
last_updated: "2025-10-27"
---

# Layer System Fundamentals

## Overview

The BC Code Intelligence MCP server uses a **4-layer knowledge architecture** that enables progressive customization from embedded defaults through company-wide standards to project-specific overrides. Understanding this layered approach is fundamental to effective MCP configuration.

## The Four-Layer Architecture

### **Layer 0: Embedded Knowledge** (Priority 0)
- **Source**: NPM package embedded knowledge (git submodule)
- **Content**: Core BC knowledge - 87+ topics, 14 specialists, methodologies
- **Scope**: Universal - available to all users
- **Modifiable**: No - read-only embedded content
- **Use Case**: Default knowledge base, zero-configuration experience

**Key Characteristics:**
- Highest reliability - always available
- Officially maintained BC knowledge
- Comprehensive coverage of BC development patterns
- Updates via npm package updates

### **Layer 1-2: Company Knowledge** (Priority 20-50)
- **Source**: Git repositories (Azure DevOps, GitHub, etc.)
- **Content**: Company-specific specialists, standards, patterns
- **Scope**: Organization-wide
- **Modifiable**: Yes - via git repository updates
- **Use Case**: Company coding standards, custom specialists, org patterns

**Key Characteristics:**
- Centrally managed company knowledge
- Authentication-based access control (PAT tokens)
- Cached locally for performance (`.bckb-cache/git-repos/`)
- Can override embedded knowledge with company-specific guidance

### **Layer 3: Team Knowledge** (Priority 60-80)
- **Source**: Shared team directories or repositories
- **Content**: Team conventions, project templates, shared patterns
- **Scope**: Team or department
- **Modifiable**: Yes - team-managed
- **Use Case**: Team-specific patterns, shared project standards

**Key Characteristics:**
- Team collaboration and knowledge sharing
- More specific than company, less specific than project
- Useful for department or team conventions

### **Layer 4: Project Knowledge** (Priority 100)
- **Source**: Local `./bc-code-intel-overrides/` directory
- **Content**: Project-specific overrides, custom topics, local specialists
- **Scope**: Single project/workspace
- **Modifiable**: Yes - locally managed
- **Use Case**: Project-specific patterns, temporary overrides, local experimentation

**Key Characteristics:**
- Highest priority - always wins in conflicts
- Local filesystem access (no authentication needed)
- Perfect for project-specific customization
- Useful for testing new knowledge before promoting to higher layers

## Layer Resolution and Override Behavior

### **Priority-Based Resolution**

When the same topic/specialist exists in multiple layers:
1. **Highest priority layer wins** (Project > Team > Company > Embedded)
2. Content is **completely replaced**, not merged
3. Only the highest-priority version is returned

**Example:**
```
Embedded Layer (0): sam-coder.md (official specialist)
Company Layer (20): sam-coder.md (company-customized version)
Project Layer (100): sam-coder.md (project-specific override)

Result: Project layer version is used
```

### **Content Type Support**

All layers support three content types (as of v1.5.0):
- **Topics**: BC domain knowledge (in `domains/` subdirectory)
- **Specialists**: AI persona definitions (in `specialists/` subdirectory)
- **Methodologies**: Systematic workflows (in `methodologies/` subdirectory)

Each layer independently manages these three content types.

## Layer Configuration

### **Embedded Layer** (No Configuration Needed)
```json
// Automatically available - no configuration required
// Loaded from NPM package embedded-knowledge/ directory
```

### **Git Layer** (Company/Team Knowledge)
```json
{
  "knowledge_layers": [
    {
      "name": "Company BC Standards",
      "type": "git",
      "priority": 20,
      "source": {
        "repository_url": "https://dev.azure.com/your-org/BC-Knowledge/_git/bc-standards",
        "branch": "main",
        "auth": {
          "type": "pat",
          "token_env_var": "AZURE_DEVOPS_PAT"
        }
      }
    }
  ]
}
```

### **Project Layer** (Local Overrides)
```json
{
  "knowledge_layers": [
    {
      "name": "Project Overrides",
      "type": "project",
      "priority": 100,
      "source": {
        "path": "./bc-code-intel-overrides"
      }
    }
  ]
}
```

## Directory Structure Requirements

Each layer must follow this structure for content type discovery:

```
layer-root/
├── domains/              # BC knowledge topics
│   ├── performance/
│   ├── api-design/
│   └── [other domains]/
├── specialists/          # AI specialist definitions
│   ├── sam-coder.md
│   ├── johan-debug.md
│   └── [other specialists].md
└── methodologies/        # Systematic workflows
    ├── optimize-performance/
    └── [other workflows]/
```

**Important:** Git layers look for these subdirectories. If missing, that content type won't be loaded from that layer.

## Workspace Management and Layer Loading

### **VS Code Workspace Detection Issue**

The VS Code MCP extension doesn't set `process.cwd()` to the workspace root, causing project layer detection to fail.

**Solution:** Use workspace management tools (v1.5.0+)

```typescript
// Set workspace root explicitly
set_workspace_root({ 
  workspace_root: "/path/to/your/workspace" 
})

// Triggers:
// 1. process.chdir() to workspace
// 2. Full service reinitialization
// 3. Layer reloading with new workspace context
```

### **Lazy Initialization Pattern**

The MCP server uses lazy initialization:
1. **Startup**: Load embedded knowledge only (fast startup)
2. **First Tool Call**: If workspace not set, prompt for `set_workspace_root`
3. **After Workspace Set**: Full initialization including all configured layers

## Layer Caching and Performance

### **Git Layer Caching**

Git repositories are cloned to `.bckb-cache/git-repos/{hash}/`:
- **First Load**: Clone repository (may take seconds)
- **Subsequent Loads**: Use cached local copy (sub-100ms)
- **Updates**: Periodic pull or manual cache invalidation

### **Performance Characteristics**
- Embedded Layer: <10ms (in-memory)
- Project Layer: <50ms (filesystem access)
- Git Layer (cached): <100ms (filesystem + parsing)
- Git Layer (uncached): 2-5 seconds (git clone operation)

## Best Practices

1. **Start Simple**: Use embedded knowledge first, add layers as needed
2. **Layer Discipline**: Keep clear separation between layer purposes
3. **Override Sparingly**: Only override when truly necessary for your context
4. **Document Decisions**: Each layer should have clear purpose documentation
5. **Test Incrementally**: Add one layer at a time, validate before adding next
6. **Cache Management**: Understand git layer caching for performance tuning

## Common Patterns

### **Individual Developer** (Zero Config)
```
Embedded Layer only - no configuration needed
```

### **Company with Standards** (Git Layer)
```
Embedded Layer (BC knowledge) + Company Git Layer (org standards)
```

### **Team with Project Overrides** (All Layers)
```
Embedded + Company Git + Project Local
```

## Troubleshooting

**Layer not loading?**
- Check directory structure (domains/, specialists/, methodologies/)
- Verify authentication (PAT tokens for git layers)
- Check layer priority configuration
- Use diagnostic tools if enabled

**Content not found?**
- Verify content type is in correct subdirectory
- Check YAML frontmatter format (required for specialists)
- Confirm layer priority order
- Search across all layers to see where content exists

## See Also

- [Content Types Structure](content-types-structure.md) - Format requirements for each content type
- [Layer Override Strategies](layer-override-strategies.md) - When and how to override
- [Configuration File Discovery](configuration-file-discovery.md) - Where config files are found
- [Workspace Detection Solutions](workspace-detection-solutions.md) - Solving workspace issues
- [Git Layer Configuration](git-layer-configuration.md) - Git layer setup details
