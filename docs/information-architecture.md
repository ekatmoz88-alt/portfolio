# Information Architecture (IA) for API Documentation

## 1. Core Principles
- **Developer-first**: Docs must answer "How do I make it work?" faster than "What is it?".
- **Single Source of Truth**: OpenAPI spec (`openapi.yaml`) drives endpoint references; Markdown fills in context/tutorials.
- **Progressive Disclosure**: High-level overview → Guides → Detailed API reference.

## 2. Documentation Layers
1. **Overview & Concepts**: Business context, auth flows, glossary.
2. **Quickstart / Getting Started**: 5-minute setup (keys, base URL, first cURL).
3. **Guides / How-to's**: Task-oriented (e.g., "Create an Order", "Handle Webhooks").
4. **API Reference**: Auto-generated/structured endpoints, params, schemas, status codes.
5. **Support & Extras**: Error codes, changelog, SDKs/libraries, FAQ.

## 3. Navigation & Findability
- Flat structure for small projects; grouped by resource (Orders, Users, Webhooks) for larger ones.
- Persistent sidebar (concept in portfolio site / GitHub TOC).
- Cross-linking between guides and exact API endpoints.

## 4. Localization (EN/RU)
- Default language: English (`README.md`, `docs/*.md`).
- Russian version: `README.ru.md` + mirrored guides if needed.
- Terminology consistency: glossary + style guide.

## 5. Maintenance
- Versioning via Git branches/tags.
- Changelog reflects spec changes.
- CI checks (link checks, OpenAPI linting) prevent broken docs.

