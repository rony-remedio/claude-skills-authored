# claude-context

Code search MCP for Claude Code. Make entire codebase the context for any coding agent.

## Install

```
/plugin marketplace add https://rony-remedio.github.io/remedio-skills/
/plugin install claude-context@remedio
```

## Post-install: make it actually work

Upstream `@zilliz/claude-context-core` has a pre-flight probe that times out on Zilliz Serverless cold-starts (see [Known issue](#known-issue-zilliz-serverless-cold-start) below). Until [PR #291](https://github.com/zilliztech/claude-context/pull/291) merges, every fresh install hits this unless two lines in the npx'd server are patched.

**Easiest fix:** paste this prompt into Claude Code (in any repo / session) immediately after installing the plugin:

```
Patch my local @zilliz/claude-context-core npx cache to work around the
Zilliz Serverless cold-start bug (upstream: zilliztech/claude-context#289,
PR #291).

Find the file:
  ~/.npm/_npx/*/node_modules/@zilliz/claude-context-core/dist/vectordb/milvus-vectordb.js

Apply two edits:

1. Inside the MilvusClient constructor call (near the top of the file,
   around line 25, look for `ssl: milvusConfig.ssl || false,`), add a
   new trailing property:

       timeout: Number(process.env.MILVUS_TIMEOUT_MS) || 60000,

2. Inside `async checkCollectionLimit() {`, make `return true;` the first
   statement — before the existing `if (!this.client)` check. Leave the
   rest of the function body intact so the diff is minimal.

Verify both edits stuck with:
  grep -n "MILVUS_TIMEOUT_MS\|return true" <file>

Then tell me to run /mcp and reconnect claude-context.
```

After the agent confirms both edits are in place, run `/mcp` → reconnect `claude-context`. The server starts cleanly against Zilliz Serverless (60-second Milvus gRPC deadline, no more dummy-probe timeout).

### Why this patch isn't in our wrapper

The two lines live inside a published npm package (`@zilliz/claude-context-core`), not in files this repo controls. To fix it for every installer we'd need to fork + publish under our own npm scope — worthwhile if upstream drags. For now, the prompt above is the one-shot path.

### When the patches vaporize

They live at `~/.npm/_npx/<hash>/node_modules/…`. If npx picks a new cache hash (package version shift, dep tree change), the patches are gone and you re-run the prompt. This is why it's a workaround, not a fix.

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
- `milvus_timeout_ms` — gRPC deadline for Milvus operations (string, default `60000`). Only takes effect once the post-install patch above is applied OR upstream PR #291 merges.

### Heads-up on secret storage

`milvus_token` is marked `sensitive: false` on purpose. Claude Code's docs say sensitive userConfig values go to the OS keychain, but the keychain write path isn't wired up yet ([anthropics/claude-code#39455](https://github.com/anthropics/claude-code/issues/39455)). Flipping `sensitive: true` results in the value being silently dropped. For now the token lives in `~/.claude/settings.json` — file is user-only readable, but be aware.

## Built-in values

The wrapper ships the Ollama embedding profile. To switch to OpenAI / Voyage / etc., edit the plugin's `.mcp.json` directly or submit a separate wrapper profile.

- `EMBEDDING_PROVIDER = Ollama`
- `EMBEDDING_MODEL = nomic-embed-text`
- `OLLAMA_HOST = http://localhost:11434`

## Known issue: Zilliz Serverless cold-start

**Symptoms:** First call to `index_codebase` fails with `DEADLINE_EXCEEDED`; MCP log shows `MilvusVectorDatabase.resolveAddress` / `checkCollectionLimit` stack.

**Cause:** Upstream `checkCollectionLimit()` does a pre-flight `createCollection` + `dropCollection` probe using the Milvus node SDK's default gRPC deadline (15 s). Zilliz Serverless cold-starts exceed that. REST transport bypasses the probe; gRPC doesn't.

Tracking: [zilliztech/claude-context#289](https://github.com/zilliztech/claude-context/issues/289), [#290](https://github.com/zilliztech/claude-context/issues/290), [#154](https://github.com/zilliztech/claude-context/issues/154). Fix: [PR #291](https://github.com/zilliztech/claude-context/pull/291) — still open at time of writing.

**Alternatives to the post-install prompt:**

1. **Use local Milvus** instead of Zilliz Serverless. No cold-start, probe completes under 15 s:
   ```bash
   docker run -d -p 19530:19530 milvusdb/milvus:latest
   ```
   Then set `milvus_address = http://localhost:19530`, leave `milvus_token` blank.

2. **Wait for PR #291** to merge and bump the plugin pin. The `MILVUS_TIMEOUT_MS` userConfig is already wired through — once upstream reads it, a single `/plugin update` fixes it for everyone without the post-install prompt.

## Upstream

Source: <https://github.com/zilliztech/claude-context>
