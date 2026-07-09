# Kodama Docs

The official documentation repository for the Kodama ecosystem.

This repo contains **ecosystem-wide** documentation only: architecture, standards, specifications, and operational guidance shared across all Kodama projects. Product-specific documentation lives in its respective repository.

## What belongs here

| Directory | Purpose |
|---|---|
| [`architecture/`](architecture/) | How the entire ecosystem fits together |
| [`standards/`](standards/) | Conventions every repository must follow |
| [`operations/`](operations/) | How the platform is operated day-to-day |
| [`deployment/`](deployment/) | Shared deployment and infrastructure |
| [`adr/`](adr/) | Architecture Decision Records — why decisions were made |
| [`roadmap/`](roadmap/) | Ecosystem-wide planning and milestones |

## What does not belong here

These topics live in their respective repositories:

| Topic | Repository |
|---|---|
| Brand, mission, voice & tone | `kodama-brand` / `brand.kodama.page` |
| Note product specs, API, UI | `kodama-note` / `note.kodama.page` |
| Security protocol details | `kodama-security-protocol` |
| Marketing site, SEO, landing pages | `kodama.page` / `kodama-web` |

## Kodama at a glance

Kodama is a forest of single-purpose **Places** — internet utilities at `{product}.kodama.page/{slug}`. No accounts. No profiles. One URL, one purpose.

| Place | Domain | Status |
|---|---|---|
| Note | `note.kodama.page` | Live |
| Drop | `drop.kodama.page` | Beta |
| Secret | `secret.kodama.page` | Coming soon |
| Poll | `poll.kodama.page` | Coming soon |
| Room | `room.kodama.page` | Coming soon |
| Link | `link.kodama.page` | Coming soon |

All Places share a common **Platform API** at `api.kodama.page`, backed by Supabase PostgreSQL.

## Repository map

| Repository | Role |
|---|---|
| `kodama-web` | Marketing hub at `kodama.page` |
| `api.kodama.page` | Shared Platform API + database schema |
| `note.kodama.page` | Kodama Note frontend |
| `drop.kodama.page` | Kodama Drop frontend |
| `brand.kodama.page` | Brand guidelines site |
| `kodama-note` | Note product specifications |
| `kodama-security-protocol` | Security protocol specification |
| `kodama-docs` | This repository |

Platform repositories are maintained under the **Kodama-Platform** GitHub organization. Product deployment repositories may live under individual accounts.

## Contributing

1. Read the relevant [standards](standards/) before opening a PR.
2. For architectural changes, add or update an [ADR](adr/).
3. Keep product-specific details out of this repo — link to the product repository instead.
4. Follow the [commit convention](standards/Commit%20Convention.md) and [branch strategy](standards/Branch%20Strategy.md).
