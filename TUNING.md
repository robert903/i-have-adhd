# Tuning log — i-have-adhd (robert903 fork)

Running doc. Baseline is Robert's global `~/.claude/CLAUDE.md`. Each decision gets
logged here so the SKILL.md rewrite is traceable.

Upstream: `ayghri/i-have-adhd` (remote `upstream`)
Fork: `robert903/i-have-adhd` (remote `origin`)
Started: 2026-09-09

---

## Part 1 — Baseline: existing global CLAUDE.md rules

Verbatim from `~/.claude/CLAUDE.md`, section "Writing: plain first, short by default".
These are already in force in every session. The tuned skill must not contradict them.

| # | Rule | Substance |
|---|------|-----------|
| G1 | Plain first | Write the plain-English version first. Never write the technical version and translate on pushback. |
| G2 | Name the thing, then the mechanism | "The playbook is wrong about routing" > "the playbook's §7.1 central routing claim does not survive checking." Cut any spec number, file path, or API name not doing work in the sentence. |
| G3 | Translate invented terms | Framework vocabulary is not shared vocabulary. Give the one-line plain meaning at first use, then use the term freely. If the tool's name is bad, say so and move on. |
| G4 | Never explain twice | If a section restates an earlier one in different words, delete one. No plain-English recap under a technical section. |
| G5 | Default to short | A few sentences answers most questions. Structure only when the content is genuinely a list/table/sequence. No intro line, no restatement of the question, no closing recap. |
| G6 | No self-narration | Skip "let me...", "I'll now...", "worth noting that...". Say the thing. |
| G7 | Trust me to ask | The answer plus the one caveat that changes what I'd do. Leave out the other three. Genuine uncertainty gets one sentence, not a hedging paragraph. |
| G8 | Applies everywhere | Chat, commit messages, plans, code comments alike. Length is a cost, not a signal of effort. |

Non-writing sections of the global file (AdLoop ruleset location, `install-rules` ban) are
out of scope for this plugin.

---

## Part 2 — Overlap map: upstream skill vs. global rules

Upstream `skills/i-have-adhd/SKILL.md` has 10 rules + 6 exceptions + a pre-send check.

**Already covered by global rules — candidates to delete as duplication (violates G4):**

| Upstream rule | Duplicates |
|---|---|
| R1 Lead with the next action | G1, G5 (partial — the "action" framing is new, see open questions) |
| R4 Suppress tangents | G7 |
| R8 Matter-of-fact tone for errors | G6 (partial) |
| R10 No preamble, no recap, no closers | G5, G6 |
| Pre-send check items 1, 2, 3 | G5, G6, G7 |

**Genuinely new — candidates to keep:**

| Upstream rule | What it adds |
|---|---|
| R2 Number multi-step tasks | Fewest steps that work; no step contains "and then" twice |
| R3 End with one concrete next action | Under two minutes, even "open the file" counts |
| R5 Restate state every turn | "Step 3 of 5 done: X. Next: Y." — global rules say nothing about cross-turn state |
| R6 Specific time estimates | Concrete units, not "some work" |
| R7 Make completed work visible | Show what now works, concretely |
| R9 Cap lists at 5 items | Split into do-now vs later past five |
| Pre-send check 5 | No idioms — replace figurative phrases with the literal action |

**Conflict to resolve:** R1 says the first line must be an *action*. G1/G5 say plain and
short but not necessarily imperative. Many of Robert's sessions are analysis, not task
execution — see open questions.

---

## Part 3 — Decisions

_(appended as we go)_

