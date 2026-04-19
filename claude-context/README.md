# claude-context

Code search MCP for Claude Code. Make entire codebase the context for any coding agent.

## Install

```
/plugin marketplace add https://rony-remedio.github.io/remedio-skills/
/plugin install claude-context@remedio
```

## What this installs

An MCP server registered as `claude-context`, launched with:

```
npx -y @zilliz/claude-context-mcp@latest
```

## Built-in values

Shipped as-is in the plugin's `.mcp.json`:

- `EMBEDDING_PROVIDER` = `Ollama`
- `EMBEDDING_MODEL` = `nomic-embed-text`
- `MILVUS_ADDRESS` = `http://localhost:19530`

## Upstream

Source: <https://github.com/zilliztech/claude-context>
