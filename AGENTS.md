# Agent entry point

Before doing any work in this repository, read and follow
[`.agents/AGENTS.md`](.agents/AGENTS.md). It contains the repository operating
principles and directive routing.

For repository setup, testing, builds, pushes, CI, or deployment, read and
follow [`directives/testing_and_deployment.md`](directives/testing_and_deployment.md).

Required quality gates:

- Before every `git push`, run the project's fast `preflight` gate.
- Before a deploy, production build, large change, or final handoff, run the
  complete `test:gate`.
- Never push, deploy, or claim completion while a required gate is failing.
- In npm projects, expose these gates as `npm run preflight` and
  `npm run test:gate`. Production `npm run build` must run the full gate before
  creating the bundle.
