# AGENTS.md

## Repo shape
Pure OpenAPI 3.0.3 definition — no application code or tests. Entry point is
`openapi/openapi.yaml`; `redocly.yaml` registers it as API `sample@v1`.

## Commands
- `npm ci` — reproducible install (CI and sandbox). Use `npm install` only when changing dependencies.
- `npm test` — `redocly lint`; the only verification. Exits 0 with one expected config warning.
- `npm run build` — bundles to `dist/bundle.yaml` (gitignored).
- `npm start` is broken with the pinned CLI 2.x (`preview-docs` was removed). Preview locally with
  `npx redocly build-docs openapi/openapi.yaml -o out.html` (verified) or `npx redocly preview`.

## Sandbox (opencode-sandbox-kit)
Run this repo in an sbx sandbox. The canonical OpenCode `sbx run` command and the
`sbx settings set kit.allowedSources` setup are in `README.md` under "Sandbox (local development
environment)". Commands are PowerShell, multiline, without `--name`, with workspace mount `.` plus the
read-only Maven cache mount, `--skills=off`, `--static-mcp idea,k8s,docker`, and the
`sbx-opencode-tooling:latest` image.

Sandbox quirk: filesystem passthrough blocks symlinks, so npm installs fail with `EPERM` unless bin links
are skipped. The kit sets `npm_config_bin_links=false` globally; do not re-export it. Because this also
leaves `node_modules/.bin` empty, the npm scripts invoke the Redocly CLI via
`node node_modules/@redocly/cli/bin/cli.js`; keep that form when adding scripts.

## Editing the spec
- Put path files in `openapi/paths/v1/` and register each one in `openapi/openapi.yaml`. Unregistered
  files are silently absent from the API and not linted.
- Refs are relative to the referencing file: `../../components/...` from path files, `paths/v1/...`
  from `openapi/openapi.yaml`.
- Schemas live in `openapi/components/schemas/{common,v1}/`; shared error schema is
  `components/schemas/Problem.yaml`. Responses/parameters have matching component folders.
- `no-unused-components: error` means every component must be referenced or lint fails.
- `no-server-example.com: off` intentionally allows the `example.com` server; don't re-enable it.
- `info.version` in `openapi/openapi.yaml` is independent of `package.json`; CI only bumps
  `package.json` for publishing.

## Redocly config gotcha
`redocly.yaml` still uses the v1 top-level `theme:` key. Redocly 2.x ignores it and prints
`Property 'theme' is not expected here` on every lint/build. The `docs/index.html` template and theme
colors currently have no effect; move them under a top-level `openapi:` block if you need them. Do not
assume new spec edits caused this warning.

## CI / release
- Default branch is `development`. Workflow triggers on push to
  `main|master|development|feature/**|release/**`, PRs to `main|master|development`, and nightly.
- CI order: `npm ci` -> `npm test` -> `npm run build` -> rewrite version -> `npm publish` to GitHub
  Packages (`@dboeckli` on npm.pkg.github.com) -> upload `dist/bundle.yaml`.
- Every push to `development` or `feature/**` publishes a package, so work on `feature/*` branches.
- CI uses Node 22; `@redocly/cli` is pinned exactly (2.5.0) and updated by Renovate, where the
  `github-actions` manager is disabled. Dependabot only manages GitHub Actions.

## Docs to distrust
Apart from the Sandbox section, `README.md` and `openapi/**/README.md` are generic Redocly template docs.
Trust `package.json` and `redocly.yaml` over their prose.
