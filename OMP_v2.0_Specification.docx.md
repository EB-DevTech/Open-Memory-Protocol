

**OPEN MEMORY PROTOCOL**

**(OMP)**

Technical Standard Specification

Version 2.0 — Draft

April 2026

| CORE PRINCIPLE Your AI conversations belong to you. Not to the platform. Not to the model provider. To you. OMP exists so that every person can back up, recover, migrate, and control every conversation they have with any AI system — across every platform, forever. |
| :---- |

*The iCloud of AI — Backup, Sync, and Recover Your Conversations Everywhere*

## **Status of This Document**

This is the Open Memory Protocol (OMP) Version 2.0 Draft specification. It defines a vendor-neutral, open standard for user-owned AI conversation backup, recovery, and cross-platform portability — with enterprise governance extensions for regulated industries.

This specification is published under a royalty-free license and is open for public review and contribution.

# **Table of Contents**

**PART I**

**USER PORTABILITY STANDARD**

*The iCloud of AI — Backup, Sync, and Recover Across Every Platform*

# **1\. Abstract**

The Open Memory Protocol (OMP) defines a universal, vendor-neutral standard that gives every person full ownership, backup, recovery, and portability of their AI conversations across all platforms. Think of it as the iCloud of AI — your conversations are automatically backed up, searchable across every platform, and recoverable if anything goes wrong. OMP specifies a common exchange format, a set of platform obligations, and user-facing operations that ensure no person’s AI history is ever locked inside a single product.

OMP is structured in two parts. Part I (Sections 1–9) defines the core user portability standard — the minimum required to give every person control over their conversations. Part II (Sections 10–16) extends the standard with enterprise governance, regulatory compliance, and operational features for organizations deploying AI in regulated environments. Part I can stand alone. Part II builds on Part I.

# **2\. The Problem**

## **2.1 What Users Face Today**

A person using AI today has conversations scattered across multiple platforms. Each platform stores those conversations in a different proprietary format — or doesn’t let the user export them at all:

| Platform | Export Format | Portability |
| :---- | :---- | :---- |
| ChatGPT | Nested JSON DAG with internal node IDs, message trees, and status flags. A 2-year history can exceed 500MB. | Cannot be imported into any other platform. |
| Claude | Flat JSON array with tool call artifacts and system metadata. | No other platform accepts this format. |
| Gemini | Google Takeout. Chat history often not included; only Gems and Workspace data export reliably. | Incompatible with all other platforms. |
| Cursor | Local SQLite database. No built-in export. | Completely locked to the local machine. |
| Perplexity | Q\&A pairs with citation metadata. | No standard format. No import path elsewhere. |
| Grok | BSON timestamps and proprietary structure. | Incompatible with everything. |
| Copilot | No conversation export available. | Total lock-in. |

The result: if you switch from ChatGPT to Claude, or from Claude to Gemini, you lose everything. Every preference learned, every decision documented, every piece of context accumulated — gone.

| THE ANALOGY iCloud backs up your photos, contacts, messages, and documents — automatically, across every Apple device you own. If you lose your phone, everything comes back. If you switch devices, everything follows you. Now imagine that for your AI conversations. Every chat with ChatGPT, Claude, Gemini, Copilot — automatically backed up, searchable, portable, and under your control. That’s what OMP is. The iCloud of AI. |
| :---- |

## **2.2 Why This Matters**

* **Personal continuity:** You’ve spent months teaching an AI your preferences, your projects, your communication style. That accumulated context has real value. Losing it is like losing a notebook.

* **Backup and recovery:** Platforms go down. Companies shut down. Accounts get locked. iCloud protects your photos from all of this — OMP does the same for your AI conversations. If your conversations aren’t backed up in a format you control, they can vanish overnight.

* **Freedom of choice:** Vendor lock-in through data captivity is anti-competitive. A user should be able to try a new AI platform without sacrificing their history on the old one.

* **Digital rights:** Your conversations are your data. You have a right to access, export, back up, and delete them. Multiple regulations already mandate this — but no standard exists to make it practical.

# **3\. Terminology and Conventions**

## **3.1 Key Words**

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” are interpreted as described in RFC 2119\.

## **3.2 Definitions**

| Term | Definition |
| :---- | :---- |
| User | Any individual who has conversations with AI systems and owns the resulting data. |
| Platform | Any AI system, application, or service that a User interacts with (e.g., ChatGPT, Claude, Gemini, Cursor, a custom AI agent). |
| Conversation | A complete, coherent exchange between a User and a Platform, comprising one or more Messages. |
| Message | A single turn within a Conversation: one prompt, one response, one system instruction, or one tool invocation. |
| Memory Vault | A User’s complete collection of OMP-formatted conversation data, stored in a location the User controls. |
| OMP Archive | A portable file (.omp.zip) containing one or more Conversations in OMP format, suitable for backup, transfer, or import. |
| Memory Host | A service that stores and serves a User’s Memory Vault (can be local, cloud, or self-hosted). |
| Connector | A software component that translates between a Platform’s native format and OMP format. |
| Memory Record | Structured intelligence derived from Conversations (summaries, preferences, extracted entities, decisions). Used for cross-conversation context. |

