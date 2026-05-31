---
name: omni-devops
description: Set up or review Cloudflare Workers Static Assets deployments for multiple long-lived branches with explicit Worker targets, Wrangler environments, branch guard scripts, safe npm deploy commands, dashboard guidance, and DNS/manual Cloudflare checklists. Use when a project needs production/UAT/dev/lab or other branch-to-Worker deployment mapping, when auditing Cloudflare Workers Builds settings, when replacing unsafe generic deploy commands, or when explaining manual Cloudflare setup to a technical service provider.
metadata:
  version: "1.0.0"
---

# Omni DevOps

Use this skill to turn a frontend/static project into a guarded Cloudflare Workers deployment setup with one Cloudflare Worker per long-lived branch or environment. Keep the setup configurable: do not assume branch names, Worker names, domains, or the number of environments.

## Core Pattern

Use Cloudflare Workers Static Assets as the deployment base. Model each long-lived environment as:

- Git branch: `main`, `uat`, `dev`, or a project-specific branch.
- Wrangler target: top-level production config or an `env.<key>` config.
- Cloudflare Worker: one stable Worker per long-lived branch.
- Stable hostname: apex, subdomain, or route per Worker.
- Deploy command: branch guard first, then Wrangler with the intended environment.

Temporary branches such as feature, fix, copy, experiment, and lab branches deploy only if the user explicitly wants a long-lived environment for them. Otherwise, block them from automated deployment.

## Workflow

1. Inspect first.
   Read `package.json`, lockfiles, `wrangler.jsonc` or `wrangler.toml`, framework config, existing deployment docs, and current branch. Identify package manager, build command, build output, Worker names, routes/custom domains, and existing scripts.

2. Ask for the environment map only when it is not discoverable.
   Use compact questions in chat:
   - Which long-lived branches should deploy?
   - What Worker name and hostname should each branch use?
   - Which branch families should never deploy?

3. Configure Wrangler.
   Keep production top-level unless the repo already uses another pattern. Add one `env.<key>` per additional long-lived environment. For every environment, set its actual Worker `name`, route/custom domain, assets directory, observability, and non-secret vars such as `ENVIRONMENT`.

4. Add a guard script.
   Create `scripts/guard-deploy.mjs`. It should accept a target key, detect branch from CI variables first and local git second, print target/branch/mode, allow only the configured branch for that target, block configured temporary branch patterns, and require a clean working tree for local deploys only.

5. Add safe package scripts.
   Make generic `deploy` fail with an intentional message. Add local scripts that run guard -> build -> Wrangler, and CI scripts that run guard -> Wrangler because Cloudflare Workers Builds has a separate build command field.

6. Document manual Cloudflare steps.
   Create or update deployment docs in the repo. Explain dashboard setup, DNS records, branch controls, deploy commands, and why generic `npx wrangler deploy` is not recommended for multi-Worker repos.

7. Validate without deploying.
   Run guard tests if present, typecheck, lint where practical, build, formatting checks for touched files, and `git diff --check`. Do not run `wrangler deploy` unless the user explicitly asks.

## Wrangler Guidance

Prefer `wrangler.jsonc` for new work. Use route style consistently with the existing production setup:

- If production uses `routes`, use `routes` for other hostnames.
- If production uses Custom Domains, use equivalent Custom Domain style for other hostnames.
- If a new subdomain is used with route style, remind the user to create a proxied DNS record.

Remember that many Wrangler keys are not inherited into environments. Explicitly review bindings such as `vars`, KV, R2, D1, Durable Objects, services, queues, secrets, and observability. Duplicate only non-secret config that is needed for the target environment. Never create or expose secrets.

Example environment shape:

```jsonc
{
  "name": "project-prod",
  "assets": { "directory": "./dist" },
  "vars": { "ENVIRONMENT": "production" },
  "routes": [{ "pattern": "example.com/*", "zone_id": "<zone-id>" }],
  "env": {
    "uat": {
      "name": "project-uat",
      "assets": { "directory": "./dist" },
      "vars": { "ENVIRONMENT": "uat" },
      "routes": [{ "pattern": "uat.example.com/*", "zone_id": "<zone-id>" }]
    }
  }
}
```

## Package Script Pattern

Use names that match the user's environments. A common pattern:

```json
{
  "scripts": {
    "deploy": "node -e \"console.error('Use an explicit deploy target intentionally.'); process.exit(1)\"",
    "deploy:production": "node scripts/guard-deploy.mjs production && npm run build && wrangler deploy --env=\"\"",
    "deploy:uat": "node scripts/guard-deploy.mjs uat && npm run build && wrangler deploy --env uat",
    "deploy:production:cf": "node scripts/guard-deploy.mjs production && wrangler deploy --env=\"\"",
    "deploy:uat:cf": "node scripts/guard-deploy.mjs uat && wrangler deploy --env uat"
  }
}
```

Use the package manager already present in the repo: npm, pnpm, yarn, or bun. Adjust commands accordingly.

Explain `:cf` scripts plainly: Cloudflare Workers Builds already runs the separate Build command, so the deploy command should not build again. The `:cf` script still runs the guard and passes the correct Wrangler environment.

## Guard Script Requirements

Build the guard from a data table so environments are customizable. Include:

- Target argument validation.
- Branch detection order: `WORKERS_CI_BRANCH`, `GITHUB_REF_NAME`, then `git branch --show-current`.
- Optional CI detection with `WORKERS_CI`, `WORKERS_CI_BRANCH`, or common CI vars.
- Exact branch allow-list per target.
- Blocked branch patterns such as `feature/*`, `fix/*`, `copy/*`, `lab/*`, or user-provided equivalents.
- Local clean-tree check using `git status --porcelain`.
- Clear stdout/stderr messages.
- Non-zero exit on unsafe attempts.

If tests are appropriate, use Node's built-in test runner so no dependency is needed.

## Cloudflare Manual Checklist

Give the user simple chat instructions after configuring the repo:

1. In Cloudflare DNS, add any new hostname.
   For route-style subdomains, usually:
   - Type: `CNAME`
   - Name: environment subdomain, such as `uat`
   - Target: apex domain, such as `example.com`
   - Proxy: enabled
   - TTL: Auto

2. In Workers & Pages, create or rename each Worker to match Wrangler `name`.

3. In each Worker, connect the same Git repository through Workers Builds.

4. Set Production branch to the environment branch for that Worker.

5. Disable builds for non-production branches unless the user explicitly wants preview deployments.

6. Set Build command to the project build command, such as `npm run build`.

7. Set Deploy command to the guarded Cloudflare script, such as `npm run deploy:uat:cf`.

8. Confirm route/domain after first deploy.

Explain why not to use Cloudflare's default `npx wrangler deploy` in multi-Worker repos: it targets the top-level Wrangler config unless `--env <target>` is provided, so it can deploy the wrong Worker. The guarded package script is safer because it checks branch and target before calling Wrangler.

## Review Checklist

When reviewing an existing project, report:

- Package manager and build command.
- Build output directory and assets config.
- Worker names and branch mapping.
- Routes/custom domains and DNS records needed.
- Whether generic deploy is blocked.
- Whether every deploy command runs a guard first.
- Whether Cloudflare Build commands should use local deploy scripts or `:cf` scripts.
- Whether dashboard branch controls disable unwanted branch builds.
- Any mismatch between dashboard Worker names and Wrangler `name` values.

Keep the tone practical and interactive. The user is technical, but may be operating the Cloudflare dashboard manually; give short checklists and tell them exactly which field to fill.
