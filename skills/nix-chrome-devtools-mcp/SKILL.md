---
name: nix-chrome-devtools-mcp
description: chrome-devtools-mcp launcher — a single bash script that resolves Playwright's bundled Chrome-for-Testing and Node.js directly through Nix. See repo README for usage, override mechanisms, and version compatibility.
---

# nix-chrome-devtools-mcp

Drop-in launcher for the [Chrome DevTools MCP server](https://github.com/ChromeDevTools/chrome-devtools-mcp). Self-contained — `bin/serve` resolves all inputs via `nix build` / `nix shell`.

Full docs in the [repo README](https://github.com/juspay/nix-chrome-devtools-mcp/blob/main/README.md).

This directory is the skill of the repository's Agent Plugin. It documents the launcher for agents; `mcp.json` launches `./skills/nix-chrome-devtools-mcp/bin/serve`, so the skill and the MCP server come from the same plugin.