# **4\. The OMP Universal Exchange Format**

This is the heart of the standard. The OMP Exchange Format defines how AI conversations are structured so that any platform can export them, any user can back them up, and any compatible system can import them. It is designed to be:

* **Human-readable:** JSON-based, inspectable with any text editor.

* **Flat and simple:** No nested DAGs, no internal node IDs, no platform-specific artifacts. A conversation is a flat, ordered list of messages.

* **Self-contained:** Every conversation carries all the metadata needed to understand it, with no external references required.

* **Lossless:** Platform-specific metadata is preserved in an extensions field rather than discarded.

## **4.1 Message Object**

A Message is the atomic unit of the exchange format. Every turn in a conversation — user prompt, AI response, system instruction, tool call — is one Message.

| Field | Type | Required | Description |
| :---- | :---- | :---- | :---- |
| id | string (UUID v4) | MUST | Unique identifier for this message. |
| role | enum: "user" | "assistant" | "system" | "tool" | MUST | Who produced this message. |
| content | string | ContentBlock\[\] | MUST | The message text, or an array of multimodal content blocks (see 4.1.1). |
| timestamp | string (ISO 8601 UTC) | MUST | When this message was created. |
| model | string | SHOULD | The AI model that generated this message (e.g., "gpt-4o", "claude-sonnet-4-20250514"). Null for user messages. |
| platform | string | SHOULD | The platform where this message was created (e.g., "chatgpt", "claude", "gemini"). |
| attachments | Attachment\[\] | MAY | Files, images, or documents attached to this message. |
| tool\_calls | ToolCall\[\] | MAY | Tool/function calls made during this message. |
| token\_usage | { prompt: int, completion: int } | MAY | Token counts for this message. |
| annotations | Annotation\[\] | MAY | User notes, bookmarks, or tags on this message. |
| extensions | object | MAY | Platform-specific metadata preserved during export. Prefixed with the platform name (e.g., chatgpt\_node\_id, claude\_artifact\_id). |

### **4.1.1 Content Block Types**

When the content field is an array (for multimodal messages), each block MUST have a type field:

| Type | Required Fields | Description |
| :---- | :---- | :---- |
| text | text (string) | Plain text content. |
| image | media\_type, data (base64 or URL) | An image included in or generated by the message. |
| document | media\_type, data (base64 or URL), filename | A PDF, spreadsheet, or other document. |
| code | language, text | A code block with language identifier. |
| tool\_use | tool\_name, tool\_input | A tool/function invocation. |
| tool\_result | tool\_name, output | The result returned by a tool. |

### **4.1.2 Attachment Object**

{

  "filename": "report.pdf",

  "media\_type": "application/pdf",

  "size\_bytes": 245000,

  "data": "base64-encoded string | URL",

  "source": "user\_upload | ai\_generated"

}

## **4.2 Conversation Object**

A Conversation is an ordered collection of Messages representing a complete exchange.

| Field | Type | Required | Description |
| :---- | :---- | :---- | :---- |
| id | string (UUID v4) | MUST | Unique identifier for this conversation. |
| title | string | SHOULD | Human-readable title (auto-generated or user-defined). |
| created\_at | string (ISO 8601 UTC) | MUST | When the conversation started. |
| updated\_at | string (ISO 8601 UTC) | MUST | When the last message was added. |
| platform | string | MUST | The originating platform identifier. |
| model | string | SHOULD | The primary AI model used (if consistent across the conversation). |
| messages | Message\[\] | MUST | Ordered array of Messages, from first to last. |
| message\_count | integer | MUST | Total number of messages. |
| tags | string\[\] | MAY | User-defined tags for organization. |
| context | ContextRef\[\] | MAY | Links to business objects, projects, or topics. |
| summary | string | MAY | AI-generated or user-written summary of the conversation. |
| language | string (BCP 47\) | MAY | Primary language of the conversation (e.g., "en", "es", "zh"). |
| extensions | object | MAY | Platform-specific metadata preserved during export. |

## **4.3 Memory Record Object**

A Memory Record captures structured knowledge derived from conversations — the things an AI has learned about you that persist across sessions. These are what make AI feel “personal” and are the most valuable data to preserve.

