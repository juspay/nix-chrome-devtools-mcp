# nix-chrome-devtools-mcp

A drop-in launcher for the [Chrome DevTools MCP server](https://github.com/ChromeDevTools/chrome-devtools-mcp), packaged for [APM](https://microsoft.github.io/apm/).

`bin/serve` resolves Playwright's bundled Chrome-for-Testing and Node.js directly through Nix. One bash file, one [`apm.yml`](https://microsoft.github.io/apm/reference/manifest-schema/) entry, no devshell required.

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

`bin/serve` pins `chrome-devtools-mcp` to an explicit version. Each `chrome-devtools-mcp` release transitively pins a Puppeteer release, which expects a specific Chrome-for-Testing milestone. nixpkgs' `playwright-driver.browsers` provides a Chrome that may be a few major versions older than Puppeteer's expectation; CDP is largely backward-compatible, so the core operations (`navigate`, `evaluate`, `take_snapshot`, network/console inspection, screenshots) work, but Lighthouse-driven tooling (`performance_*` insights) is the most version-sensitive piece. If a `chrome-devtools-mcp` upgrade breaks against your nixpkgs Chrome, bump both ends together — pick a Playwright-driver revision whose Chrome milestone matches Puppeteer's expected version.

## Upgrading

1. `curl -s https://registry.npmjs.org/chrome-devtools-mcp/latest | jq -r .version` to see the current release.
2. Update the pinned version in `bin/serve`.
3. Test against your project's Chrome.

## License

Apache 2.0 — see [`LICENSE`](./LICENSE).
