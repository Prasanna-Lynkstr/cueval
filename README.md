# Cueval — by Lynkstr Labs

> From raw data to release-ready.

Cueval is an AI **dataset curation and evaluation platform** for teams building and
fine-tuning language models. It covers the full pipeline: ingest raw data → clean and
score → human review → experiment tracking → eval → release approval — with
multi-tenant isolation and a Lynkstr Labs super-admin operator view.

## What this repo is

A **single self-contained `index.html`** prototype — vanilla JS + CSS, no frameworks,
no backend, no build step. All data lives in browser memory; mock logic simulates AI
scoring, eval, and LLM calls. The only external dependency is Google Fonts (CDN).

## Running it

Just open `index.html` in a modern browser. For a clean origin (recommended, needed for
some File APIs to behave consistently):

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

Demo accounts (all password `demo123`) are listed on the login screen. Highlights:

| Email | Tenant | Role |
|-------|--------|------|
| `superadmin@cueval.ai` | — | Super Admin (cross-tenant) |
| `admin@cueval.ai` | IITM Pravartak (Enterprise) | Admin |
| `admin@legalai.com` | Legal AI Corp (Pro) | Admin |
| `admin@healthbot.in` | HealthBot India (Starter) | Admin — sees limit warnings |

## Documentation

- **[`CLAUDE.md`](CLAUDE.md)** — the engineering charter. Architecture decisions, coding
  standards, multi-tenancy isolation rules, error/logging discipline, design-system
  compliance, and the definition of done. **Read this before making any change.**
- **[`docs/PRODUCT_SPEC.md`](docs/PRODUCT_SPEC.md)** — the full product specification
  (screens, data model, mock logic, multi-tenancy, super-admin).

## Contributing

Every change must comply with `CLAUDE.md`. In short: one file, no frameworks, all data
access tenant-scoped, design tokens only, everything keyboard-accessible, log through the
`log.*` facade, and verify in a real browser across the affected roles/tenants before
calling it done.