| Field | Type | Required | Description |
| :---- | :---- | :---- | :---- |
| id | string (UUID v4) | MUST | Unique identifier. |
| record\_type | enum: "preference" | "fact" | "decision" | "summary" | "entity" | "instruction" | "custom" | MUST | What kind of memory this is. |
| content | string | MUST | The memory in natural language (e.g., “User prefers dark mode”, “User is vegetarian”). |
| source\_conversations | string\[\] (UUIDs) | SHOULD | Which conversations produced this memory. |
| created\_at | string (ISO 8601 UTC) | MUST | When this memory was created. |
| updated\_at | string (ISO 8601 UTC) | MUST | When this memory was last modified. |
| confidence | number (0.0–1.0) | MAY | How confident the system is in this memory. |
| expires\_at | string (ISO 8601 UTC) | MAY | When this memory should be auto-reviewed or deleted. |
| supersedes | string (UUID) | MAY | ID of a previous Memory Record this one replaces. |
| platform | string | SHOULD | Which platform created this memory. |
| tags | string\[\] | MAY | User-defined tags. |
| active | boolean | MUST | Whether this memory is currently in use (allows soft-disable without deletion). |

## **4.4 The OMP Archive Format (.omp.zip)**

An OMP Archive is the portable container for backup, transfer, and import. It is a standard ZIP file with the extension .omp.zip containing:

my-ai-backup.omp.zip

├── manifest.json

├── conversations/

│   ├── {conversation-id-1}.json

│   ├── {conversation-id-2}.json

│   └── ...

├── memories/

│   ├── {memory-id-1}.json

│   └── ...

└── attachments/

    ├── {attachment-hash}.png

    └── ...

### **4.4.1 Manifest Schema**

{

  "omp\_version": "2.0",

  "export\_timestamp": "2026-04-07T12:00:00Z",

  "user\_id": "string (optional, for multi-user systems)",

  "source\_platform": "chatgpt | claude | gemini | multi-platform | ...",

  "counts": {

    "conversations": 142,

    "messages": 8753,

    "memories": 67,

    "attachments": 23

  },

  "date\_range": {

    "earliest": "2023-03-14T08:22:00Z",

    "latest": "2026-04-07T11:58:00Z"

  },

  "platforms\_included": \["chatgpt", "claude", "gemini"\],

  "checksum": "sha256:abc123..."

}

| DESIGN DECISION: WHY A ZIP FILE? Every user already knows how to handle ZIP files. They can store them on a USB drive, upload them to Google Drive, email them to themselves. No special software required. The .omp.zip extension signals OMP compatibility, but it’s still a standard ZIP that any tool can open. This is intentional. The format should survive even if every OMP tool disappears. |
| :---- |

# **5\. User Operations**

OMP defines five core operations that every user MUST be able to perform. These are the non-negotiable rights the standard exists to guarantee.

## **5.1 BACKUP**

| What it does Creates a complete, self-contained copy of all your AI conversations and memories in OMP format. |
| :---- |

A compliant Memory Host MUST provide the ability to create a full backup of the user’s Memory Vault as an OMP Archive (.omp.zip). Backups MUST:

* Include all Conversations, Messages, Memory Records, and Attachments.

* Conform to the OMP Exchange Format (Section 4).

* Be completeable within 72 hours of request (for archives under 10GB).

* Include a valid manifest.json with accurate counts and a SHA-256 checksum.

Memory Hosts SHOULD support scheduled automatic backups (daily, weekly, monthly) to a user-specified destination (local folder, cloud storage, email).

## **5.2 RESTORE**

| What it does Imports an OMP Archive back into a Memory Host, reconstructing your complete conversation history. |
| :---- |

A compliant Memory Host MUST accept a valid OMP Archive and restore its contents. The restore operation MUST:

* Parse and validate the manifest.json and verify the checksum.

* Import all Conversations, Messages, Memory Records, and Attachments.

* Detect and handle duplicates by comparing Message IDs (skip if already present; do not overwrite).

* Report a restore summary: records imported, duplicates skipped, errors encountered.

Memory Hosts SHOULD support selective restore (e.g., “import only conversations from Claude”, “import only conversations tagged ‘work’”).

## **5.3 MIGRATE**

| What it does Moves your conversations from one platform to another, preserving as much context as possible. |
| :---- |

Migration is a BACKUP from the source platform followed by a RESTORE to the destination. The standard defines this as a two-step process to keep each step simple and auditable. However, compliant Memory Hosts MAY implement direct platform-to-platform migration as a convenience feature, provided the underlying data passes through OMP format.

When migrating between platforms with different capabilities (e.g., from a platform that supports tool calls to one that doesn’t), the destination MUST:

* Preserve all data it can natively represent.

* Store data it cannot natively represent in the extensions field, so it is not lost.

