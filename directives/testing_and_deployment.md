# Testing and deployment verification gate

## Goal

Make local pushes and production deployments fail early when type, lint, test,
or integration checks are broken. Cloudflare and other CI providers must never
be the first place routine quality errors are discovered.

## Required public commands

Every generated repository must define equivalent commands for its package
manager and stack. For npm projects, use these exact script names:

| When | Command | Purpose |
| --- | --- | --- |
| Before every push | `npm run preflight` | Fast typecheck and lint gate |
| Before deploys, large changes, and final handoff | `npm run test:gate` | Complete project verification suite |
| Production build and CI/Pages | `npm run build` | Run the full gate, then bundle |

`preflight` should normally finish quickly enough to run before every push. At
minimum it must include all configured static checks:

```json
{
  "scripts": {
    "preflight": "npm run typecheck && npm run lint"
  }
}
```

`test:gate` must include every verification tier the repository actually uses,
for example:

```json
{
  "scripts": {
    "test:gate": "npm run typecheck && npm run lint && npm test && npm run test:e2e"
  }
}
```

Add Python, integration, migration, security, or data-pipeline suites when they
exist. Do not add passing no-op scripts for missing tiers; either configure the
real check or document why the tier does not apply.

## Verified production build

The production build must run the complete gate before any bundle is created.
For projects that need a shell wrapper, use a checked-in script such as
`scripts/build-verified.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

npm run test:gate
npm run build:bundle
```

Then expose it through `package.json`:

```json
{
  "scripts": {
    "build": "bash scripts/build-verified.sh",
    "build:bundle": "vite build"
  }
}
```

Adapt `build:bundle` to the framework. Avoid recursive definitions where
`build-verified.sh` invokes `npm run build` again.

Cloudflare Pages or Workers must call the verified `npm run build`; deployment
configuration must not bypass it by calling the bundler directly.

## Standard workflow

1. Implement the change and its focused tests.
2. Run the relevant focused checks while iterating.
3. Run `npm run preflight` immediately before every push.
4. Run `npm run test:gate` before a deploy, production build, large-change
   handoff, or completion claim.
5. Fix failures at their source and rerun the failed command.
6. Rerun the required combined gate from the beginning before continuing.

Warnings may remain only when the project's written policy explicitly allows
them. Errors always block push and deployment.

## Repository bootstrap checklist

When creating a repository from this template:

1. Detect the actual language, framework, package manager, and deployment
   target.
2. Configure real typecheck, lint, unit, integration, and E2E commands that
   apply to the project.
3. Add the `preflight` and `test:gate` entry points.
4. Make the production build execute `test:gate` before bundling.
5. Configure CI and Cloudflare to use the verified production build.
6. Run both gates once and record any project-specific exceptions below.

## Failure protocol

If a gate fails:

1. Stop the push or deployment.
2. Read the complete error and reproduce the failing tier directly.
3. Fix the implementation, configuration, or legitimate test contract.
4. Do not disable, skip, or weaken the check merely to make the gate pass.
5. Rerun the failed tier, then rerun the required combined gate.
6. Add durable lessons or project-specific edge cases to this directive.

## Project-specific checks

Record additional commands, allowed warnings, CI requirements, deployment
details, and known test isolation constraints here as the repository evolves.
