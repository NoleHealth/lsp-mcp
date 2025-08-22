# LSP-MCP Local Setup

This is a local installation of [lsp-mcp](https://github.com/jonrad/lsp-mcp) configured to work with Claude Desktop via stdio transport.

## Overview

LSP-MCP provides Language Server Protocol (LSP) capabilities to LLMs/AI Agents, giving Claude access to language-aware context from codebases including:

- Document symbols
- Diagnostics
- Hover information
- Code completion context
- And more LSP features

## Installation

This setup has been configured with:

- ✅ lsp-mcp built from source
- ✅ TypeScript/JavaScript language server (`typescript-language-server`)
- ✅ Python language server (`pylsp`)
- ✅ Project-specific configuration files

## Project Configuration

### For Project-Specific Setup (Recommended)

Each project should have its own LSP configuration. Here's how to set it up using the `aro_poc` monorepo as an example:

#### 1. Create LSP Configuration Directory

```bash
mkdir /home/nole/dev/solutions/dream-weaver/aro_poc/.lsp-mcp
```

#### 2. Create Project LSP Config

Create `/home/nole/dev/solutions/dream-weaver/aro_poc/.lsp-mcp/lsp.config.json`:

```json
{
  "lsps": [
    {
      "id": "typescript",
      "extensions": ["ts", "tsx", "js", "jsx"],
      "languages": ["typescript", "javascript"],
      "command": "typescript-language-server",
      "args": ["--stdio"]
    },
    {
      "id": "python",
      "extensions": ["py"],
      "languages": ["python", "python2", "python3"],
      "command": "pylsp",
      "args": []
    }
  ],
  "workspace": "./apps"
}
```

**Key Point**: The `workspace` field points to where your actual code lives. For monorepos with code in subdirectories (like `apps/frontend`, `apps/backend`, `apps/cli`), set it to `"./apps"`.

#### 3. Configure MCP in Project

Add to your project's `.mcp.json`:

```json
{
  "mcpServers": {
    "lsp": {
      "command": "node",
      "args": [
        "/home/nole/dev/shared/mcp-servers/lsp-mcp/dist/index.js",
        "--config",
        "./.lsp-mcp/lsp.config.json"
      ],
      "cwd": "/home/nole/dev/solutions/dream-weaver/aro_poc"
    }
  }
}
```

### For Global Setup (Alternative)

Add this to your global `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "lsp": {
      "command": "node",
      "args": [
        "/home/nole/dev/shared/mcp-servers/lsp-mcp/dist/index.js",
        "--config",
        "/home/nole/dev/shared/mcp-servers/lsp-mcp/local.config.json"
      ]
    }
  }
}
```

## Supported Languages

### Currently Configured:

- **TypeScript/JavaScript** (.ts, .tsx, .js, .jsx)
- **Python** (.py)

### Adding More Language Servers

To add support for additional languages:

1. Install the language server globally or locally
2. Add configuration to `local.config.json`:

```json
{
  "lsps": [
    // ... existing configs
    {
      "id": "rust",
      "extensions": ["rs"],
      "languages": ["rust"],
      "command": "rust-analyzer",
      "args": []
    }
  ]
}
```

### Popular Language Servers:

- **Rust**: `rust-analyzer`
- **Go**: `gopls`
- **Java**: `jdtls`
- **C/C++**: `clangd`
- **PHP**: `intelephense`

## Usage

Once configured, you can ask Claude to analyze code by:

1. Mentioning specific files or code snippets
2. Asking for language-specific insights
3. Requesting code analysis or symbol information

### Examples with aro_poc Monorepo:

```bash
# Frontend TypeScript Analysis
"Use LSP to analyze the React components in apps/frontend/src/components/"
"Show me TypeScript interfaces used in the NotificationManager"
"Get document symbols for the ErrorCategorizer component"

# Backend Python Analysis
"Use LSP to analyze the FastAPI endpoints in apps/backend/api/"
"Show me class definitions in the lineage services"
"Check Python imports and dependencies across the backend"

# Cross-Package Analysis
"Use LSP to find all references to 'Environment' across the monorepo"
"Show me type definitions shared between frontend and backend"
"Analyze the API contract between frontend and backend services"
```

### Monorepo Project Discovery

The LSP will automatically discover projects within the workspace:

- **TypeScript projects**: Found via `tsconfig.json`, `package.json` files
- **Python projects**: Found via `pyproject.toml`, `setup.py` files

For the aro_poc example:

- `apps/frontend/` → TypeScript project (React/Vite)
- `apps/backend/` → Python project (FastAPI)
- `apps/cli/` → Python project (CLI tool)

## Troubleshooting

### Language Server Not Found

If you get "command not found" errors:

- Ensure the language server is installed and in your PATH
- For npm packages: `npm list -g` to verify installation
- For system packages: `which <command>` to check availability

### Permission Issues

- Language servers need read access to your source files
- Ensure Claude Desktop has appropriate file permissions

### Debug Mode

You can run lsp-mcp manually for debugging:

**Global config:**

```bash
node /home/nole/dev/shared/mcp-servers/lsp-mcp/dist/index.js --config /home/nole/dev/shared/mcp-servers/lsp-mcp/local.config.json
```

**Project-specific config:**

```bash
cd /home/nole/dev/solutions/dream-weaver/aro_poc
node /home/nole/dev/shared/mcp-servers/lsp-mcp/dist/index.js --config ./.lsp-mcp/lsp.config.json
```

## Updates

To update lsp-mcp:

```bash
cd /home/nole/dev/shared/mcp-servers/lsp-mcp
git pull
yarn install
yarn build
```

## Architecture

This setup uses:

- **Transport**: stdio (direct communication with Claude Desktop)
- **Language Servers**: Installed locally on host system
- **File Access**: Direct filesystem access to all projects
- **Network**: No network requirements (all local)