* Inform the user of any data that could not be fully represented (e.g., “23 tool call records were preserved in metadata but are not displayed in this platform”).

## **5.4 DELETE**

| What it does Permanently removes specific conversations, memories, or your entire Memory Vault. |
| :---- |

A compliant Memory Host MUST support:

* Deletion of individual Messages, Conversations, or Memory Records by ID.

* Deletion of all data matching a filter (e.g., “all conversations from platform X”, “all data before date Y”).

* Full vault deletion: permanent removal of all user data.

Deletion MUST be permanent and irrecoverable within 30 days, including from backups. Deletion of a Conversation MUST cascade to all child Messages. The Memory Host MUST confirm the scope of deletion before executing (e.g., “This will permanently delete 142 conversations containing 8,753 messages. Proceed?”).

## **5.5 SEARCH**

| What it does Find anything you’ve ever said to or received from any AI, across all platforms. |
| :---- |

A compliant Memory Host MUST support keyword search across all Conversations and Memory Records. Compliant Memory Hosts SHOULD additionally support:

* Semantic search (find by meaning, not just exact words).

* Filtering by platform, date range, model, tags, and conversation.

* Full-text search within attachments (PDFs, documents).

Search results MUST include the source platform, conversation context, and timestamp for each result.

# **6\. Platform Obligations**

For OMP to work, AI platforms must participate. This section defines what platforms MUST, SHOULD, and MAY do to be OMP-compliant.

## **6.1 Export Obligation (REQUIRED)**

Every OMP-compliant platform MUST provide a mechanism for users to export their complete conversation history in OMP format. This means:

* A user-accessible export function (not hidden behind API keys or developer tools).

* Export produces a valid OMP Archive (.omp.zip) conforming to Section 4\.

* Export includes ALL user conversations, not a subset.

* Export is available within 72 hours of request.

* Export is free of charge (a platform MUST NOT charge for data export).

## **6.2 Import Capability (RECOMMENDED)**

Platforms SHOULD accept OMP Archives for import, allowing users to bring their history from other platforms. When importing:

* The platform MUST validate the archive’s manifest and checksum.

* Messages SHOULD be displayable in the platform’s native conversation interface.

* Memory Records SHOULD be integrated into the platform’s memory/personalization system if one exists.

* Data the platform cannot natively display MUST be preserved in extensions, not discarded.

## **6.3 Connector API (RECOMMENDED)**

Platforms SHOULD expose a Connector API that allows authorized third-party Memory Hosts and backup tools to:

* Stream new conversations to the user’s Memory Vault in real-time (push model).

* Periodically sync the user’s conversation history (pull model).

The Connector API MUST use OAuth 2.0 or equivalent for authentication and MUST require explicit user consent before any data is shared with a third party.

## **6.4 Connector Registry**

The OMP Working Group maintains a public Connector Registry listing platform-specific connectors that translate between native formats and OMP format. This enables portability even for platforms that do not natively support OMP:

| Platform | Connector Type | Maintained By |
| :---- | :---- | :---- |
| ChatGPT | Export file converter (conversations.json → .omp.zip) | OMP Working Group |
| Claude | Export file converter (claude-export.json → .omp.zip) | OMP Working Group |
| Gemini | Google Takeout converter | Community |
| Cursor | SQLite database extractor | Community |
| Perplexity | Export file converter | Community |
| Grok | Export file converter | Community |
| Custom / Local LLMs | Ollama, LM Studio, Open WebUI adapters | Community |

Connector specifications are published in the OMP GitHub repository. Any developer can contribute a connector for any platform.

# **7\. Memory Host Requirements**

A Memory Host is where your OMP data lives. It can be a cloud service, a self-hosted server, a desktop app, or even a folder on a USB drive. The standard supports all deployment models.

## **7.1 Deployment Models**

| Model | Description | Best For |
| :---- | :---- | :---- |
| Local-first | Data stored entirely on the user’s device. No cloud. Example: a desktop app with SQLite \+ local folder. | Maximum privacy. Users who don’t trust cloud providers. |
| Self-hosted | User runs their own Memory Host server (Docker, VPS, NAS). Full control, accessible from multiple devices. | Technical users and small businesses who want portability across devices. |
| Cloud-hosted | A service provider runs the Memory Host. User accesses via web/app. Provider handles infrastructure. | Convenience. Non-technical users who want automatic backup and sync. |
| Hybrid | Local storage with optional cloud sync. User chooses what syncs and what stays local. | Balance of privacy and convenience. |

## **7.2 API Surface**

Memory Hosts that expose an API (all models except pure local-folder) MUST implement the following RESTful endpoints:

### **7.2.1 Conversations**

