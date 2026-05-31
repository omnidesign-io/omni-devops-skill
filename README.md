# Omni DevOps Skill

Version: 1.2.0

`omni-devops` is an AI agent skill for initializing or reviewing Cloudflare-first simple website projects with multiple long-lived branches and automatic Cloudflare Workers deployment.

The first release packages the workflow currently used by Omni Design: one repository, multiple stable branches, one Cloudflare Worker per environment, guarded deploy scripts, and manual Cloudflare dashboard setup. GitHub Actions is intentionally not part of the design.

## When To Use This Skill

Use this skill when a project needs:

- A simple frontend or static website deployed with Cloudflare Workers Static Assets.
- Multiple long-lived branches such as `main`, `uat`, `dev`, or project-specific equivalents.
- One explicit Cloudflare Worker target per branch/environment.
- Branch-safe deployment scripts that prevent accidental deploys to the wrong Worker.
- Cloudflare Workers Builds configured manually in the dashboard.
- DNS, route, custom domain, and Worker setup guidance for a technical service provider.
- A review of an existing Cloudflare/Wrangler setup before changing deployment behavior.

Do not use this skill when the goal is to build a GitHub Actions pipeline. The intended deployment model is Cloudflare Workers Builds plus guarded package scripts.

## Intended Project Shape

The skill helps AI agents produce or review:

- `wrangler.jsonc` or `wrangler.toml` with explicit Worker names and environment targets.
- `scripts/guard-deploy.mjs` to validate branch and deploy target before Wrangler runs.
- Safe `package.json` deploy scripts such as `deploy:production`, `deploy:uat`, and `deploy:uat:cf`.
- Deployment documentation explaining Cloudflare dashboard fields, DNS records, and branch controls.

## Future Evolution

This repository is meant to grow as the workflow matures. Good future additions include:

- More environment-map examples.
- Guard script templates and tests.
- Cloudflare dashboard checklists for common project types.
- Migration notes from Pages or ad hoc Wrangler deploys.
- Review checklists for bindings such as KV, D1, R2, queues, Durable Objects, and secrets.

Keep the skill itself concise. Put durable procedural knowledge in `SKILL.md`; add references or scripts only when they make repeated future work more reliable.

## Installation

Install this repository as an agent skill by copying or linking `SKILL.md` to the provider-agnostic global directory `~/.agents/skills/omni-devops/SKILL.md`, or copy `SKILL.md` into a local `omni-devops` skill directory.

## Release

- 1.2.0: Made the skill agent-agnostic rather than Codex-specific, and updated all documentation references to point to the new provider-agnostic global directory `~/.agents/`.
- 1.1.0: Introduced search indexing (noindex) controls for non-production environments to act as a deployment guardrail.
- Initial public release: 1.0.0
