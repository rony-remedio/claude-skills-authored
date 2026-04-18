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

Exposes four tools: `index_codebase`, `search_code`, `clear_index`, `get_indexing_status`.

## Configure at install time

Claude Code prompts for these values when you run `/plugin install claude-context@remedio`. You can edit them later with `/plugin config claude-context@remedio` or directly in `~/.claude/settings.json` under `pluginConfigs["claude-context@remedio"].options`.

- `milvus_address` — Milvus / Zilliz Cloud endpoint (e.g. `http://localhost:19530` or `https://<cluster>.api.<region>.zillizcloud.com`).
- `milvus_token` — Milvus / Zilliz Cloud access token.
- `milvus_timeout_ms` — gRPC deadline for Milvus operations. Default `60000`. Safe to leave unless you're on Zilliz Serverless (see Known issue below).

### Heads-up on secret storage

`milvus_token` is marked `sensitive: false` on purpose. Claude Code's docs say sensitive userConfig values go to the OS keychain, but the keychain write path isn't wired up yet ([anthropics/claude-code#39455](https://github.com/anthropics/claude-code/issues/39455)). Flipping `sensitive: true` results in the value being silently dropped. For now the token lives in `~/.claude/settings.json` — file is user-only readable, but be aware.

## Built-in values

The wrapper ships the Ollama embedding profile (MILVUS-only config baked). To switch embeddings to OpenAI / Voyage / etc., edit the plugin's `.mcp.json` directly or submit a separate wrapper profile.

- `EMBEDDING_PROVIDER = Ollama`
- `EMBEDDING_MODEL = nomic-embed-text`
- `OLLAMA_HOST = http://localhost:11434`

## Known issue: Zilliz Serverless cold-start timeout

**Symptoms:** First call to `index_codebase` fails with `DEADLINE_EXCEEDED` on Zilliz Serverless; MCP log shows `MilvusVectorDatabase.resolveAddress` / `checkCollectionLimit` stack.

**Cause:** Upstream `@zilliz/claude-context-core`'s `checkCollectionLimit()` does a pre-flight `createCollection` + `dropCollection` probe using the Milvus node SDK's default gRPC deadline (15 s). Zilliz Serverless cold-starts exceed that. REST transport bypasses the probe; gRPC doesn't.

Tracking: [zilliztech/claude-context#289](https://github.com/zilliztech/claude-context/issues/289), [#290](https://github.com/zilliztech/claude-context/issues/290), [#154](https://github.com/zilliztech/claude-context/issues/154). Fix: [PR #291](https://github.com/zilliztech/claude-context/pull/291) — still open at time of writing.

**Workarounds until the fix ships:**

1. **Use local Milvus** instead of Zilliz Serverless. No cold-start, probe completes under 15 s:
   ```bash
   docker run -d -p 19530:19530 milvusdb/milvus:latest
   ```
   Then set `milvus_address = http://localhost:19530`, leave `milvus_token` blank.

2. **Patch the local npx cache** (dev-box only; wiped on next cache refresh):
   ```bash
   # find the cache dir
   ls ~/.npm/_npx/*/node_modules/@zilliz/claude-context-core/dist/vectordb/
   # edit milvus-vectordb.js — bump the timeout and short-circuit checkCollectionLimit
   ```

3. **Wait for PR #291** and bump this plugin once the fix is published. The `MILVUS_TIMEOUT_MS` userConfig is already wired through — no wrapper change needed when that lands.

## Upstream

Source: <https://github.com/zilliztech/claude-context>