| Method | Endpoint | Required | Description |
| :---- | :---- | :---- | :---- |
| POST | /conversations | MUST | Store a new Conversation. |
| GET | /conversations/{id} | MUST | Retrieve a Conversation with all Messages. |
| GET | /conversations | MUST | List all Conversations (supports pagination, filtering by platform, date, tags). |
| DELETE | /conversations/{id} | MUST | Delete a Conversation and all child Messages. |
| PATCH | /conversations/{id} | SHOULD | Update title, tags, or summary. |

### **7.2.2 Memory Records**

| Method | Endpoint | Required | Description |
| :---- | :---- | :---- | :---- |
| POST | /memories | MUST | Create a new Memory Record. |
| GET | /memories | MUST | List all Memory Records (supports filtering by type, platform, tags). |
| GET | /memories/{id} | MUST | Retrieve a single Memory Record. |
| PATCH | /memories/{id} | SHOULD | Update content, tags, active status, or expiry. |
| DELETE | /memories/{id} | MUST | Delete a Memory Record. |

### **7.2.3 Backup and Restore**

| Method | Endpoint | Required | Description |
| :---- | :---- | :---- | :---- |
| POST | /backup | MUST | Initiate a full backup. Returns a job ID. |
| GET | /backup/{job\_id} | MUST | Check backup status. Returns download URL when complete. |
| POST | /restore | MUST | Upload an OMP Archive for import. Returns a job ID. |
| GET | /restore/{job\_id} | MUST | Check restore status. Returns summary when complete. |

### **7.2.4 Search**

| Method | Endpoint | Required | Description |
| :---- | :---- | :---- | :---- |
| GET | /search?q={query} | MUST | Keyword search across all conversations and memories. |
| GET | /search?q={query}\&semantic=true | SHOULD | Semantic search (by meaning). |

## **7.3 Pagination**

All list endpoints MUST support cursor-based pagination:

* **cursor** — Opaque token from the previous response.

* **limit** — Results per page (default: 50, max: 200).

Responses MUST include next\_cursor (null if no more results) and has\_more (boolean).

## **7.4 Error Responses**

All errors MUST use standard HTTP status codes and return:

{ "error": { "code": "string", "message": "string", "details": {} } }

| Status | Code | Meaning |
| :---- | :---- | :---- |
| 400 | invalid\_request | Malformed request or missing required fields. |
| 401 | unauthorized | Missing or invalid credentials. |
| 403 | forbidden | Authenticated but not authorized. |
| 404 | not\_found | Resource does not exist. |
| 409 | conflict | Duplicate ID on import. |
| 413 | payload\_too\_large | Archive exceeds size limit. |
| 429 | rate\_limited | Too many requests. Retry-After header provided. |
| 500 | internal\_error | Server error. Retry with backoff. |

# **8\. Security**

## **8.1 Transport**

All API communications MUST use TLS 1.2 or later. TLS 1.3 is RECOMMENDED. Plaintext HTTP MUST NOT be used.

## **8.2 Encryption at Rest**

Memory Hosts MUST encrypt all stored data using AES-256 or equivalent. OMP Archives SHOULD support optional password-based encryption (AES-256-CBC with PBKDF2 key derivation) for local backups.

## **8.3 Authentication**

Memory Hosts exposing an API MUST support at least one of:

* OAuth 2.0 Bearer tokens (RECOMMENDED for cloud and third-party access).

* API keys (acceptable for self-hosted, single-user deployments).

* Mutual TLS (RECOMMENDED for server-to-server communication).

## **8.4 Archive Integrity**

OMP Archives MUST include a SHA-256 checksum in the manifest. On restore, Memory Hosts MUST verify the checksum before importing. If verification fails, the restore MUST be rejected with a clear error message.

# **9\. Conformance Levels (Part I)**

OMP defines two conformance levels for the core user portability standard:

## **9.1 Level 1: Portable**

The minimum bar. A Memory Host or Platform at this level guarantees basic backup and portability.

Requirements:

* Produce and consume valid OMP Archives (.omp.zip) conforming to Section 4\.

* Support the BACKUP, RESTORE, and DELETE operations (Section 5).

* Validate checksums on import.

* Transport encryption (TLS 1.2+).

## **9.2 Level 2: Connected**

Full portability with real-time sync and search. Everything in Level 1, plus:

* Implement the full API surface (Section 7.2).

* Support keyword search across all data.

* Support at least one Connector for real-time or periodic sync from a major platform.

* Support cursor-based pagination on all list endpoints.

* Support scheduled automatic backups.

## **9.3 Conformance Declaration**

Implementations claiming OMP conformance MUST publish a declaration at:

GET /.well-known/omp

