# openclaw-mem0

Long-term memory plugin for [OpenClaw](https://openclaw.ai) agents, powered by [Mem0](https://mem0.ai).

Supports both the Mem0 cloud platform and self-hosted open-source deployments using any OpenAI-compatible provider (OpenRouter, DashScope, LocalAI, etc).

## Features

- **5 memory tools** — `memory_search`, `memory_store`, `memory_list`, `memory_get`, `memory_forget`
- **Session + long-term scopes** — short-term (session) and long-term (user) memory
- **Auto-recall** — injects relevant memories before each agent turn
- **Auto-capture** — stores key facts after each agent turn
- **Dual mode** — Mem0 platform (cloud) or open-source (self-hosted)
- **Provider-agnostic** — works with OpenRouter, DashScope, LocalAI, or any OpenAI-compatible API
- **Lazy provider loading** — only loads the SDKs you actually use, no bloated installs

## Install

```bash
openclaw plugins install github:tensakulabs/openclaw-mem0
```

## Configuration

Add to your `openclaw.json`:

### Platform mode (Mem0 cloud)

```json
{
  "plugins": {
    "entries": {
      "openclaw-mem0": {
        "config": {
          "mode": "platform",
          "apiKey": "${MEM0_API_KEY}",
          "userId": "default",
          "autoCapture": true,
          "autoRecall": true
        }
      }
    }
  }
}
```

### Open-source mode (self-hosted)

```json
{
  "plugins": {
    "entries": {
      "openclaw-mem0": {
        "config": {
          "mode": "open-source",
          "userId": "default",
          "autoCapture": true,
          "autoRecall": true,
          "oss": {
            "embedder": {
              "provider": "openai",
              "config": {
                "apiKey": "${OPENROUTER_API_KEY}",
                "baseURL": "https://openrouter.ai/api/v1",
                "model": "openai/text-embedding-3-small"
              }
            },
            "vectorStore": {
              "provider": "qdrant",
              "config": {
                "host": "localhost",
                "port": 6333,
                "collectionName": "memories"
              }
            },
            "llm": {
              "provider": "openai",
              "config": {
                "apiKey": "${OPENROUTER_API_KEY}",
                "baseURL": "https://openrouter.ai/api/v1",
                "model": "openai/gpt-4o-mini"
              }
            }
          }
        }
      }
    }
  }
}
```

## What's different from the official plugin

The official `@mem0/openclaw-mem0` has several issues that break self-hosted deployments:

1. **Auto-recall silently discards memories** — wrong property name in the hook return ([mem0ai/mem0#4037](https://github.com/mem0ai/mem0/issues/4037))
2. **Embeddings ignore your `baseURL`** — always hits `api.openai.com` instead of your configured provider ([mem0ai/mem0#4040](https://github.com/mem0ai/mem0/issues/4040))
3. **Missing provider SDKs crash at startup** — `mem0ai` eagerly imports 17 provider SDKs regardless of which you use
4. **Qdrant version warnings** — noisy compatibility warnings with newer Qdrant servers

This plugin fixes all of these. It vendors a patched build of the mem0 OSS module with lazy dynamic imports — only the SDKs you actually configure get loaded.

## License

Apache-2.0
