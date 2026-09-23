---
name: loop
description: Hand a job to an orcaloops loop instead of doing it in this session. Use when the person says "hand this to a loop", "run this in the background", "kick off a long job", "do this on every PR", or asks for work that is wide, long-running, budgeted or recurring.
allowed-tools: mcp__plugin_orcaloops_orcaloops__loops_search, mcp__plugin_orcaloops_orcaloops__loops_list, mcp__plugin_orcaloops_orcaloops__loops_get, mcp__plugin_orcaloops_orcaloops__loops_invoke, mcp__plugin_orcaloops_orcaloops__loops_draft, ListMcpResourcesTool, ReadMcpResourceTool
---

# Hand work to a loop

## Two nouns

A **loop** is a reusable *document*: the steps it runs — agents, people and deterministic actions — the evaluators that judge their outcomes and pick the next path, the `inputs:` it takes and the one outcome it produces. An **invocation** is *one walk* of that document, with its own inputs, steps and budget. `loopId` and `invocationId` are **different ids** — `loops_*` tools take the first, `invocations_*` tools take the second, and mixing them calls the wrong tool every time.

A loop is **living**. The person changes it as the work teaches them what the job actually is, and every change is published as a new **immutable version** that moves the head; each invocation records the version it walked, so changing a loop never touches an invocation that already ran. There is no revert — going back means publishing the earlier document again as the next version: **rollback is a forward update**.

## When to hand work over instead of doing it yourself

Hand it over when the job is wide (many files, many parallel attempts), long (it outlives this session), budgeted, or recurring on a merge or a clock. Not a single edit, not a question, not anything you can finish in your own turn.

## Search, then run, change or author — in that order

**Always call `loops_search { kind: "definition" }` before authoring anything.** Then:

*An existing loop already covers the ask* → run it: `loops_invoke { loopId, inputs }`. Done.

*An existing loop nearly covers it* → **change it, with the person watching**: follow the loop-author skill, starting from `loops_draft { action: "open", slug, fromLoopId }`, publish when they say so, then run it. **Changing a near-match is always better than authoring a near-duplicate.**

*Nothing like it exists* and the ask names a class of job ("review every PR for security"), or the person says "every time" / "whenever" / "each PR" → **author it** the same way, from `loops_draft { action: "open", slug }`. Publishing it makes it real — that is **version 1** — then run it for the instance the person actually asked about. Expect to change it after the first run or two; that is what versions are for.

*It is genuinely one-off* → **invoke ad hoc** with `loops_invoke { prompt }` and author nothing. **When it is close, invoke ad hoc**: an unwanted loop is clutter the person has to archive, and authoring one later is one conversation away.

## Running a loop starts a walk

`loops_invoke { loopId, inputs }` starts a **walk** of the loop's current document. `inputs` are the names the document's `inputs:` declares, and a value is whatever JSON that input's type takes — a string, a number, an object for an issue. A refusal names the input that was missing or wrong: fix that one and invoke again, never guess around it. A loop that has no document yet (an older template loop) still runs the way it always did.

For `loops_invoke { prompt }`, the `prompt` is what the person said, **not your summary of it** — it becomes the statement the run is steered by, so summarising it is the single most damaging thing you can do here. The draft phase **creates nothing**. Show `understood` and `unanswered`. Never fill an `unanswered` field on their behalf. Never call the confirm phase in the same turn as the draft without the person answering.

Then **keep working** — the walk runs elsewhere, and blocking on it is the failure this whole design exists to prevent.

## Money, and the link

**Authoring, drafting and publishing are free; invoking starts work and spends money.** A loop has no status of its own — only its invocations do.

Every tool result carries a `url` (or `draftUrl` / `liveUrl`). Right after each `loops_invoke` call, hand it to the person unprompted, on its own line, so they watch the walk move in real time without asking:

Watch it here: <url>

When a result carries a `canvasUrl`, the loop holds a **document** — the graph of steps, evaluators and routes the engine walks — and a graph the person cannot see is a graph they cannot approve, so hand the canvas over the same way, unprompted, on its own line:

Show them the canvas: <canvasUrl>
