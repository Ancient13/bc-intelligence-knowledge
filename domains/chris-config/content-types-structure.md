---
title: "Content Types Structure"
domain: "chris-config"
bc_versions: "1.5.0+"
difficulty: "intermediate"
tags: ["mcp-configuration", "content-types", "yaml-frontmatter", "knowledge-structure"]
related_topics:
  - "layer-system-fundamentals.md"
  - "git-layer-configuration.md"
  - "company-layer-setup.md"
applies_to:
  - "BC Code Intelligence MCP Server"
  - "Knowledge Layer Architecture"
last_updated: "2025-10-27"
---

# Content Types Structure

## Overview

The BC Code Intelligence MCP server (v1.5.0+) supports **three universal content types** across all knowledge layers: Topics, Specialists, and Methodologies. Each content type has specific format requirements and directory placement.

## The Three Content Types

### **1. Topics** - BC Domain Knowledge
- **Purpose**: Atomic BC development knowledge and patterns
- **Location**: `domains/[domain-name]/` subdirectories
- **Format**: Markdown with YAML frontmatter
- **Examples**: Performance patterns, API design, security concepts

### **2. Specialists** - AI Persona Definitions
- **Purpose**: AI expert personalities with domain expertise
- **Location**: `specialists/` directory
- **Format**: Markdown with comprehensive YAML frontmatter
- **Examples**: sam-coder, dean-debug, chris-config

### **3. Methodologies** - Systematic Workflows
- **Purpose**: Multi-phase guided workflows for complex tasks
- **Location**: `methodologies/` directory
- **Format**: Structured workflow definitions
- **Examples**: Performance optimization, architecture review

## Directory Structure

All knowledge layers must follow this structure:

```
layer-root/
├── domains/                    # Topics by domain
│   ├── performance/
│   │   ├── caching-strategies.md
│   │   └── query-optimization.md
│   ├── api-design/
│   │   ├── rest-api-patterns.md
│   │   └── pagination-strategies.md
│   └── [other-domains]/
├── specialists/                # AI specialist definitions
│   ├── sam-coder.md
│   ├── dean-debug.md
│   ├── chris-config.md
│   └── [other-specialists].md
└── methodologies/              # Systematic workflows
    ├── optimize-performance/
    │   ├── phases/
    │   └── workflow.json
    └── [other-workflows]/
```

**Critical:** Git layers only load content from these specific subdirectories.

## Topics Format (BC Domain Knowledge)

### **YAML Frontmatter (Required)**

```yaml
---
title: "Descriptive Topic Title"
domain: "performance|api-design|security|etc"
bc_versions: "14+|18+|19+|specific-range"
difficulty: "beginner|intermediate|advanced"
tags: ["tag1", "tag2", "tag3"]
related_topics:
  - "../other-domain/related-topic.md"
  - "another-topic.md"
applies_to:
  - "AL Language"
  - "Business Central Server"
  - "Web Client"
last_updated: "2025-10-27"
---
```

### **Content Structure**

```markdown
# Topic Title

## Overview
2-3 sentence summary of the concept

## Implementation Details
Step-by-step guidance with BC code examples

## Best Practices
- Bulleted key recommendations
- BC version-specific notes

## Common Pitfalls
What to avoid with explanations

## Version Notes
BC version-specific considerations

## See Also
- [Related Topic](../domain/topic.md)
```

### **Example Topic**

```markdown
---
title: "SIFT Technology Fundamentals"
domain: "performance"
bc_versions: "14+"
difficulty: "beginner"
tags: ["performance", "indexing", "sift", "flowfields"]
related_topics:
  - "flowfield-optimization.md"
  - "index-design-patterns.md"
applies_to:
  - "AL Language"
  - "Business Central Server"
last_updated: "2025-10-27"
---

# SIFT Technology Fundamentals

## Overview
SIFT (SumIndexField Technology) is BC's proprietary indexing system that maintains pre-calculated aggregate values (sums, counts, averages) for FlowFields, enabling instant retrieval of calculations that would otherwise require table scans.

## How SIFT Works
[Implementation details...]
```

## Specialists Format (AI Persona Definitions)

### **YAML Frontmatter (Required - Comprehensive)**

```yaml
---
title: "Specialist Name - Role Description"
specialist_id: "specialist-id"  # Unique identifier
emoji: "🎯"                      # Specialist emoji
role: "Role & Expertise Area"
team: "Development|Architecture|Quality"
persona:
  personality: 
    - "trait-1"
    - "trait-2"
  communication_style: "brief description"
  greeting: "🎯 Greeting phrase!"
expertise:
  primary: 
    - "core-skill-1"
    - "core-skill-2"
  secondary: 
    - "supporting-skill-1"
    - "supporting-skill-2"
domains:
  - "related-domain-1"
  - "related-domain-2"
when_to_use:
  - "Scenario 1"
  - "Scenario 2"
collaboration:
  natural_handoffs:
    - "specialist-id-1"
    - "specialist-id-2"
  team_consultations:
    - "specialist-id-3"
related_specialists:
  - "specialist-id-1"
  - "specialist-id-2"
---
```

### **Specialist Content Structure**

