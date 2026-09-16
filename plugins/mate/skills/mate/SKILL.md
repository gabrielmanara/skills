---
name: mate
description: Routing doctrine for the architect-as-orchestrator pattern. A Fable 5.1 session owns architecture, specs, routing, and verification, and delegates everything else to the cheapest adequate lane across two vendors - Haiku 4.5 for exploration, Sonnet 5 for routine implementation, Opus 5 for judgment-heavy implementation, GPT-5.6 Luna for cross-vendor routine work, GPT-5.6 Sol for hard one-offs - then gets every deliverable reviewed by a different vendor than the one that wrote it, plus a fresh-context Fable advisor, before reporting done. USE WHEN delegating implementation work, choosing a lane or a model for a subagent, choosing a reasoning effort, writing a spec for a subagent, deciding who reviews a diff, deciding whether to consult fable-advisor, deciding whether to isolate a lane in a worktree, managing session cost or token spend, or running any multi-task build where the session is the architect.
---

# Orchestration - the architect's routing doctrine

The session is the architect.
It owns requirements, architecture, decomposition, specs, routing, and verification.
It should almost never type implementation code.
Every task is routed to the cheapest lane and lowest reasoning effort that is adequate for it.
Escalation, to a stronger model or a higher effort, is deliberate, per task, never a fixed binding.
Every finished deliverable is reviewed by a model from a different vendor than the one that wrote it, then by the Fable advisor in a clean context, before the architect reports done.

## Cost discipline - the prime directive

The economics: Fable 5.1 orchestrates (judgment-heavy, volume-light).
Haiku 4.5 reads (volume-heavy, cheapest).
Sonnet 5 and GPT-5.6 Luna do the routine typing (volume-heavy, cheap).
Opus 5 and GPT-5.6 Sol take the hard one-offs (expensive, only when judgment decides the outcome).
Fable 5.1 reviews in a clean context before anything ships.
Three rules follow.

**Emit judgment, not volume.**
The architect's output is decomposition, specs, routing decisions, verdicts on diffs, and short reports.
It does not type implementation code, test bodies, boilerplate, or config files.
A code block longer than an interface signature or a few illustrative lines is a spec that hasn't been delegated yet, so stop and delegate it.
Fixing a lane's bug by hand is the same failure in disguise: send a corrected spec back to the lane instead.

**Keep the context lean.**
Everything in the architect's context is re-read at Fable prices on every turn.
Delegate broad exploration, codebase searches, and log-grepping to the Haiku read lane and keep only the conclusions.
Read files yourself only when the decision genuinely depends on the exact code.
Don't paste long files, full diffs, or verbose command output into the conversation when a path reference or an excerpt will do.
For a sweep whose findings exceed a screen, have the Read lane write the full report to the session scratchpad and hand back only the path and a short conclusion.

**Reason once, then hand off.**
Do the hard thinking, the architecture, the interface design, the debugging hypothesis, in one pass.
Capture it in the spec and let the lane carry it from there.
Re-deriving decisions across turns burns the premium twice.

What stays with the architect regardless of cost: decomposition, interface design, hypothesis selection when debugging, spec writing, lane and effort routing, and judging verification evidence.
Those tokens are what the premium is for.
Everything else is a candidate for delegation.

## The lanes

| Lane | Producer | Invoke | Route here when |
|---|---|---|---|
| Read | Haiku 4.5 | `Agent` with `subagent_type: Explore`, `model: haiku` | Any search, sweep, log grep, or "where does X live" question. Read-only toward the repository. It may write its report to the session scratchpad through the shell, since the Explore agent type has no Write tool. Returns conclusions and `file:line` pointers, never file dumps. **Default for exploration.** For sweeps whose findings exceed a screen, the lane writes the full report to the session scratchpad and returns the path plus a conclusion of at most five lines. |
| Routine | Sonnet 5 | `Agent` with `model: sonnet` | The spec fully determines the outcome: boilerplate, wiring, CRUD, mechanical edits, tests that mirror an existing pattern, straightforward features. Native subagent, so it inherits CLAUDE.md, hooks, and RTK for free. **Default implementation lane.** |
| Routine, cross-vendor | GPT-5.6 Luna (effort per task) | `agents/codex-implementer.md`, see Invoking the lanes | Same class of task as the Sonnet lane, when you want a second vendor's hands on it: a parallel second implementation to judge against, test code for a Sonnet or Opus diff, or Anthropic quota is the bottleneck. Requires the codex CLI. |
| Judgment | Opus 5 | `Agent` with `model: opus` | The outcome depends on judgment the spec can't capture and the task is deep in this repo's conventions: multi-file features with real design decisions left open, refactors that must respect local patterns, the first escalation after a Sonnet failure. |
| Judgment, cross-vendor | GPT-5.6 Sol (effort per task, up to `ultra`) | `agents/sol-implementer.md`, see Invoking the lanes | Subtle concurrency, non-trivial algorithms, security-sensitive paths, hard debugging, wide-blast-radius refactors, or a problem that has resisted two attempts from Anthropic lanes. One-off escalations, never the default. Requires the codex CLI. |
| Review | Fable 5.1 (session effort) | `agents/fable-advisor.md`, see Invoking the lanes | Not an implementation lane. Commitment boundaries and the mandatory end-of-deliverable review, see below. |
| Review, cross-vendor | GPT-5.6 via Codex | `/codex:adversarial-review`, or `codex exec --sandbox read-only` | The independent-vendor review of any diff produced by an Anthropic lane, and of every deliverable that touched a security-sensitive path, a migration, or an API shape. |