{

  "omp\_version": "2.0",

  "conformance\_level": "portable | connected | enterprise",

  "implementation": { "name": "string", "version": "string" },

  "capabilities": \["backup", "restore", "search", "semantic\_search",

                    "auto\_backup", "realtime\_sync", "connectors", ...\],

  "connectors": \["chatgpt", "claude", "gemini", ...\],

  "compliance\_profiles": \[\]

}

**PART II**

**ENTERPRISE GOVERNANCE EXTENSIONS**

*Audit, Compliance, and Organizational Controls Built on the User Portability Foundation*

Part II extends the core user portability standard with features required by organizations operating in regulated industries. Every feature in Part II builds on Part I — the user’s rights to backup, restore, migrate, and delete are never diminished by enterprise features.

# **10\. Enterprise Authentication and Authorization**

## **10.1 Multi-Tenant Data Isolation**

In organizational deployments, Memory Hosts MUST enforce strict data isolation between tenants. A query by User A MUST NOT return data belonging to User B under any circumstances, including error conditions.

## **10.2 Role-Based Access Control**

| Role | Scope | Use Case |
| :---- | :---- | :---- |
| User | Full control over own Memory Vault. | Individual employees accessing their own AI history. |
| Delegate | Scoped read/write access granted by the User via explicit consent. | AI agents, apps, or team members acting on behalf of the user. |
| Admin | Tenant-wide read access, compliance operations, user management. | IT administrators and compliance officers. |

## **10.3 OAuth 2.0 Scopes**

| Scope | Permission |
| :---- | :---- |
| omp:read | Read Conversations and Memory Records. |
| omp:write | Create new Conversations and Memory Records. |
| omp:delete | Delete records within the user’s Vault. |
| omp:backup | Trigger and download backups. |
| omp:admin | Tenant-level administration. |

# **11\. Audit and Accountability**

## **11.1 Audit Logging**

Enterprise Memory Hosts MUST maintain an immutable audit log of all operations. Each entry MUST include:

* User identity (who performed the action).

* Action type (read, write, delete, backup, restore, export, access grant/revoke).

* Target resource (conversation ID, memory ID, or “full vault”).

* Timestamp (ISO 8601 UTC).

* Success/failure status.

* Source IP or client identifier.

Audit logs MUST be retained for the longer of: (a) 90 days, or (b) the minimum required by the applicable compliance profile (e.g., 6 years for HIPAA).

## **11.2 AI Provenance Tracking**

Every Message in OMP includes model and platform fields. Enterprise Memory Hosts MUST ensure these fields are populated and immutable after storage. This creates an auditable chain showing which AI system produced which output — critical for AI governance, bias auditing, and regulatory compliance.

# **12\. Conflict Resolution**

## **12.1 Conversations**

Conversations are append-only. Two clients adding messages to the same conversation MUST NOT conflict. Both are accepted, ordered by timestamp. Identical timestamps use order-of-receipt as tiebreaker.

## **12.2 Memory Records**

Memory Records support a supersedes field. When conflicts occur:

1. First write wins.

2. Second writer receives HTTP 409 with the winning record’s ID.

3. The rejected client MAY retry after incorporating the winner.

## **12.3 ETags**

Enterprise Memory Hosts SHOULD support ETags (RFC 7232\) on mutable resources for optimistic concurrency control.

# **13\. Regulatory Compliance Framework**

OMP’s user portability features (Part I) inherently satisfy many regulatory requirements — the right to export is the right to data portability; the right to delete is the right to erasure. This section maps OMP to specific regulations and defines additional controls for regulated sectors.

## **13.1 General Data Protection**

| Regulation | Jurisdiction | OMP Alignment |
| :---- | :---- | :---- |
| GDPR | European Union | BACKUP \= Art. 20 portability. DELETE \= Art. 17 erasure. OAuth scopes \= Art. 6–7 consent. Audit logs \= Art. 30 records of processing. |
| CCPA / CPRA | California | BACKUP/SEARCH \= right to know. DELETE \= right to delete. MIGRATE \= right to portability. |
| TDPSA | Texas | Full rights mapping: access (SEARCH), delete (DELETE), portability (BACKUP/MIGRATE), consent (OAuth scopes), opt-out of advertising (OMP prohibits ad use of memory data). |
| LGPD | Brazil | Portability, deletion, and consent controls map directly to OMP operations. |
| POPIA | South Africa | Purpose limitation via scoped Delegate access. Deletion and portability supported. |

## **13.2 AI Governance**

### **13.2.1 Texas Responsible AI Governance Act (TRAIGA)**

OMP supports TRAIGA compliance through:

* **Transparency:** model and platform fields on every Message provide full traceability of AI outputs.

* **Risk assessment:** x\_risk\_level metadata enables risk classification of AI interactions.

* **Human oversight:** x\_human\_review metadata records when a human reviewed or overrode an AI output.

