# Strawberry Moon Project — Claude Code Guide

This is a **Strawberry Moon** project. Strawberry Moon is an AI-powered product management platform that organizes product strategy and execution through a structured document hierarchy. All documents are **Markdown files** with specific naming conventions and parent-child relationships.

---

## Document Hierarchy

Strawberry Moon uses two parallel workstreams: **Product** (strategy) and **FLASH** (execution).

### Product — Strategy Hierarchy (top-down)

```
Market Analysis (.market.md)
 └── PRD (Product Requirements Document)    ← 1:many under Market
      ├── Architecture (.architecture.md)   ← 1:many under PRD
      └── UX Design (.ux.md)               ← 1:many under PRD
```

### FLASH — Execution Hierarchy (top-down)

```
Press Release
 └── Epic            ← 1:many under Press Release
      └── User Story  ← 1:many under Epic
```

Press Releases are grouped by **cadence period** (quarters like Q1, Q2, or halves like H1, H2).

---

## File Naming Conventions

All documents use `.md` extension with a **dual-extension** pattern to identify category:

| Category              | Extension          | Example                          |
|-----------------------|--------------------|----------------------------------|
| Market Analysis       | `.market.md`       | `housing-in-uk.market.md`        |
| Product Requirement   | `.prd.md`          | `analytics-platform.prd.md`      |
| Architecture          | `.architecture.md` | `system.architecture.md`         |
| UX Design             | `.ux.md`           | `dashboard.ux.md`                |
| Press Release         | `.pr.md`           | `v2-launch.pr.md`                |
| Epic                  | `.epic.md`         | `1.user-onboarding.epic.md`      |
| User Story            | `.story.md`        | `1.1.signup-flow.story.md`       |
| Roadmap               | `.roadmap.md`      | `2025-roadmap.roadmap.md`        |

### FLASH Numbering

FLASH documents (Epics and User Stories) use **sequential numbering prefixes** that drive sidebar sort order:
- Epics: `1.name.epic.md`, `2.name.epic.md`
- Stories under Epic 1: `1.1.name.story.md`, `1.2.name.story.md`
- Stories under Epic 2: `2.1.name.story.md`, `2.2.name.story.md`

---

## YAML Front Matter

Epics and User Stories use YAML front matter for metadata. Press Releases use `targetDate`.

### Status Values

| Status      | Meaning                        |
|-------------|--------------------------------|
| `Draft`     | Initial creation, not reviewed |
| `Ready`     | Ready for development          |
| `In Progress` | Currently being worked on    |
| `Review`    | Under review                   |
| `Done`      | Completed                      |
| `Blocked`   | Blocked by dependency          |

### MoSCoW Priority (Epics only)

Must, Should, Could, Will Not

### T-Shirt Sizing (Epics only)

XS, S, M, L, XL

---

## AI Agents

The platform uses specialized AI agents. Full agent definitions and personality profiles:

| Agent | Definition | DISC Profile | Role |
|-------|-----------|-------------|------|
| **SCOUT** | `.strawberrymoon/SCOUT.agent.md` | `.strawberrymoon/SCOUT.disc.md` | Market research & segment analysis |
| **SONAR** | `.strawberrymoon/SONAR.agent.md` | `.strawberrymoon/SONAR.disc.md` | Product planning & requirements |
| **COMPASS** | `.strawberrymoon/COMPASS.agent.md` | `.strawberrymoon/COMPASS.disc.md` | System architecture |
| **AURA** | `.strawberrymoon/AURA.agent.md` | `.strawberrymoon/AURA.disc.md` | UX design |
| **FLASH** | `.strawberrymoon/FLASH.agent.md` | `.strawberrymoon/FLASH.disc.md` | Execution planning |
| **CADE** | `.strawberrymoon/CADE.agent.md` | `.strawberrymoon/CADE.disc.md` | Coding & development |

Each agent has:
- An **agent definition** (`.agent.md`) describing capabilities, behavior rules, and document flow
- A **DISC personality profile** (`.disc.md`) describing communication style, decision-making approach, and collaboration patterns

**⚠️ If you are acting as a coding agent, load `.strawberrymoon/CADE.agent.md` and `.strawberrymoon/CADE.disc.md` and follow its status protocol strictly.**
