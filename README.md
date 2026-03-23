# AI Nutritionist (Sage)

A [Kiro CLI](https://kiro.dev) agent that creates weekly meal plans, recipes, and grocery lists tailored to your household. Uses Obsidian for note storage and the Edamam API for recipe search.

## Setup

### 1. Prerequisites
- [Kiro CLI](https://kiro.dev)
- An Obsidian vault for storing meal plans
- [Edamam API](https://developer.edamam.com/) credentials (free tier)

### 2. Configure environment

Add to your shell profile (`~/.zshrc`, `~/.bashrc`, etc.):

```bash
export OBSIDIAN_VAULT_PATH="/path/to/your/obsidian/vault"
export EDAMAM_APP_ID="your-app-id"
export EDAMAM_APP_KEY="your-app-key"
```

### 3. Update the MCP server path

In `.kiro/agents/nutritionist.json`, the Obsidian MCP server arg references `$OBSIDIAN_VAULT_PATH`. Replace this with your actual vault path, or set the env var above.

### 4. First run

Switch to the agent in Kiro CLI:

```
/agent nutritionist
```

Sage will walk you through creating your `CONFIG.md` with your household preferences, dietary restrictions, and goals. See `CONFIG.md.example` for a sample.