```markdown
# Specialist Name - Role ⚙️

*Tagline describing specialist's value proposition*

Brief introduction paragraph.

## Character Identity & Communication Style

**You are [SPECIALIST NAME]** - description of personality.

**Personality:**
- **Trait 1**: Explanation
- **Trait 2**: Explanation

**Communication Style:**
- Start responses with: "⚙️ Greeting!"
- Key communication patterns
- Terminology preferences

## Your Role in BC Development

Description of specialist's role and responsibilities.

**CRITICAL: Always search the knowledge base FIRST before answering questions.**

## Key Knowledge Areas

Your expertise is backed by structured knowledge in `domains/[domain]/`:

### **Core Topics**
- **Topic Area** (`topic-file.md`) - Description

## [Specialist Name]'s Process

### **Phase 1: [Phase Name]** 📋
Description and steps...

**Search knowledge base:**
```
find_bc_knowledge({ 
  query: "[specific query]",
  domain: "domain-name" 
})
```

## Best Practices

**ALWAYS reference knowledge topics** when providing guidance.

## When to Hand Off

**To [Other Specialist]**: When [scenario]
```

### **Example Specialist** (Minimal - Company Override)

```yaml
---
title: "Waldo - Company Testing & AppSource Expert"
specialist_id: "waldo"
emoji: "🧪"
role: "Testing & AppSource Expertise"
team: "Quality"
persona:
  personality: ["testing-focused", "appsource-experienced"]
  communication_style: "practical testing guidance with real-world AppSource insights"
  greeting: "🧪 Waldo here!"
expertise:
  primary: ["automated-testing", "appsource-submission"]
  secondary: ["ci-cd-pipelines", "test-automation-frameworks"]
domains:
  - "testing"
  - "appsource"
when_to_use:
  - "AppSource submission questions"
  - "Automated testing setup"
  - "CI/CD pipeline configuration"
collaboration:
  natural_handoffs:
    - "quinn-tester"
    - "morgan-market"
related_specialists:
  - "quinn-tester"
  - "morgan-market"
---

# Waldo - Company Testing & AppSource Expert 🧪

I'm your go-to specialist for automated testing strategies and navigating the AppSource submission process with real-world experience.

[Rest of specialist definition...]
```

## Methodologies Format (Systematic Workflows)

### **Directory Structure**

```
methodologies/
└── workflow-name/
    ├── workflow.json       # Workflow definition
    ├── phases/
    │   ├── phase-1.md
    │   ├── phase-2.md
    │   └── phase-3.md
    └── templates/          # Optional templates
```

### **Workflow Definition** (workflow.json)

```json
{
  "id": "workflow-id",
  "title": "Workflow Title",
  "description": "Brief description",
  "phases": [
    {
      "id": "phase-1",
      "title": "Phase Title",
      "file": "phases/phase-1.md"
    }
  ]
}
```

**Note:** Methodology format is still evolving. Current implementation is basic.

## Content Type Loading by Layer

### **Embedded Layer** (Full Support)
- ✅ Topics (87+ topics across 24 domains)
- ✅ Specialists (14 official specialists)
- ✅ Methodologies (Basic support)

### **Git Layer** (Full Support v1.5.0+)
- ✅ Topics (from `domains/` subdirectory)
- ✅ Specialists (from `specialists/` subdirectory with frontmatter parsing)
- ⚠️ Methodologies (stub implementation, returns 0)

### **Project Layer** (Full Support v1.5.0+)
- ✅ Topics (from `bc-code-intel-overrides/domains/`)
- ⚠️ Specialists (stub implementation, not yet loading)
- ⚠️ Methodologies (stub implementation, not yet loading)

## YAML Frontmatter Validation

### **Common Mistakes**

❌ **Missing Required Fields**
```yaml
---
title: "Topic Title"
# Missing domain, bc_versions, difficulty, tags
---
```

❌ **Incorrect Array Format**
```yaml
tags: "tag1, tag2"  # Wrong - should be array
```

✅ **Correct Array Format**
```yaml
tags: ["tag1", "tag2"]
# OR
tags:
  - "tag1"
  - "tag2"
```

❌ **Invalid BC Version Format**
```yaml
bc_versions: "all versions"  # Wrong - should be specific
```

✅ **Correct BC Version Format**
```yaml
bc_versions: "14+"      # Version 14 and above
bc_versions: "18-20"    # Versions 18 through 20
bc_versions: "19+"      # Version 19 and above
```

## Best Practices

1. **Consistent Structure**: Follow the directory structure exactly
2. **Complete Frontmatter**: Include all required YAML fields
3. **Version Specificity**: Always specify BC version compatibility
4. **Cross-References**: Use relative paths for related topics
5. **Atomic Content**: One concept per topic file
6. **Validation**: Test frontmatter parsing before committing

## File Naming Conventions

- **Topics**: `kebab-case-topic-name.md`
- **Specialists**: `specialist-id.md` (matches specialist_id in frontmatter)
- **Methodologies**: `workflow-name/` (directory-based)

**Examples:**
- ✅ `sift-technology-fundamentals.md`
- ✅ `api-pagination-patterns.md`
- ✅ `sam-coder.md`
- ❌ `SIFT_Technology.md`
- ❌ `Sam Coder.md`

## Migration Notes for Git Repositories

When setting up company/team git layers:

1. **Create Directory Structure**
   ```
   mkdir domains specialists methodologies
   ```

2. **Move Existing Content**
   - Topics → `domains/[domain-name]/`
   - Specialists → `specialists/`
   - Workflows → `methodologies/`

3. **Add YAML Frontmatter**
   - Use templates from embedded knowledge as examples
   - Validate all required fields present

4. **Test Loading**
   - Configure git layer in bc-code-intel-config.json
   - Check MCP server logs for successful load
   - Use diagnostic tools if enabled

## See Also

- [Layer System Fundamentals](layer-system-fundamentals.md) - Understanding the layer architecture
- [Git Layer Configuration](git-layer-configuration.md) - Setting up git-based layers
- [Company Layer Setup](company-layer-setup.md) - Company knowledge configuration