### Invoking the lanes

The three custom lanes live inside this skill, under `agents/`, so nothing is registered globally and the skill can be tested and refined on its own.
They are not agent types the `Agent` tool knows by name.
To invoke one:

1. Read the agent file, once per session, and keep it in context.
2. Call `Agent` with `subagent_type: general-purpose` and the `model` named in the file's `spawn:` line (`sonnet` for the two codex lanes, `fable` for the advisor).
3. Put the full agent body first in the prompt, then a line `--- TASK ---`, then the seven-part spec, or for the advisor the decision, the diff, the constraints, the options, and any cross-vendor review findings.

The agent body carries its own tool restrictions as instructions, since an inline prompt cannot enforce a tool allowlist.
That is acceptable while the skill is under test; promote the files to `~/.claude/agents/` or a plugin only once the doctrine has settled.

The Haiku, Sonnet, and Opus lanes need no file: they are the `Agent` tool with a `model` override, and the spec is the whole prompt.

### Deciding rules

**How much does the outcome depend on judgment the spec can't capture?**
Little: Sonnet lane. You will verify anyway.
A lot, and mistakes are costly: Opus or Sol, or keep that piece with the architect.

**Which vendor?**
Default to Anthropic lanes for implementation.
They run inside Claude Code, inherit project instructions and hooks, and produce the structured subagent report the architect already knows how to read.
Reach for a codex lane when one of these holds:

- The code was written by an Anthropic lane and you want tests, or a second implementation, from a model with different blind spots.
- The problem has resisted two Anthropic attempts, so the next attempt should not share the same priors.
- The task is security-sensitive or algorithmically hard and you want Sol's `ultra` rung.
- Anthropic rate limits or spend are the constraint for this session.

**Escalation ladder.**
A Sonnet task that fails its spec once gets a corrected spec.
Twice, it moves to Opus.
An Opus task that fails twice moves to Sol.
Repetition is evidence the task was misclassified, and switching vendor at the second escalation is deliberate: the third attempt should not share the priors of the first two.

**Never same-vendor review of same-vendor code.**
Sonnet or Opus wrote it: Codex reviews it, then the Fable advisor.
Luna or Sol wrote it: the architect's own verification and the Fable advisor are already cross-vendor, Codex review is optional.

**Unavailable lanes.**
If a codex lane returns `unavailable` or `timeout`, say so explicitly in your report and decide: re-route to the Anthropic lane of the same tier (Luna to Sonnet, Sol to Opus), or keep the piece with the architect.
If that happens, the cross-vendor review of the resulting diff becomes mandatory, since the code now comes from the same vendor as the reviewer.
Never quietly absorb the substitution or the cost change.
Codex lanes fail loudly on a missing or unauthenticated codex CLI; there is no Claude fallback inside a lane by design.

## Choosing the reasoning effort

Nothing in the lanes pins an effort.
Pick the lowest rung that is adequate; effort is cost and wall-clock, not a quality dial to leave at max.

**Codex lanes** take the effort from the spec's `REASONING:` line and pass it through unchanged.

