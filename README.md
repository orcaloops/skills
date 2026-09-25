# orcaloops/skills

The Claude Code and Codex plugin marketplace for Orcaloops. It holds one
plugin, `orcaloops`, for each.

## What the plugin gives you

- Three skills:
  - `/orcaloops:loop` — hand a job to a loop instead of doing it in the
    session.
  - `/orcaloops:loops` — follow up on your loops: what is running, what
    changed, and what is waiting on you.
  - `/orcaloops:loop-author` — write or change a loop in conversation while it
    takes shape on a live canvas.
- The MCP server registration: the plugin registers the hosted `orcaloops` MCP
  server at `https://api.orcaloops.ai/mcp`. There is nothing else to install.

## Install

```
claude plugin marketplace add orcaloops/skills
claude plugin install orcaloops@orcaloops
```

That is the whole setup. The first time orcaloops is needed, Claude Code asks
you to sign in. Let it, or run `/mcp`, pick `orcaloops` and choose
**Authenticate**, then approve in the browser. Tokens refresh on their own.
**Clear authentication** in `/mcp` signs you out.

Already installed? Update to the hosted server:

```
claude plugin marketplace update orcaloops
claude plugin update orcaloops@orcaloops
```

If you once added the server by hand at user scope, remove that entry with
`claude mcp remove orcaloops -s user`, or both servers load.

## Codex

```
codex plugin marketplace add orcaloops/skills
codex plugin add orcaloops@orcaloops
```

Then approve the browser sign-in. If it does not open by itself, run
`codex mcp login orcaloops`. The same three skills and the same hosted server
come with it.

## Scripts and CI

`claude -p` cannot open a browser, but it reuses a sign-in you did once
interactively on the same machine.

For a machine where nobody can sign in, add a project `.mcp.json` entry with the
same URL and an `ocl_` access token in a header:

```json
{
  "mcpServers": {
    "orcaloops": {
      "type": "http",
      "url": "https://api.orcaloops.ai/mcp",
      "headers": { "Authorization": "Bearer ${ORCALOOPS_TOKEN}" }
    }
  }
}
```

It takes over from the plugin's server, because Claude Code treats a plugin
server with the same URL as a duplicate
([scope hierarchy and precedence](https://code.claude.com/docs/en/mcp#scope-hierarchy-and-precedence)).
Its tools are then named `mcp__orcaloops__<tool>` rather than
`mcp__plugin_orcaloops_orcaloops__<tool>`, so permission rules must use those
names.

### Minting the `ocl_` token

There is no Tokens screen yet, so minting is one RPC call you make with your
signed-in browser session.

1. Open a signed-in [orcaloops.ai](https://orcaloops.ai) tab, open the devtools
   console, and copy a Clerk session JWT:

   ```js
   await window.Clerk.session.getToken()
   ```

   It is short-lived — mint the token right after copying it.

2. `POST https://api.orcaloops.ai/rpc` with that JWT as the bearer. The endpoint
   speaks NDJSON (one JSON message per line), so the body is a single
   `Request` line naming the rpc and carrying its payload:

   ```sh
   curl -sS -X POST https://api.orcaloops.ai/rpc \
     -H 'authorization: Bearer <Clerk session JWT>' \
     -H 'content-type: application/ndjson' \
     --data-binary '{"_tag":"Request","id":"1","tag":"accessTokens.mint","payload":{"label":"laptop"},"headers":[]}
   '
   ```

   The `accessTokens.mint` payload:

   - `label` — **required.** A human label so you can recognise this token in
     the list and revoke it later.
   - `customerId` — optional, and **required in practice if you belong to more
     than one customer**: it pins the token to one of your own memberships. A pin
     can only narrow a token's reach, never widen it.
   - `expiresInDays` — optional. Absent means the token does not expire, and
     revocation is the control.

   The reply is one `Exit` line; the token is `exit.value.token`:

   ```json
   {"_tag":"Exit","requestId":"1","exit":{"_tag":"Success","value":{"id":"01…","label":"laptop","token":"ocl_…","expiresAt":null}}}
   ```

3. **The plaintext is shown once.** Only its SHA-256 digest is stored, so no read
   path can ever hand it back — if you lose it, mint another and revoke the old
   one (`accessTokens.revoke`).

Make this call with the Clerk session, never with an `ocl_` token: a token can
never mint another token.

## This repository is generated

This repository is published automatically from the Orcaloops source. Every
publish overwrites it, so hand edits here are lost. Do not open pull requests
against it.
