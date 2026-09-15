# Your coding agent forgets on purpose. A PreCompact hook is where you save it

Your coding agent doesn't have amnesia. It has a budget. When the context window fills up, the host compacts it into a summary — and summaries keep the *what* while losing the *why*. For engineering decisions, the why is everything.

This is a write-up of a tool I came across that is built around exactly that moment — the one where a decision can still be saved: **myc**, an open-source (MIT) local task-and-memory layer for coding agents, built on Bun and TypeScript. Its author describes three days of an agent re-doing the same work before writing it; the design follows from that.

## The bug report was the author

The cycle, as the README tells it, looked like this. Early in a session you'd decide something: "we're doing X, not Y, because Z." Hours later the context got compacted, the reason drowned, and the agent — honestly, politely, with no memory of ever thinking otherwise — proposed Y again.

Notice the failure mode: it's not that the agent forgot. It's that it *had no way to know there was something to remember*. Any memory layer you query after the fact is useless if nothing was written into it at the moment the context died.

So the saving moment is exactly one: **right before compaction**. Claude Code exposes that moment as the `PreCompact` hook event. That's where myc lives.

## What the hook does

`myc wire` installs the hook. When compaction is about to happen, two things run, in order:

1. **The session episode is written to disk first** — raw, into the L0 layer, marked private, with secrets masked (`password=`, `api_key=`, and friends are matched on word boundaries so prose stays prose). The write happens before anything else: if the hook exceeds its timeout, you lose the summary, not the record.
2. **A rescue packet is printed back into the context that survives** — the short list of things that must not be lost.

```
$ myc absorb-session --reason manual --transcript … --agent claude
# myc: context is being compacted — here is what must not be lost
episode sess-5jh8je4g050m saved (265 B)
NEXT     myc show sess-5jh8je4g050m · myc ready --claim
```

Distillation of what was saved is queued, never done on the write path.

## Candidates, not facts

Here's the design decision that makes myc deliberately a bit inconvenient: decisions the hook lifts from the transcript ("we decided / because" lines) are stored as *candidates* in `pending_review` state. They do not come back from search, `recall`, `prime`, or the MCP tools until a human confirms them. `prime` says so in its footer: `N pending review hidden — myc review`.

```
$ myc review
PENDING REVIEW 2 · 1 in this session's prime · 1 from other sessions · session 3f9c21aa
$ myc review confirm memory-6k2x…
```

The reasoning: a memory that hallucinates is worse than no memory, because it gets believed. Confirming a candidate turns it into knowledge exactly the way a normal note is born; rejecting it retracts the note, and retracted notes are out of every retrieval path.

## Speed is a constraint, not a metric

A memory tool lives in the agent's hot path — it gets called dozens of times per session. If it's slow, the agent starts economizing on it, and economizing on memory *is* forgetting. So every hot path in myc has a latency budget enforced by tests.

Measured on 100,000 nodes (2026-09-11, darwin-arm64, myc 0.3.6, p99):

| operation | p99 | budget |
|---|---|---|
| `prime` (session context packet) | 0.608 ms | 30 ms |
| `search` (hybrid) | 8.215 ms | 25 ms |
| `write` | 0.327 ms | 5 ms |
| cold start | 21.337 ms | 60 ms |

Reproduce it yourself:

```bash
bun install -g @aistastudio/myc
bun run scripts/bench-latency.ts   # from a source checkout
```

Honest footnote: those absolute numbers are calibrated on one machine, and CI doesn't enforce them on other hardware. CI *does* enforce structural claims (query plans, prefiltering) and relative ones (the healthy path measured against a deliberately degraded one, interleaved so hardware cancels out). Ranking is measured too: boosts take MRR@10 from 0.520 to 0.867, two-hop graph expansion from 0.193 to 0.422 — on corpora that contain a control group which gets *worse* when the feature works, so you can't fake a win by shaping the corpus.

## Memory anchored to code — and honest when the code is gone

Since 0.3.10 a note can be anchored to a span of code, and the anchor follows the code as it moves — across a refactor, even into another file (verified on TypeScript and Python). When the code is no longer where the anchor put it, the knowledge is not deleted: it ranks lower and says why. `recall` marks the row `[code moved ×0.64]`, `[code unverified ×0.5]` or `[code gone ×0.2]`; knowledge whose every anchor is lost stays out of `prime`, and the footer counts it: `N with code gone hidden`. That is the answer to the question every memory tool eventually faces — "will it start lying to me in a month?" — and it is the part I have not seen elsewhere.

## The rest of the toolbox, one line each

- **Tasks**: a graph with dependencies, blockers inherited down the parent chain, and atomic claims — two agents never get the same task.
- **Search**: hybrid (BM25 + vectors fused by RRF). Semantics are opt-in: the embedding model (129 MB) downloads only when you ask (`myc models fetch`); until then search is lexical and *says so on every answer*. No silent fallbacks.
- **Code index**: built-in tree-sitter — symbols, callers, search by meaning for TypeScript/TSX/JS/Python. Grammars are fetched once; indexing never touches the network.
- **Multi-agent sanity**: `myc run -- bun test` puts heavy commands through one machine-wide queue, so parallel agents stop strangling each other.
- **Sync**: only the oplog goes to git, merged per field — two machines editing the same note converge.
- **Migration from beads**: `myc import-beads` brought over a live project — 796 tasks, 972 dependencies, 265 notes — in 889 ms; re-running it syncs instead of duplicating. (The first release silently dropped the export's comments; found the next day on a real project, fixed in 0.2.0, and described by the author in the README.)

## The limits, stated before you hit them

- **Bun only.** The runtime binds to `bun:sqlite` — SQLite and sqlite-vec ship inside Bun with no native Node bindings. It will not start on plain Node.js or Deno. That's the deal: one runtime, one binary, zero native build steps.
- **macOS and Linux**; on Windows use WSL — the launcher doesn't start in cmd or PowerShell.
- **Single user.** No server, no team mode, no ACL. Those are designed and tracked (the team milestone has open subtasks; the live count is on the site), not implemented. Swarm routing and memory distillation haven't been started.

## Try it and break it

```bash
bun install -g @aistastudio/myc   # 3.42 MB, pulls nothing
myc init && myc wire              # hooks + MCP for Claude Code, Codex, opencode, Kimi
```

The project is about a week old and ships a release close to every day, which is another way of saying: now is the cheapest time to tell the author what's wrong. If you run agents daily, the question worth answering in the issues is whether decisions survive your sessions today — and if not, whether a compaction hook feels like the right place to save them.

Repo: https://github.com/aistastudio/myc
Site (every number printed next to the command that reproduces it): https://aistastudio.github.io/myc/
