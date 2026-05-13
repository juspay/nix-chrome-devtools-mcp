# nix-chrome-devtools-mcp

One-click setup for the [Chrome DevTools MCP server](https://github.com/ChromeDevTools/chrome-devtools-mcp) in any Nix-based project. Delivered as an [APM](https://microsoft.github.io/apm/) package: add a single line to your project's [`apm.yml`](https://microsoft.github.io/apm/reference/manifest-schema/) and `apm install` does the rest — Chrome (Playwright's bundled Chrome-for-Testing) and Node.js resolve through Nix; the MCP server gets wired into every supported harness's runtime config (Claude Code, Codex, OpenCode, …). No devshell to author, no flake to compose. The `nixpkgs` source is overridable per process or system-wide.

## Usage

Add to your project's [`apm.yml`](https://microsoft.github.io/apm/reference/manifest-schema/):

```yaml
dependencies:
  apm:
    - juspay/nix-chrome-devtools-mcp
```

`apm install` aggregates the MCP entry from this package's `apm.yml` into your runtime configs (`.mcp.json`, `.codex/config.toml`, `opencode.json`).

## Requirements

Nix — see [nixos.asia/en/install](https://nixos.asia/en/install). Flakes must be enabled (`experimental-features = nix-command flakes`).

## Overriding the nixpkgs source

`bin/serve` reads `$NIXPKGS_FLAKE` (default `nixpkgs`). Override per-consumer:

- **Per process**: `NIXPKGS_FLAKE=github:NixOS/nixpkgs/<rev>` in the consumer's environment.
- **System-wide**: `nix registry pin nixpkgs github:NixOS/nixpkgs/<rev>`.
- **Project-local pin**: declare an overriding `mcp:` entry for `chrome-devtools` in the consumer's root `apm.yml` (root deps win APM's name-first dedupe).

## Chrome version compatibility

`chrome-devtools-mcp` is always pulled at `@latest` via `npx`. Each release transitively pins a Puppeteer version that expects a specific Chrome-for-Testing milestone; nixpkgs' `playwright-driver.browsers` provides a Chrome that may run a few major versions behind. CDP is largely backward-compatible, so the core operations (`navigate`, `evaluate`, `take_snapshot`, network/console inspection, screenshots) work, but Lighthouse-driven tooling (`performance_*` insights) is the most version-sensitive piece. If a `chrome-devtools-mcp` release breaks against your nixpkgs Chrome, override `NIXPKGS_FLAKE` to a newer nixpkgs revision whose Playwright Chrome matches Puppeteer's expected milestone.

## License

Apache 2.0 — see [`LICENSE`](./LICENSE).