* **Impact assessment:** Memory Hosts can query by model to generate AI impact reports.

### **13.2.2 FTC Act — Section 5**

OMP addresses FTC concerns through:

* **Anti-lock-in:** BACKUP and MIGRATE prevent unfair data captivity.

* **Transparency:** model fields prevent deceptive misrepresentation of AI sources.

* **Data security:** TLS, AES-256, and audit logging meet FTC reasonable security expectations.

## **13.3 Sector-Specific Compliance Profiles**

### **13.3.1 Healthcare — HIPAA**

When OMP data contains Protected Health Information (PHI):

* **Classification:** x\_contains\_phi boolean metadata MUST be set on affected records.

* **Access control:** RBAC with minimum necessary access. Delegates MUST receive narrowest sufficient scope.

* **Audit retention:** Six (6) years minimum for PHI-related audit logs.

* **Encryption:** AES-256 at rest. Separate key management for PHI data RECOMMENDED.

* **BAAs:** Memory Hosts handling PHI MUST execute Business Associate Agreements.

* **Breach notification:** Per HIPAA Breach Notification Rule (45 CFR §§ 164.400–414).

* **Disposal:** DELETE operations on PHI records MUST purge from all backups within the stated timeframe.

### **13.3.2 Financial Services — GLBA**

When OMP data contains Nonpublic Personal Information (NPI):

* **Classification:** x\_contains\_npi boolean metadata MUST be set.

* **Safeguards Rule:** Information security program consistent with 16 CFR Part 314\.

* **Access logs:** Five (5) year retention of third-party access records.

* **Privacy notices:** Clear scope descriptions during Delegate consent flow.

* **Revocation:** Immediate access termination on user revocation.

* **Incident response:** Documented plan for unauthorized access to NPI.

### **13.3.3 Children’s Privacy — COPPA**

When users are known or expected to be under 13:

* **Age gating:** Age verification required before storing any OMP data.

* **Parental consent:** Verifiable parental consent per 16 CFR § 312.5.

* **Data minimization:** Minimum necessary collection. Profiling and behavioral analysis PROHIBITED.

* **Parental control:** Parents MUST be able to review, delete, and terminate data collection.

* **No third-party sharing:** Delegate access PROHIBITED unless parent-authorized and service-essential.

* **Retention limits:** Configurable auto-deletion (30, 90, or 180 days).

## **13.4 Compliance Metadata Fields**

| Field | Type | Profile | Description |
| :---- | :---- | :---- | :---- |
| x\_contains\_phi | boolean | HIPAA | Record contains Protected Health Information. |
| x\_contains\_npi | boolean | GLBA | Record contains Nonpublic Personal Information. |
| x\_child\_user | boolean | COPPA | Record belongs to a user under 13\. |
| x\_parental\_consent\_id | string | COPPA | Reference to parental consent authorization. |
| x\_risk\_level | "low" | "medium" | "high" | "critical" | TRAIGA | AI governance risk classification. |
| x\_human\_review | boolean | TRAIGA | Human reviewed/approved this AI output. |
| x\_retention\_expires\_at | ISO 8601 | COPPA / General | Auto-deletion timestamp. |
| x\_regulatory\_hold | boolean | General | Prevents deletion for legal hold purposes. |

# **14\. Enterprise Conformance (Level 3\)**

Level 3: Enterprise builds on Levels 1 and 2 (Part I) and adds:

* Multi-tenant data isolation (Section 10.1).

* Role-based access control with Admin and Delegate roles (Section 10.2).

* Immutable audit logging (Section 11.1).

* AI provenance enforcement (Section 11.2).

* Conflict resolution with ETags (Section 12).

* Support for declared compliance profiles and their metadata fields (Section 13).

* Administrative API for tenant and user management.

Enterprise Memory Hosts MUST declare their compliance profiles in the conformance endpoint:

{

  "omp\_version": "2.0",

  "conformance\_level": "enterprise",

  "compliance\_profiles": \["hipaa", "glba", "coppa", "traiga", "tdpsa"\],

  "baa\_available": true,

  "data\_residency\_regions": \["us-east", "us-west", "eu-west"\]

}

# **15\. Governance**

## **15.1 OMP Working Group**

The specification is maintained by an open Working Group with no single-vendor veto. The Working Group comprises implementing organizations, independent experts, privacy advocates, and user representatives.

## **15.2 Change Process (OMP Enhancement Proposals)**

1. Any participant submits an OEP via the public GitHub repository.

2. 30-day public comment period for substantive changes.

3. Working Group consensus required. Simple majority vote as fallback.

4. Approved changes are incorporated into the next spec revision.

## **15.3 Intellectual Property**

