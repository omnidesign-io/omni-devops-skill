# Changelog

All notable changes to the `omni-devops` skill will be documented in this file.

## [1.2.0] - 2026-06-01

This release makes the skill provider-agnostic and relocates the master installation path to the global `~/.agents/` folder.

### Changed

- **Provider-Agnostic Skill Location**: Relocated master skill file from `/Users/hanny/.codex/skills/omni-devops/SKILL.md` to `/Users/hanny/.agents/skills/omni-devops/SKILL.md` (`~/.agents/`).
- **Terminology & Reference Updates**: Cleaned up the workflow and references in `README.md` and `AGENTS.md` to refer to general AI agents rather than Codex specifically.

## [1.1.0] - 2026-05-31

This release introduces search indexing controls for non-production environments to act as a crucial deployment guardrail.

### Added

- **Search Indexing Controls**: Model search indexing as a deployment guardrail. All deployed non-production environments (e.g. UAT, staging, preview, demo, QA, and dev) must be configured with `noindex` signals by default.
- **Build-Time Public Env Variable**: Guidance on using build-time variables (e.g., `PUBLIC_NOINDEX=true`) to set the `noindex` flag at build time for static site frameworks (like Astro, Next.js, Vite, Eleventy, etc.).
- **Page-Level robots meta tags**: Added recommendation to use robots `noindex` meta tags in generated HTML:
  ```html
  <meta name="robots" content="noindex, nofollow, noarchive" />
  <meta name="googlebot" content="noindex, nofollow, noarchive" />
  ```
- **Robots.txt restrictions**: Recommended ensuring non-production `robots.txt` omits the production sitemap to avoid indexing confusion.
- **Cloudflare Workers Builds integration**:
  - Detailed build configuration settings for non-production environments utilizing the noindex build commands.
  - Setup validation check to view page source and robots.txt outputs on deployed subdomains.
- **Review checklist updates**: Included audits for non-production `noindex` build configurations, robots meta tags, and `robots.txt` properties.