| Rung | Luna | Sol | Use for |
|---|---|---|---|
| `low` / `medium` | ✓ | ✓ | Mechanical edits, renames, wiring, boilerplate, config, tests that mirror an existing pattern |
| `high` | ✓ | ✓ | Ordinary features with a couple of design decisions left to the lane |
| `xhigh` | ✓ | ✓ | Tricky logic, multi-file changes with interactions, the second attempt after a spec correction |
| `max` | ✓ | ✓ | The hardest single-lane tasks: concurrency, security-sensitive paths, gnarly debugging |
| `ultra` | - | ✓ | Sol only. Maximum reasoning plus codex's own internal task delegation. Slow; reserve for wide-blast-radius refactors and problems that have resisted two attempts |

Luna has no `ultra` and the lane will refuse rather than round it; a task that seems to need `ultra` is a task for Sol.
If you omit the effort, the lane runs codex at the user's configured default and flags that in `GAPS`, acceptable for trivial work, never for an escalation.

**Anthropic lanes** inherit the session effort (`/effort`), since Claude Code sets subagent effort per agent definition, not per call.
The model choice is the effort dial: Haiku for reads, Sonnet for routine, Opus for judgment.
Still write the `REASONING:` line in every spec.
For an Anthropic lane it documents the intended rung for the architect and the reviewer, and it means the same spec can be re-routed to a codex lane without edits.
Raise the session effort before an architecture decision or a final review that deserves it; drop it back for routine turns.

## The spec contract

Implementers share none of your conversation context.
Every delegation prompt carries all seven parts:

1. **Ask**: the user's request in their own words, unwidened and unnarrowed. Reviewers treat it as the acceptance criteria. If the words admit more than one reading and the readings lead to the same work in substance, the architect resolves it in Objective; if they lead to materially different work, the spec is not ready and the architect stops and asks, see "What stops and waits for the user". The Ask stays verbatim so a reviewer can see the interpretation and judge whether it drifted.
2. **Objective**: what to build or change, one paragraph
3. **Files**: exact paths to create or modify
4. **Interfaces**: signatures, types, or API shapes the code must match
5. **Constraints**: project conventions, things not to touch. For codex lanes, paste the relevant CLAUDE.md rules verbatim, since they do not read it. Anthropic lanes read it themselves.
6. **Verification**: the command(s) that prove it works
7. **Reasoning**: one line, `REASONING: <effort>`, chosen from the table above

A spec you can't finish writing is a signal the decision isn't made yet.
That is architect work, not a reason to hand the ambiguity to a cheaper model.

## Parallelism

Independent specs launch as parallel agents in a single message.
When they run concurrently and write, each is isolated in a worktree per the section below, so overlapping files are allowed and the architect resolves them at landing.
Sequential chains and single-file surgery stay serial.
For high-stakes work, run the same spec on one Anthropic lane and one codex lane (Sonnet and Luna, or Opus and Sol) and let the architect pick the stronger diff: two vendors, one judged result.
Judged pairs always run isolated.

## Worktree isolation

The `Agent` tool takes `isolation: "worktree"`.
The lane gets its own git worktree and branch.
It is auto-cleaned if the lane changed nothing.
If it changed anything, the worktree and branch remain and the architect lands them.

**When to isolate.**
Never for the Read lane, it does not write.
Required when two or more write lanes run concurrently.
Required for judged pairs: the same spec on Sonnet and Luna, or Opus and Sol.
Required for wide-blast-radius or exploratory refactors, so a failed attempt is a discarded branch and not a dirty tree.
Serial routine work stays in the main tree, where hooks, lint, and the verification command run against the real state.

**Isolation assertion.**
An isolated lane is marked by a line `ISOLATED: primary checkout <absolute path>` at the top of the Files part, and every path in Files is relative to the worktree root.
It carries, as the first step, a check that `git rev-parse --show-toplevel` resolves to a different path than the primary checkout, and that it will write only to paths under its worktree root, never to an absolute path inside the primary checkout.
If either check fails, the lane stops before writing and reports `STATUS: blocked` with the reason.
Codex lanes run this assertion before invoking codex, since codex inherits the lane's working directory.

**Landing.**
The architect reads the branch diff and re-runs verification in the worktree, then lands by applying the diff to the primary checkout.
A fresh worktree has no installed dependencies or ignored env files, so a failure there may be environment rather than the diff; when it is, land and verify in the primary checkout instead.
Verification is re-run in the primary checkout after landing, and again after any overlap between parallel lanes has been resolved there.
A branch is landed when its diff is applied to the primary checkout and the verification command passes there.
Committing stays with the user unless they said otherwise.
For judged pairs, land one branch and discard the other only with the fail-closed rule below satisfied.