The specification is published under CC BY 4.0. Code (reference implementation, SDKs, connectors) is published under Apache 2.0. Contributions require a Contributor License Agreement granting the Working Group a perpetual, irrevocable, royalty-free license.

## **15.4 Standards Body Path**

The Working Group intends to seek formal recognition from the Linux Foundation (Joint Development Foundation) as the initial governance home, with a parallel IETF Internet-Draft submission for the core protocol. Long-term, ISO/IEC recognition may be pursued.

# **16\. Extension Mechanism and Versioning**

## **16.1 Custom Fields**

Implementations MAY add custom fields prefixed with x\_ (e.g., x\_sentiment\_score). Compliant implementations MUST ignore unrecognized x\_ fields on import and MUST preserve them on export.

## **16.2 Webhooks (Optional)**

Memory Hosts MAY support webhook notifications for: conversation.created, memory.created, backup.completed, access.revoked.

## **16.3 Versioning**

* **Major (v2 → v3):** Breaking changes. Previous version supported for 24 months.

* **Minor (2.0 → 2.1):** Backward-compatible additions. Existing clients MUST NOT break.

* **Patch (2.0.0 → 2.0.1):** Clarifications and non-functional changes.

# **Appendices**

## **Appendix A: Media Types**

| Media Type | Description |
| :---- | :---- |
| application/vnd.omp.conversation+json | A single OMP Conversation object. |
| application/vnd.omp.memory+json | A single OMP Memory Record object. |
| application/vnd.omp.archive+zip | A complete OMP Archive (.omp.zip). |

## **Appendix B: Regulatory Compliance Matrix**

| Regulation | OMP Mechanism |
| :---- | :---- |
| GDPR (EU) | BACKUP (Art. 20), DELETE (Art. 17), consent scopes (Art. 6–7), audit logs (Art. 30). |
| CCPA / CPRA (CA) | SEARCH (right to know), DELETE (right to delete), BACKUP/MIGRATE (portability). |
| TRAIGA (TX) | model/platform provenance, x\_risk\_level, human oversight annotations. |
| TDPSA (TX) | Full rights mapping: access, delete, portability, consent, ad opt-out. |
| FTC Act § 5 | Anti-lock-in via portability, model traceability, security requirements. |
| HIPAA | x\_contains\_phi, 6-year audit retention, RBAC, BAA, breach notification. |
| GLBA | x\_contains\_npi, Safeguards Rule, 5-year access logs, incident response. |
| COPPA | Age gating, parental consent, data minimization, retention limits. |

## **Appendix C: Relationship to Other Standards**

| Standard | Relationship |
| :---- | :---- |
| MCP (Anthropic / LF) | MCP connects AI to tools. OMP stores the resulting conversations. Complementary, not competing. |
| A2A (Google / LF) | A2A governs agent-to-agent communication. OMP stores what those agents discussed. Complementary. |
| Mem0 / OpenMemory | Mem0 is a memory product. OMP is a portability standard. Mem0 could implement OMP to enable export/import. |
| AMP (pūrmemo) | AMP defines a conversation export format. OMP extends this with Memory Records, compliance, and a full API surface. |
| OAuth 2.0 (RFC 6749\) | OMP uses OAuth for authentication. OMP does not replace or redefine OAuth. |
| DTI (Data Transfer Initiative) | DTI works on policy-level data portability. OMP provides the technical implementation DTI’s policies require. |

## **Appendix D: Example OMP Conversation File**

{

  "id": "550e8400-e29b-41d4-a716-446655440000",

  "title": "Help me plan a garden",

  "created\_at": "2026-03-15T09:30:00Z",

  "updated\_at": "2026-03-15T09:45:00Z",

  "platform": "chatgpt",

  "model": "gpt-4o",

  "message\_count": 4,

  "tags": \["gardening", "home"\],

  "messages": \[

    {

      "id": "msg-001",

      "role": "user",

      "content": "I have a 10x12 backyard in Houston, TX...",

      "timestamp": "2026-03-15T09:30:00Z",

      "model": null,

      "platform": "chatgpt"

    },

    {

      "id": "msg-002",

      "role": "assistant",

      "content": "Great\! Houston is in USDA Zone 9a...",

      "timestamp": "2026-03-15T09:30:12Z",

      "model": "gpt-4o",

      "platform": "chatgpt"

    }

  \],

  "extensions": {

    "chatgpt\_conversation\_template\_id": null

  }

}

## **Appendix E: Acknowledgments**

This specification was developed with input from AI practitioners, enterprise architects, privacy advocates, digital rights organizations, and the broader open-source community. The OMP Working Group welcomes contributions from all interested parties.

*End of Open Memory Protocol (OMP) Specification v2.0 — Draft*

**Your AI conversations belong to you.**