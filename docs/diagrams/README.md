# Freemium Platform Diagrams

This directory contains all architectural and flow diagrams for the Freemium platform.

## Available Diagrams

### 📊 **System Architecture**
- **File**: [system_architecture.md](./system_architecture.md)
- **Purpose**: High-level system architecture showing all platform components
- **Audience**: Technical stakeholders, architects, developers
- **Referenced in**: PRD Section 4 (Technical Requirements)

### 🔄 **Data Flow**  
- **File**: [flow_v1.md](./flow_v1.md)
- **Purpose**: End-to-end data flow from ingestion to user interaction
- **Audience**: Product managers, developers, QA engineers
- **Referenced in**: PRD Section 1 (Overview), Todo items

### 🔗 **CorTenX Integration Flow**
- **File**: [cortenx_integration_flow.md](./cortenx_integration_flow.md)
- **Purpose**: Sequence diagrams for asset creation and blockchain synchronization
- **Audience**: Backend developers, integration engineers
- **Referenced in**: `specs/cortenx_integration.md`

## Diagram Standards

### **Mermaid Format**
All diagrams use Mermaid DSL for:
- Version control friendly (text-based)
- Easy editing and collaboration
- Automatic rendering in GitHub/VS Code
- Export capabilities (SVG/PNG)

### **Naming Convention**
- `{diagram_type}_{version}.md` (e.g., `flow_v1.md`)
- Include version number for major changes
- Use descriptive filenames

### **File Structure**
Each diagram file should include:
```markdown
# {Diagram Title} v{Version}

## Overview
Brief description of what the diagram shows

## {Diagram Type} Diagram
```mermaid
[diagram content]
```

## Key Components
- Description of major components
- Relationships and data flow
- Technology choices

## Status
- Version, creation date, owner, review status
```

### **Color Scheme**
Consistent styling across diagrams:
- **CorTenX/Blockchain**: `#ff9999` (light red)
- **MCP Server**: `#e8f5e8` (light green) 
- **UI Components**: `#f3e5f5` (light purple)
- **Reports/Output**: `#fff2cc` (light yellow)
- **Deployment**: `#e1f5fe` (light blue)

## Usage in Documentation

### **PRD References**
Link to diagrams using relative paths:
```markdown
📊 **[System Architecture Diagram](diagrams/system_architecture.md)**
🔄 **[Data Flow Diagram](diagrams/flow_v1.md)**
```

### **Specifications References**
Technical specs can reference specific diagram sections:
```markdown
See [System Architecture - API Layer](../diagrams/system_architecture.md#api-layer)
```

## Maintenance

### **Version Control**
- Create new version files for major changes (e.g., `flow_v2.md`)
- Update references in documentation when versions change
- Archive old versions with clear status

### **Review Process**
1. Create diagram in drafts/ subdirectory
2. Technical review by architecture team
3. Move to main diagrams/ directory
4. Update documentation references

## Export Options

### **For Presentations**
```bash
# Export to SVG for presentations
mmdc -i system_architecture.md -o system_architecture.svg

# Export to PNG for documents  
mmdc -i flow_v1.md -o flow_v1.png
```

### **For Documentation**
- GitHub automatically renders Mermaid in .md files
- VS Code Mermaid extension for live preview
- Mermaid Live Editor for online editing

## Status
- **Last Updated**: 2025-06-14
- **Owner**: James  
- **Review Status**: Consolidated from embedded diagrams 