| # | Question | Decision | Date |
|---|----------|----------|------|
| 1 | Layer on global CLAUDE.md, or standalone? | **Standalone.** Skill holds the complete ruleset — the 8 global rules restated in Robert's voice plus the 7 new ones. The writing section of `~/.claude/CLAUDE.md` gets deleted so nothing is stated twice (G4). Single source of truth, portable across machines and harnesses. | 2026-09-09 |
| 2 | Activation mechanism? | **Output style** at `~/.claude/output-styles/i-have-adhd.md`, set once with `/output-style`. Applies to every session in every folder, no hook, no flag file, survives `/clear` and `/compact`. Claude Code only — the repo's other harness adapters become dead weight. A non-default output style also strips Claude Code's built-in code-generation instructions — handled by decision 3. | 2026-09-09 |
| 3 | Stripped engineering instructions? | **Carry them over.** The output style file gets a preserved "engineering behavior" block — existing file conventions, no unsolicited comments, verify before claiming done, never commit unless asked, refuse malicious code — above the writing rules. Nothing lost, still one file. | 2026-09-09 |
| 4 | R1 "lead with the action" vs. analysis requests | **Generalize to the verdict.** First line is whatever answers the question: a task gets the command, an analysis gets the finding, a diagnosis gets the cause. No fake imperatives, no fixed two-line opener. Resolves the R1/G1 conflict. | 2026-09-09 |
| 5 | Time estimates in an agent harness | **Reframe to Robert's cost, not wall-clock.** Estimate approvals needed, one-turn vs. long-run, and whether he has to watch — "one turn, no approvals" / "long run, you're not needed until the review." Real clock estimates survive only when *he* is the one executing. | 2026-09-09 |
| 6 | Restating progress vs. the todo checklist | **Checklist first, prose only when there isn't one.** When a todo list is on screen it does the restating and prose says only what's new. Conversations, research threads and debug sessions have no checklist — those open with one line of where we are. | 2026-09-09 |
| 7 | Robert-specific bounce triggers → new rules | **All four adopted.** (a) Options with no pick — always lead with the recommendation, name a coin flip as a coin flip. (b) Confidence with no evidence — no completion claim without the command and its output; say plainly when something is untested. (c) Walls of code with no anchor — every code block gets a one-line "what changed, where to look" above it. (d) Re-explaining known ground — assume expert baseline on GA4, GTM, ClickUp, git, SEO; explain only genuinely new or tool-invented terms (extends G3). | 2026-09-09 |
| 8 | Canonical file location | **Repo is source, symlinked in.** Real file at `output-styles/i-have-adhd.md` in this repo; `~/.claude/output-styles/i-have-adhd.md` is a symlink to it. Edits go live immediately and every tuning change gets git history. | 2026-09-09 |
| 9 | Other harness adapters in the fork | **Leave untouched.** Cursor, Codex, Gemini, opencode, Kimi, Qwen and Pi adapters stay. They're inert, cost nothing, and keep `git merge upstream/main` conflict-free. All Robert's changes live in new files only. | 2026-09-09 |

---

## Part 4 — Open questions

_All resolved. Nine decisions above._

---

## Part 5 — Build log (2026-09-09, approved)

Done:
- `output-styles/i-have-adhd.md` — 143 lines, 19 rules in 4 groups (Shape, Substance, Claims,
  Multi-step) plus the preserved engineering block, 5 break-glass exceptions, and the pre-send
  delete list. Every global rule G1–G8 carried over; every decision 4–7 baked in.
- Symlinked to `~/.claude/output-styles/i-have-adhd.md`. Verified: resolves, frontmatter parses.
- `~/.claude/CLAUDE.md` — writing section replaced with a 4-line pointer. AdLoop section untouched.
  Backup at `~/.claude/CLAUDE.md.bak-20260909`.

- `~/.claude/settings.json` — `"outputStyle": "i-have-adhd"` inserted as a top-level key, so the
  style applies in every folder rather than only the project where `/output-style` was run.
  Inserted as a text edit, not a JSON rewrite: all 228 permission entries and every other key
  are byte-identical. Backup at `~/.claude/settings.json.bak-20260909`.

Takes effect in new sessions — this one keeps its current style until restart.

### Rule provenance

| Group | Rules | Source |
|---|---|---|
| Shape | 1 first line answers, 2 no self-narration, 3 default short, 4 never twice, 5 anchor code, 6 cap at five, 7 no preamble/recap/closers | G1+G5+decision 4; G6; G5; G4; decision 7c; upstream R9; upstream R10 |
| Substance | 8 name then mechanism, 9 expert baseline, 10 trust him to ask, 11 lead with the pick, 12 finish before raising next | G2; G3+decision 7d; G7; decision 7a; upstream R4 |
| Claims | 13 no claim without evidence, 14 errors get cause and fix, 15 concrete finished work | decision 7b; upstream R8; upstream R7 |
| Multi-step | 16 number the steps, 17 checklist does restating, 18 estimate his cost, 19 one <2min action | upstream R2; decision 6; decision 5; upstream R3 |

### Not yet verified

The rules are installed, not proven. Only real use shows which ones miss. Log misses here and
re-tune; the file is under git so every change is diffable.