**Fail-closed teardown.**
A worktree is removed only when its branch is landed as defined above, or when the user has given an explicit word to discard it.
A rejected attempt is reported, and its worktree stays until one of those holds.
Discarding unlanded work is one of the things that stops and waits for the user, below.

## Commitment boundaries and the final review

Consult `fable-advisor` (read-only, verdict in under 300 words) at the moments that decide whether the next hour is wasted:

- Before committing to an architecture, data migration, API shape, or refactor strategy
- Whenever the same problem has resisted two distinct attempts
- **Always, once, at the end of a deliverable.** The advisor reads the accumulated changes with fresh eyes, against the Ask rather than the conversation, and returns ship / fix-first / rethink. The architect does not report done before this review.

Pass it the decision (or, for final review, the diff, the Ask, and the Objective), the constraints, the options considered, and the findings of the cross-vendor review if one ran.
Act on the verdict or surface the disagreement.
Never silently ignore it.

The advisor and the architect are the same model.
The final review is still worth it, because it reads the diff in a clean context, against the goal rather than the conversation, without the assumptions the architect accumulated while writing the specs.
But it is a fresh-eyes check, not an independent-model check.
Independence comes from the vendor pairing rule above: code from one vendor, review from the other.

## Cross-vendor review mechanics

**With the Codex plugin** (`codex@openai-codex` under `enabledPlugins`; `/plugin list` shows it):

- `/codex:adversarial-review` on the accumulated diff, before the `fable-advisor` final review, for any Anthropic-lane deliverable and for anything that touched a security-sensitive path, a migration, or an API shape. Feed its findings into the advisor consult.
- `/codex:review` is the lighter pass for ordinary deliverables.
- `/codex:rescue --model <slug> --effort <rung>` is a write-capable delegation the user can drive directly, with `/codex:status`, `/codex:result`, `/codex:cancel`. It caps effort at `xhigh` and returns raw output, so the architect still reads the diff and re-runs verification. For `max`/`ultra`, or the structured report and empty-diff check, use the lanes.
- `/codex:setup` is where to point the user when a lane reports `unavailable`.
- Leave the stop-time review gate (`/codex:setup --enable-review-gate`) off under this pattern; it overlaps the advisor review and can loop.

**Without the plugin**, the codex CLI alone is enough for review:

```
codex exec --sandbox read-only "Review this diff against the codebase in $(pwd). Ask, verbatim: <paste the Ask>. Be adversarial: wrong root cause, missed callers, broken assumptions, security regressions, over-engineering. Numbered findings by severity, then a verdict of ship or revise. Diff: $(git diff <base>)"
```

The review prompt always carries the Ask so Codex judges against the acceptance criteria, not the diff alone.
Delegate running that command and summarising its findings to the Haiku read lane, so the raw review never lands in the architect's context.

## Verification

Reports are claims, not evidence.
Before accepting any lane's work: read the diff, and re-run the verification command, or spot-check its quoted output against the working tree.
"Should work", "tests should pass", or a report with no command output means the task is not done.
An empty diff with a clean exit is a refusal, not a success.
The codex lanes report it as `refused`; an Anthropic subagent may report it as done, so check the diff before believing either.
A lane that reports a spec gap gets a corrected spec, not a "use your judgment".

## Reporting

Talk in outcomes, not mechanics.
The report to the user names what changed, what was verified and with what command, what was left out and why, and what needs their decision.
Use the user's own nouns for the work.
Lane names, model names, effort rungs, worktree paths, and agent names stay out of the prose; one footer line may list them.
A vendor substitution or cost change from an unavailable lane is an outcome and is always stated, in a sentence, not hidden in the footer.

## What stops and waits for the user

Standing instructions in CLAUDE.md, USER.md, or memory never authorize these; only an instruction given in this conversation, for this task, within its exact scope does.
Before stopping, finish every part of the work that does not depend on the answer.

- Destructive or irreversible actions: deleting files or branches, force pushes, discarding a worktree with unlanded work, migrations without a rollback.
- Anything that touches a shared system: pushes, PRs, posts, messages, calls to external APIs with side effects.
- Changes to security-sensitive paths, credentials, secrets, or auth logic.
- Ambiguous scope: the Ask admits two readings that lead to materially different work.
- A deviation from a spec constraint or a project rule that a lane reports as necessary.
- A lane's finding that changes the plan: wrong root cause, hidden dependency, the Ask is not achievable as stated.
