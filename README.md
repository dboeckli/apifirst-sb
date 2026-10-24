# API First with Spring Boot

**API Docs:** <https://dboeckli.github.io/apifirst-sb/>

OpenAPI 3.0.3 definition for the example Customer, Product, and Order API. This repo contains no
application code: Redocly CLI lints and bundles the spec. CI publishes the package to GitHub Packages,
uploads the bundled spec as a workflow artifact, and deploys the rendered docs to GitHub Pages.

## Published docs

- Rendered API reference: <https://dboeckli.github.io/apifirst-sb/>
- Bundled spec download: <https://dboeckli.github.io/apifirst-sb/bundle.yaml>
- Deploy trigger: push to `development` and manual `workflow_dispatch` of `OpenAPI Pages`
  (`.github/workflows/pages.yml`). Feature branches do not deploy, so they cannot overwrite the docs.

## Quick start

```bash
npm ci
npm test
npm run build
```

- `npm test` runs `redocly lint` and is the only verification.
- `npm run build` writes `dist/bundle.yaml` (gitignored).
- `npm run build:docs` writes the rendered API reference to `_site/index.html` (gitignored).
- `npm start` is broken with Redocly CLI 2.x (`preview-docs` was removed). Render static docs with
  `npx redocly build-docs openapi/openapi.yaml -o out.html` instead.

## Layout

| Path | Purpose |
| --- | --- |
| `openapi/openapi.yaml` | Entry point; registers every path and component |
| `openapi/paths/v1/` | One file per path; each file must be registered in `openapi.yaml` |
| `openapi/components/{schemas,responses,parameters,headers}/` | Reusable components |
| `redocly.yaml` | Lint rules and API registration (`sample@v1`) |
| `dist/bundle.yaml` | Generated bundle, not committed |

## Branching and versioning

- `development` is the default and integration branch. Feature branches are merged into it via PR.
- Work on `feature/<topic>`. With an issue, use `feature/<issue-number>-<short-text>` (for example
  `feature/141-sandbox-onboarding`).
- `main` is the legacy initial branch and is not used: nothing is merged into it. There is no
  `master`; `release/**` is accepted by CI but not used.
- CI runs on pushes to `main`, `master`, `development`, `feature/**`, `release/**`; PRs to `main`,
  `master`, `development`; and nightly at 02:05 UTC. In practice, only PRs into `development` are
  used.
- CI derives the package version and publishes `@dboeckli/apifirst-sb` to GitHub Packages:

| Branch | Version |
| --- | --- |
| `development` | `1.0.<run_number>-dev.<short_sha>` |
| any other branch | `0.1.<run_number>-feature.<short_sha>.<timestamp_utc>` |

- Feature builds use the `0.1` line and include a UTC timestamp; they are not prereleases of the
  `1.0` development line.
- `<run_number>` is `github.run_number`, not a release counter. The timestamp is UTC `yyyyMMddHHmm`.
- `npm version ... --no-git-tag-version` changes only the CI workspace; `package.json` stays at
  `1.0.0` in Git. No Git tag or GitHub Release is created.
- `info.version` in `openapi/openapi.yaml` is maintained manually and is independent of the package
  version. CI does not update it.
- `release/**`, `main`, and PRs also use the `0.1...-feature...` scheme; there is no dedicated
  release versioning.
- The publish step has no event guard: feature pushes, same-repo PRs, and the nightly run all
  publish (or attempt to publish) a package.

## Sandbox (local development environment)

This repo is provisioned by [opencode-sandbox-kit](https://codeberg.org/dboeckli/opencode-sandbox-kit) and runs
inside a Docker sandbox. The sandbox mounts this repo, starts the agent, and connects the IntelliJ MCP server.
Run the `sbx` commands in PowerShell on the Windows host.

Allow the kit source once:

```powershell
sbx settings set kit.allowedSources --% "[\"docker.io/\",\"codeberg.org/dboeckli/\"]"
```

Start a new OpenCode sandbox:

```powershell
sbx run opencode `
    --kit "git+https://codeberg.org/dboeckli/opencode-sandbox-kit.git#dir=opencode-agent" `
    --template docker.io/domboeckli/sbx-opencode-tooling:latest `
    --skills=off `
    --static-mcp idea,k8s,docker `
    . `
    "C:\development\maven-repo:ro"
```

## Documentation

- [Spec authoring / Redocly starter guide](docs/redocly-starter.md) — schemas, paths, `$ref`, code
  samples, and starter setup
- [AGENTS.md](AGENTS.md) — instructions for AI/agent sessions
