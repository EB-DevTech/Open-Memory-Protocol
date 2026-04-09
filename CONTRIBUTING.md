# Contributing to the Open Memory Protocol

Thank you for your interest in contributing to OMP! Every contribution matters — from fixing typos in the spec to building entirely new platform connectors.

## How to Contribute

### 1. Report Issues

Found a bug, inconsistency, or have a suggestion? [Open an issue](https://github.com/open-memory-protocol/open-memory-protocol/issues/new/choose) using one of our templates:

- 🐛 **Bug Report** — Something isn't working as documented
- 💡 **Feature Request** — Suggest an improvement
- 📝 **OEP (OMP Enhancement Proposal)** — Propose a change to the specification

### 2. Submit a Pull Request

1. **Fork** the repository
2. **Create a branch** from `main`: `git checkout -b feat/my-feature`
3. **Make your changes** following our guidelines below
4. **Test** your changes
5. **Commit** with a clear message: `feat: add Gemini connector`
6. **Push** to your fork and **open a PR**

### 3. Build a Connector

We especially welcome connectors for new platforms. See [Building Connectors](./docs/building-connectors.md) for a step-by-step guide.

## Contribution Types

| Type | Where | Review Process |
|------|-------|---------------|
| Spec changes | `spec/` | OEP process + 30-day comment period |
| Connector (new) | `connectors/<platform>/` | Standard PR review |
| Connector (fix) | `connectors/<platform>/` | Standard PR review |
| SDK changes | `sdk/` | Standard PR review |
| Documentation | `docs/` | Standard PR review |
| Reference server | `reference-server/` | Standard PR review |
| Website | `website/` | Standard PR review |

## Guidelines

### Code Style

- **TypeScript/JavaScript**: Use ESLint + Prettier (config included)
- **Python**: Follow PEP 8, use Black formatter
- **Documentation**: Use Markdown, keep language clear and inclusive

### Commit Messages

We use [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add Claude connector
fix: handle empty message arrays in ChatGPT export
docs: clarify Memory Record fields
spec: add webhook notification types (OEP-007)
chore: update dependencies
test: add integration tests for backup endpoint
```

### Testing

- All connectors must include test fixtures and automated tests
- SDK changes must include unit tests
- Reference server changes must include API tests
- Aim for >80% code coverage on new code

### Specification Changes (OEP Process)

Changes to the OMP specification follow the OMP Enhancement Proposal (OEP) process:

1. Open an issue using the OEP template
2. Describe the problem, proposed solution, and impact
3. The Working Group opens a **30-day public comment period**
4. After discussion, the Working Group votes (consensus preferred, majority fallback)
5. Approved OEPs are merged and incorporated into the next spec revision

## Contributor License Agreement (CLA)

By contributing to OMP, you agree to grant the OMP Working Group a perpetual, irrevocable, royalty-free license to use, modify, and distribute your contribution under the project's licenses (CC BY 4.0 for spec, Apache 2.0 for code).

We use a lightweight CLA-bot that will ask you to sign on your first PR.

## Community Standards

All participants must follow our [Code of Conduct](./CODE_OF_CONDUCT.md). We're building an inclusive community where everyone is welcome. Be kind, constructive, and respectful.

## Getting Help

- 💬 [GitHub Discussions](https://github.com/open-memory-protocol/open-memory-protocol/discussions) — Ask questions, share ideas
- 🐛 [Issues](https://github.com/open-memory-protocol/open-memory-protocol/issues) — Report bugs
- 📧 Email: contribute@openmemoryprotocol.org

---

**Your contributions help ensure that every person owns their AI conversations. Thank you! 🙏**
