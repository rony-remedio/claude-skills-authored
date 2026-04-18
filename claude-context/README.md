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

## Configure at install time

Claude Code will prompt you for these values when you run
`/plugin install claude-context@remedio`. Sensitive values are stored in the
OS keychain; non-sensitive values land in `~/.claude/settings.json`
under `pluginConfigs`.

- `MILVUS_TOKEN` *(sensitive)*

### Updating a value later

Run `/plugin config claude-context@remedio` to edit. Or edit
`~/.claude/settings.json` directly under `pluginConfigs.claude-context@remedio.options`
(sensitive values stay in the keychain — use the command for those).

## Built-in values

Shipped as-is in the plugin's `.mcp.json`:

- `EMBEDDING_PROVIDER` = `Ollama`
- `EMBEDDING_MODEL` = `nomic-embed-text`
- `OLLAMA_HOST` = `http://localhost:11434`

## Upstream

Source: <https://github.com/zilliztech/claude-context>
