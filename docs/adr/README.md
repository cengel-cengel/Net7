# AOT — Digital Procurement Platform (Net7)

B2B-Procurement-Plattform für **AOT**, einen B2B-Lieferanten organischer Rohstoffe für Kosmetik- und Lebensmittel-Industrie. Das System bindet das ERP **TopM NET7** an und unterstützt Sample-Requests, RFQ-Workflows, Procurement-Workflows und authentifizierte Customer-Dashboards.

> **Wichtig:** Dies ist **kein klassischer Webshop**. Kein Payment, kein öffentlicher Checkout, kein Consumer-Workflow.

## Status

**Sprint 0 — Discovery & Project Foundation**

## Operating Documents

| Dokument | Zweck |
|---|---|
| [`AGENTS.md`](./AGENTS.md) | Engineering Charter — tool-agnostisch, gilt für alle AI-Werkzeuge im Projekt |
| [`CLAUDE.md`](./CLAUDE.md) | Operative Regeln für Claude Code (wird automatisch gelesen) |
| [`ROADMAP.md`](./ROADMAP.md) | Master Delivery Roadmap, Sprint-Struktur, Rollen |
| [`docs/adr/`](./docs/adr/) | Architecture Decision Records |

## Stack (per [ADR 0001](./docs/adr/0001-stack-and-deployment.md))

- **Frontend:** Next.js (App Router) auf Vercel
- **Backend:** NestJS auf Railway
- **Datenbank:** PostgreSQL auf Railway
- **Repository-Layout:** Turborepo Monorepo (Scaffolding in Sprint 3)

## Mitwirken

Vor jeder Code-Änderung: [`AGENTS.md`](./AGENTS.md) Review-Checkliste durchlaufen
(Architecture / Security / Maintainability / Scalability / ERP Safety / Handover).
