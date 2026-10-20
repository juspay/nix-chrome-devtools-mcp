# nix-chrome-devtools-mcp

One-click setup for the [Chrome DevTools MCP server](https://github.com/ChromeDevTools/chrome-devtools-mcp) in any Nix-based project. Delivered as an [Agent Plugin](https://agent-plugins.org/specification) (v1.0.0): reference `github:juspay/nix-chrome-devtools-mcp` from an agent-distro profile and clients get the `chrome-devtools` MCP server, with Chrome (Playwright's bundled Chrome-for-Testing) and Node.js resolving through Nix. No devshell to author, no flake to compose. The `nixpkgs` source is overridable per process or system-wide.

See [introductory blog post](https://kolu.dev/blog/nix-chrome-devtools-mcp/).

## Use as an Agent Plugin

The repository root is an [Agent Plugin](https://agent-plugins.org/specification) (v1.0.0), so any Agent Plugins client can reference it directly:

```sh
AI_PROFILE=github:juspay/nix-chrome-devtools-mcp agent-distro omp
```

Clients that read the plugin get both components: the `nix-chrome-devtools-mcp` skill (`skills/`) and the `chrome-devtools` MCP server (`mcp.json`), whose command is the plugin-relative `./skills/nix-chrome-devtools-mcp/bin/serve`.

## Migrating from APM

APM support is removed. Move the dependency into your profile's `plugin.json`:

```yaml
# before — apm.yml
dependencies:
  apm:
    - juspay/nix-chrome-devtools-mcp
```

```json
// after — plugin.json
{
  "$schema": "https://agent-plugins.org/schemas/1.0.0/plugin.schema.json",
  "name": "my-profile",
  "extensions": {
    "asia.nixos.agent-distro": {
      "plugins": ["github:juspay/nix-chrome-devtools-mcp"]
    }
  }
}
```

`apm install` becomes nothing to run: agent-distro reads the plugin at launch, and the `chrome-devtools` server comes with it. See [Profiles](https://agent-distro.nixos.asia/#profiles).

## Requirements

Nix — see [nixos.asia/en/install](https://nixos.asia/en/install). Flakes must be enabled (`experimental-features = nix-command flakes`).

## Overrides

The launcher reads two env vars:

| Var | Default | Effect |
|---|---|---|
| `NIXPKGS_FLAKE` | `nixpkgs` | flake-ref used to resolve `playwright-driver.browsers` and `nodejs`. Examples: `github:NixOS/nixpkgs/<rev>`, `path:/abs/path/to/local/nixpkgs`. |
| `CHROME_DEVTOOLS_MCP_VERSION` | `latest` | npm dist-tag or version of `chrome-devtools-mcp` to install via `npx -y`. Pin to a specific version (e.g. `0.26.0`) when you need reproducibility. |

System-wide overrides also work:

- `nix registry pin nixpkgs github:NixOS/nixpkgs/<rev>` (replaces the default `nixpkgs` registry alias).

## Screencast (video) recording

The launcher runs the server with `--experimentalScreencast=true` and puts
`ffmpeg` on its `PATH` (resolved through Nix, same as Chrome and Node), so the
`screencast_start` / `screencast_stop` tools are available out of the box. They
record the selected page to an `.mp4`/`.webm` file via Puppeteer's
`page.screencast()`, which shells out to ffmpeg — no host ffmpeg install needed.

## Chrome version compatibility

`chrome-devtools-mcp` is always pulled at `@latest` via `npx`. Each release transitively pins a Puppeteer version that expects a specific Chrome-for-Testing milestone; nixpkgs' `playwright-driver.browsers` provides a Chrome that may run a few major versions behind. CDP is largely backward-compatible, so the core operations (`navigate`, `evaluate`, `take_snapshot`, network/console inspection, screenshots) work, but Lighthouse-driven tooling (`performance_*` insights) is the most version-sensitive piece. If a `chrome-devtools-mcp` release breaks against your nixpkgs Chrome, override `NIXPKGS_FLAKE` to a newer nixpkgs revision whose Playwright Chrome matches Puppeteer's expected milestone.

## License

Apache 2.0 — see [`LICENSE`](./LICENSE).
