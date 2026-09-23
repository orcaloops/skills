# orcaloops/skills

The Claude Code plugin marketplace for Orcaloops. It holds one plugin,
`orcaloops`.

## What the plugin gives you

- Three skills:
  - `/orcaloops:loop` — hand a job to a loop instead of doing it in the
    session.
  - `/orcaloops:loops` — follow up on your loops: what is running, what
    changed, and what is waiting on you.
  - `/orcaloops:loop-author` — write or change a loop in conversation while it
    takes shape on a live canvas.
- The MCP server registration: the plugin registers the `orcaloops` MCP
  server, which it runs as `orcaloops-mcp` from your `PATH`.

## Install

```
curl -fsSL https://api.orcaloops.ai/dl/orcaloops-mcp-install.sh | sh
claude plugin marketplace add orcaloops/skills
claude plugin install orcaloops@orcaloops
```

The first line installs the `orcaloops-mcp` binary. The other two install the
plugin. Keep the directory the binary was installed into on your `PATH`.

## Token

The MCP server needs an `ocl_` access token from
[https://orcaloops.ai](https://orcaloops.ai). It reads it from, in order:

1. the `ORCALOOPS_TOKEN` environment variable;
2. the file `~/.orcaloops/token` (create it with mode `0600`).

## This repository is generated

This repository is published automatically from the Orcaloops source. Every
publish overwrites it, so hand edits here are lost. Do not open pull requests
against it.
