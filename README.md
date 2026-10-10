# redcell-cli plugins (curated marketplace)

Curated marketplace of security **skills**, **tool wrappers**, and **MCP servers** for
[`redcell-cli`](https://github.com/redcell-sh/redcell-cli).

## Release model: the CLI pins a commit

The CLI does **not** follow this repo's default branch. On startup it shallow-fetches one exact,
pinned commit, verifies `git rev-parse HEAD` matches the pin, and caches it under
`<CLI home>/.tmp/plugins`. There is no zipball/HTTP fallback.

**Merging to `main` ships nothing to users.** Changes reach users only in a CLI release that bumps
the pin. To bump it:

1. Review the full diff between the current pin and the new commit (`git diff <old>..<new>`).
2. In `redcell-cli`, set `REDCELL_PLUGINS_PINNED_SHA` in
   `codex-rs/core-plugins/src/startup_sync.rs` to the new full 40-char commit SHA.
3. Ship it through the normal CLI review/release process.

## Layout

```
.agents/plugins/marketplace.json      # catalog (required, at repo root, on default branch)
.agents/plugins/api_marketplace.json  # same content (consumed by the api-curated path)
plugins/<name>/.redcell-plugin/plugin.json   # per-plugin manifest
plugins/<name>/skills/<skill>/SKILL.md     # optional skill pack
plugins/<name>/.mcp.json                   # optional MCP server(s)
```

## Packaging tiers

| Tier | Ships | Example |
|---|---|---|
| Skill pack | `skills/` dir of `SKILL.md` methodology | `pentest-recon` |
| MCP server | `.mcp.json` (stdio `command`/`docker run -i` or remote) | `web-nuclei` |
| Wrapped tool | thin MCP server shelling to a host binary | (brutespray, medusa, …) |

## Rules
- `installation`: `INSTALLED_BY_DEFAULT` (auto) · `AVAILABLE` (opt-in via `redcell plugin add`) · `NOT_AVAILABLE`.
- `authentication`: `ON_INSTALL` · `ON_USE`. (There is **no** `NONE`.)
- `category` drives the CLI's category enable/disable filter.
- Keep **weaponized payloads out of this public repo** — wrap already-installed local binaries only. Route offensive/sensitive content through the authenticated backend catalog.
- Vendored third-party code keeps its `LICENSE`; AGPL/GPL/non-commercial code is subprocess- or reference-only, never bundled.
