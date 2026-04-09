<div align="center">

# 🧠 Open Memory Protocol (OMP)

### Your AI conversations belong to you.

[![OMP Version](https://img.shields.io/badge/OMP-v2.0-00d4ff?style=for-the-badge)](./spec/omp-v2.0.md)
[![License: CC BY 4.0](https://img.shields.io/badge/Spec-CC%20BY%204.0-lightgrey?style=for-the-badge)](./LICENSE-SPEC.md)
[![License: Apache 2.0](https://img.shields.io/badge/Code-Apache%202.0-blue?style=for-the-badge)](./LICENSE-CODE.md)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen?style=for-the-badge)](./CONTRIBUTING.md)

**A universal, vendor-neutral standard for owning, backing up, and moving your AI conversations across every platform.**

[Read the Spec](./spec/omp-v2.0.md) · [Website](https://openmemoryprotocol.com) · [Connectors](#connectors) · [Contributing](./CONTRIBUTING.md)

</div>

---

## The Problem

You have conversations scattered across ChatGPT, Claude, Gemini, Cursor, Copilot, and more. Each platform locks your data in a proprietary format — or doesn't let you export it at all. If you switch platforms, you lose everything: every preference learned, every decision documented, every piece of context accumulated.

**OMP fixes this.** Think of it as the **iCloud of AI** — your conversations are automatically backed up, searchable across every platform, and recoverable if anything goes wrong.

## How It Works

```
┌──────────┐     ┌──────────┐     ┌──────────┐
│ ChatGPT  │     │  Claude  │     │  Gemini  │
│ ┌──────┐ │     │ ┌──────┐ │     │ ┌──────┐ │
│ │ JSON │ │     │ │ JSON │ │     │ │Takeout│ │
│ └──┬───┘ │     │ └──┬───┘ │     │ └──┬───┘ │
└────┼─────┘     └────┼─────┘     └────┼─────┘
     │                │                │
     ▼                ▼                ▼
  ┌─────────────────────────────────────────┐
  │         OMP Connectors (convert)        │
  └─────────────────┬───────────────────────┘
                    │
                    ▼
            ┌───────────────┐
            │  .omp.zip     │  ← Universal format
            │  ┌──────────┐ │     Human-readable JSON
            │  │manifest  │ │     Self-contained
            │  │convos/   │ │     Lossless
            │  │memories/ │ │
            │  └──────────┘ │
            └───────┬───────┘
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
  ┌──────────┐ ┌──────────┐ ┌──────────┐
  │  Local   │ │  Cloud   │ │Self-host │
  │  Folder  │ │  Vault   │ │  Server  │
  └──────────┘ └──────────┘ └──────────┘
                Memory Hosts
```

## Quick Start

### Back Up Your ChatGPT History

```bash
# 1. Export your data from ChatGPT (Settings → Data controls → Export data)
# 2. Convert to OMP format
npx @omp/cli convert chatgpt ./conversations.json -o ./my-backup.omp.zip

# 3. Validate the archive
npx @omp/cli validate ./my-backup.omp.zip
# ✓ Valid OMP Archive: 847 conversations, 12,453 messages
```

### Self-Host a Memory Vault

```bash
# Pull and run the reference server
docker pull ghcr.io/open-memory-protocol/omp-server:latest
docker run -p 3000:3000 -v omp-data:/data ghcr.io/open-memory-protocol/omp-server:latest

# Import your backup
curl -X POST http://localhost:3000/restore \
  -F "archive=@my-backup.omp.zip"
```

### Use the SDK

```typescript
import { OMPArchive } from '@omp/sdk';

// Parse an OMP archive
const archive = await OMPArchive.fromFile('./my-backup.omp.zip');
console.log(`${archive.conversations.length} conversations loaded`);

// Search across all conversations
const results = archive.search('database schema');
results.forEach(r => console.log(`[${r.platform}] ${r.conversation.title}`));
```

## Core Concepts

| Concept | Description |
|---------|-------------|
| **Conversation** | A complete exchange between you and an AI — an ordered list of messages |
| **Message** | A single turn: your prompt, the AI's response, a tool call, or a system instruction |
| **Memory Record** | Structured intelligence derived from conversations: summaries, preferences, decisions |
| **Memory Vault** | Your complete collection of OMP data, stored wherever you choose |
| **Memory Host** | The service that stores your vault — local, cloud, or self-hosted |
| **Connector** | Translates between a platform's native format and OMP format |
| **OMP Archive** | A portable `.omp.zip` file for backup, transfer, or import |

## Five User Rights

OMP guarantees every user five non-negotiable operations:

| Operation | What It Does |
|-----------|-------------|
| 🔒 **BACKUP** | Create a complete copy of all your AI conversations in OMP format |
| 📥 **RESTORE** | Import an OMP Archive back into any Memory Host |
| 🔄 **MIGRATE** | Move conversations from one platform to another |
| 🗑️ **DELETE** | Permanently remove specific conversations or your entire vault |
| 🔍 **SEARCH** | Find anything across all platforms, by keyword or meaning |

## Connectors

Convert platform-specific exports to the universal OMP format:

| Platform | Status | Maintainer |
|----------|--------|------------|
| ChatGPT  | ✅ Available | OMP Working Group |
| Claude   | ✅ Available | OMP Working Group |
| Gemini   | 🚧 In Progress | Community |
| Cursor   | 🚧 In Progress | Community |
| Perplexity | 📋 Planned | Community |
| Grok     | 📋 Planned | Community |
| Copilot  | 📋 Planned | Community |

[Build a connector →](./docs/building-connectors.md)

## Conformance Levels

| Level | Name | Requirements |
|-------|------|-------------|
| **Level 1** | Portable | Produce & consume OMP Archives, BACKUP/RESTORE/DELETE, TLS |
| **Level 2** | Connected | Full API, keyword search, connectors, auto-backup, pagination |
| **Level 3** | Enterprise | Multi-tenant isolation, RBAC, audit logs, compliance profiles |

## Project Structure

```
open-memory-protocol/
├── spec/              # The OMP v2.0 specification + JSON schemas
├── connectors/        # Platform-specific converters
├── sdk/               # TypeScript and Python client libraries
├── reference-server/  # Self-hostable Memory Host (Docker template)
├── validator/         # CLI tool to validate OMP archives
├── website/           # Landing page (Astro)
└── docs/              # Guides and API reference
```

## Hosted Solution

Don't want to self-host? **[Memory Vault](https://openmemoryprotocol.org/vault)** is a cloud-hosted Memory Host that stores your conversations, visualizes them as an interactive knowledge graph, and lets you ask AI-powered questions about your history.

## Contributing

We welcome contributions from everyone! See our [Contributing Guide](./CONTRIBUTING.md) to get started.

- 🐛 [Report a Bug](./.github/ISSUE_TEMPLATE/bug_report.md)
- 💡 [Request a Feature](./.github/ISSUE_TEMPLATE/feature_request.md)
- 📝 [Submit an OEP (OMP Enhancement Proposal)](./.github/ISSUE_TEMPLATE/oep-proposal.md)

## Governance

OMP is maintained by an open Working Group with no single-vendor veto. Changes follow the OEP (OMP Enhancement Proposal) process with a 30-day public comment period. See [Governance](./spec/omp-v2.0.md#15-governance) for details.

## License

- **Specification**: [Creative Commons Attribution 4.0 International (CC BY 4.0)](./LICENSE-SPEC.md)
- **Code** (SDKs, connectors, reference server): [Apache License 2.0](./LICENSE-CODE.md)

---

<div align="center">

**Your AI conversations belong to you.**

[⭐ Star this repo](https://github.com/open-memory-protocol/open-memory-protocol) · [Read the Spec](./spec/omp-v2.0.md) · [Join the Discussion](https://github.com/open-memory-protocol/open-memory-protocol/discussions)

</div>

