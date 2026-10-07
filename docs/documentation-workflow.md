# Documentation Workflow (Docs-as-Code)

## 1. Source
- OpenAPI spec + Markdown files in Git repo.

## 2. Authoring
- Branch per change (`docs/add-webhooks`).
- Markdown for narrative, YAML for API spec.

## 3. Review
- PR with diff preview.
- Checklist: accuracy (spec match), clarity, links, examples.

## 4. Validation (CI mindset)
- OpenAPI lint (Stoplight/Spectral rules).
- Link checker.
- Spell-check (EN/RU).

## 5. Publish
- Merge to `main` → deploy to GitHub Pages / static site.
- Changelog updated automatically or manually.

## 6. Feedback loop
- Issues/Discussions for devs; iterate on docs based on support tickets.

