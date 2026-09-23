---
name: loop-author
description: Write or change an orcaloops loop in conversation while the person watches it take shape on a live canvas. Use when the person says "make a loop that…", "write a loop for…", "change the review step on loop X", "add a step to that loop", or otherwise asks to author or edit what a loop does.
allowed-tools: mcp__plugin_orcaloops_orcaloops__loops_draft, mcp__plugin_orcaloops_orcaloops__loops_search, mcp__plugin_orcaloops_orcaloops__loops_list, mcp__plugin_orcaloops_orcaloops__loops_get, Bash(open https://orcaloops.ai:*), Bash(xdg-open https://orcaloops.ai:*), ListMcpResourcesTool, ReadMcpResourceTool
---

# Author a loop with the person watching

## What you are writing

A loop is a **document**: the steps it runs — agents, people and deterministic actions — the evaluators that judge their outcomes and pick the next path, the `inputs:` it takes and the one outcome it produces. The engine walks that document every time the loop is invoked. You write the document; the person watches it on a canvas and decides when it is right.

Take the grammar from its canonical sources, never from memory: the model document `PLAN-LOOP-MODEL-V3.md` and the worked example `orca-issue-v4.json` (with the `@fm/loops` test fixtures), both in the orcaloops source repository — find them by name. Copy the example's shape, keep a minimal node to one line, and add a key only when the job needs it. Where that repository is not at hand, let the findings teach you: write the smallest document that says what the person asked for, and each write names what is still missing.

## 1. Open a draft and put the canvas in front of the person

- **A new loop:** `loops_draft { action: "open", slug, name, description }`, with a short kebab-case `slug` taken from what the person asked for. A slug that already names a loop opens a draft of *that* loop instead, so search first and pick an unused one.
- **Changing a loop:** find it (`loops_search { kind: "definition" }`, then `loops_get`) and call `loops_draft { action: "open", slug, fromLoopId }` with that loop's own `slug` (a different slug is refused) — the draft starts from its current document, name and description.

`open` is idempotent per slug: a draft already in progress comes back as it stands. Before your first write, `loops_draft { action: "get", draftId }` and start from the `body` it returns, never from what you remember.

Keep the `draftId` and the `canvasUrl` for the whole conversation, and open the canvas for the person straight away, unprompted. In Claude Code, run `open <canvasUrl>` (macOS) or `xdg-open <canvasUrl>` (Linux) with the Bash tool. In any other client, print it on its own line:

Watch it take shape: <canvasUrl>

Every write lands on that canvas the moment it is made — the graph, the findings, the paths — so the person reviews the real document, not your description of it.

## 2. Write whole documents, and read the findings after every write

`loops_draft { action: "write", draftId, body, expectedRevision }` replaces the draft's document with `body`. Always send the **whole** document — never a fragment or a patch — and pass the `revision` the last result gave you as `expectedRevision`.

Every write returns `findings`. An `error` blocks publishing and must be fixed; a `warning` is worth telling the person about. A finding names its `rule` and, when it has one, the `node` — fix that node and write again. Never publish around an error, and never send the same document twice hoping for a different answer.

A result with `conflict: true` means someone else wrote the draft after you read it: `get` it, apply your change to that document, and write again with the revision the conflict reported. A write refused because the draft is closed is not a conflict — do not retry it; `open` again.

## 3. Summarise, then let the person steer

When a change settles, tell the person in a few lines what the loop now does: its inputs, its steps in order, where it loops back or stops, and the outcome it produces. Point at the canvas rather than drawing the graph in text. Ask what to change, and keep going until they are happy.

## 4. Publish only when the person says so

`loops_draft { action: "publish", draftId, changeNote }` turns the draft into **one new immutable version** — version 1 of a new loop, or the next version of an existing one — and returns the loop's `canvasUrl`. Invocations that already ran keep the version they ran. Publish **only on the person's explicit say-so** ("publish it", "ship it", "save it"); liking what they see is not a yes. Pass a one-line `changeNote` saying why this version exists. Publishing closes the draft; the next change starts from a fresh `open`.

Drafting and publishing are free. Running the loop is a separate decision: `loops_invoke` starts work and spends money. If the person abandons the draft, `loops_draft { action: "discard", draftId }` throws it away.
