# Harness change log (`uvm-harness`)

The cross-job ledger of every harness self-improvement **decision** — the *act* side of the factory's
self-improvement loop. `/uvm-harness` appends one entry per **applied** and **rejected** decision (and
notable **deferred** ones), and **reads this file before applying**: a proposed fix that reverts a
recent change, or repeats a previously-rejected one, is flagged to the human rather than silently
re-applied. This is the loop's anti-thrash memory.

Findings themselves live in each feature's `spec/{slug}/META.md`; this file is the durable record of
what was *done* about them.

Entry format — one section per decision, newest at the bottom:

```markdown
## {YYYY-MM-DD} — {slug} {F#}: {one-line title}
`decision=applied|rejected|deferred commit={sha|—} target={file}`
- **Rationale:** what was changed and why it generalizes / why rejected (overfit, stale,
  would-weaken-a-gate) / why deferred.
```

**One bullet, and it earns its length rather than counting it.** What a later run reads this file for
is not the summary — it is the part that stops a fix being re-applied or re-argued: a fix that was
**rejected** and on what ground, a departure from the finding's own recommendation, a cross-reference
to the commits a decision builds on or narrows, and a mis-citation caught before it landed. An entry
carrying those is doing its job at eight lines; one compressed past them is shorter and useless. Keep
out what the `META.md` finding already says: read `origin`, `severity` and `category` from there,
because this ledger records the *outcome*, not the diagnosis.

---

<!-- Decisions are appended below this line by /uvm-harness. -->

## 2026-08-07 — bootstrap: factory ported from HyperShell
`decision=applied commit=— target=.agents/`
- **Rationale:** initial import, adapted from the HyperShell factory. Renamed `hs-*` → `uvm-*`; base
  branch `develop` → `main` (this repo has no git-flow split); invariants rewritten for a single bash
  script; `temp_site.sh` replaced by `temp_root.sh` plus an offline installer fixture; `lint.sh`
  added as the static gate in place of a test suite; the FSM scripts converted to PEP 723 so they run
  under `uv run` with no project environment. The `Co-Authored-By` trailer is **kept** here, unlike
  the source repo — this repository's history already records it.

## 2026-08-07 — uvm-env-prefix F1: GOAL criteria that only static inspection can check
`decision=applied commit=132b214 target=.agents/factory/templates/GOAL.md`
- **Rationale:** the sandbox-observability rule admitted one departure, for criteria needing a real
  cluster; a documentation criterion had none and got an invented out-of-band annotation instead.
  Generalizes because `AGENTS.md`'s same-commit rule puts a documentation criterion in most cycles.
  Named the second departure rather than relaxing the rule — both still declare themselves inline.

## 2026-08-07 — uvm-env-prefix F2: `planned` was an unreachable FSM state
`decision=applied commit=fef0ccc target=.agents/skills/uvm-plan/SKILL.md`
- **Rationale:** `uvm-plan` hard-coded `status: in_progress` at plan time, so `TECH.md` claimed
  building had begun for the whole duration of the sign-off gate Step 9 then enforces, and `_fsm.py`'s
  `planned` was dead. Plan writes `planned`; `/uvm-build` Step 5 flips it on the first *completed*
  phase. `in_review` is ordered to win, so a single-phase roadmap does not stop at `in_progress`.
  Touched `templates/TECH.md` in the same commit — it is the copied starting point, so leaving its
  frontmatter at `in_progress` would have silently defeated the change.

## 2026-08-07 — uvm-env-prefix F3 + F5: phase gates unchecked against their own checklists
`decision=applied commit=61574ec target=.agents/skills/uvm-plan/SKILL.md`
- **Rationale:** applied as one commit, by agreement — the two findings are the same check in opposite
  directions (a gate that contradicts a checklist item; a gate blind to one), and splitting them would
  have meant the second commit rewriting the first's sentence. Both were observed in one cycle, which
  is what carried them past the overfit test. Guidance, not a new hard rule: the fix names the two
  failure shapes and the three reconciliations, and leaves the judgment at plan time.

## 2026-08-07 — uvm-env-prefix F4: blindness stated as "do not read", evaded by grep
`decision=applied commit=a242486 target=.agents/skills/uvm-review/SKILL.md`
- **Rationale:** `severity=high` but **strengthens** blind-review integrity rather than loosening it,
  so Safety §3's typed override did not apply. The reviewer obeyed the letter and still pulled
  PLAN/TECH/research/META lines in through two recursive greps. Restated the ban as content reaching
  context and carried the diff rule's `':(exclude)spec/'` precision over to searches; both exclusion
  forms were executed against this repo before being written down. `review-rubric.md` moved in lockstep.

## 2026-08-07 — uvm-env-prefix F6: `verify:` authored in one shell, executed in another
`decision=applied commit=c5f39f4 target=.agents/skills/uvm-build/SKILL.md`
- **Rationale:** Step 4 already refused to trust exit 0 but treated the shell as neutral; a gate tested
  interactively went green where it should have been red, because the agent shell's `grep` is a
  function wrapping `ugrep`. Re-confirmed here before writing: `grep` is a shell function in this
  session and `/usr/bin/grep` under `/bin/sh`. The red-before/green-after half is the load-bearing
  part — a gate never observed failing is not a gate.

## 2026-08-07 — maintainer request: nothing retired a seed after its cycle shipped
`decision=applied commit=7d5ac9f target=.agents/skills/uvm-roadmap/SKILL.md`
- **Rationale:** raised by the maintainer, not by a `META.md` finding — the lifecycle promoted an
  `issues/{slug}.md` into a cycle and then left it in the backlog forever. Built as a third
  operational sibling rather than folded into `/uvm-publish`, which was the first design considered
  and rejected: retirement writes outside `spec/`, so publish would have had to exclude `issues/`
  from its staleness gate, and `/uvm-review` demonstrably grades `issues/` content — the F5 finding
  processed earlier the same day was a gate going green with `issues/test-harness.md` stale. Publish
  detects and reports; `/uvm-roadmap` acts. Keeping the deletion verb out of the one irreversible
  step was the secondary reason. Detection keys off `status: adopted:{slug}` rather than the
  filename, since the two differ (`issues/trampoline-ignores-platform-override.md` →
  `trampoline-platform-override`), and the trigger carries `|| true` because `grep` exits 2 while
  printing matches when the gitignored `.security/` lane is absent. The security lane inverts the
  rule and moves rather than deletes: gitignored means no history to recover from. Lifecycle prose
  moved in lockstep — `templates/ISSUE.md`, `uvm-feature`, `AGENTS.md` and `methodology.md` all
  documented retention as unconditional.

## 2026-08-07 — uvm-env-prefix: the first retirement, and the ROADMAP numbering
`decision=applied commit=5bb08e0 target=ROADMAP.md`
- **Rationale:** exercised the new step by hand on the case already sitting in the tree rather than
  waiting for the next cycle to hit it under time pressure. Dropped the `### N.` numbering in the same
  commit: an audit of what removing one entry breaks found every casualty positional (`cycle 1`,
  `of the three`, `Three are already queued`) and every by-name reference intact, so the numbers were
  manufacturing the only breakage and renumbering-on-every-retirement would have been a standing
  chore. The seed's `Seed:` link in `spec/uvm-env-prefix/GOAL.md` is deliberately left dangling —
  `spec/{slug}/` records what was true when written, and the dangling path is the signpost that makes
  `git log --diff-filter=D` a two-step recovery.

## 2026-08-07 — prose-and-comment-pass F1: the ROADMAP edit was named only in the commit recipe
`decision=applied commit=2d5c897 target=.agents/skills/uvm-feature/SKILL.md`
- **Rationale:** Step 7 staged `ROADMAP.md` and Step 4 never mentioned it, so the steps followed
  literally commit an unmodified file, and the convention had to be recovered from `640e2f8` by hand.
  Moved both halves — the adoption marker and the rewritten entry body — into Step 4, with the marker
  shown as an indented example rather than described. Urgency is real: `/uvm-roadmap` now deletes
  retired entries, so the next promotion is the first with no prior example one `git log` away.

## 2026-08-07 — prose-and-comment-pass F2: the criterion-check enumeration went stale in one cycle
`decision=applied commit=d3c64ac target=.agents/factory/templates/GOAL.md`
- **Rationale:** revises `132b214` rather than reverting it. That fix named a second departure from
  sandbox-observability instead of relaxing the rule, and its "declares itself inline" principle is
  kept intact here; only the closed set of two is replaced by the rule it was standing in for. A prose
  criterion — checkable by neither a drive nor a command — was the third case, and the enumeration
  would have needed a fourth revision on the next unanticipated one. `uvm-feature` Step 4 duplicates
  the rule and moved in lockstep.

## 2026-08-07 — prose-and-comment-pass F3: a gate was never checked for being able to fail
`decision=applied commit=87473cc target=.agents/skills/uvm-plan/SKILL.md`
- **Rationale:** extends `61574ec`, whose two read-back directions (contradiction, blindness) are both
  about scope and neither catches an inert gate. Adds the control that plan time was missing: run each
  `verify:` now, and a gate asserting an undelivered post-condition must come back red. The plan-time
  analog of `c5f39f4`'s red-before/green-after at build time. Named the concrete trap — an interpolated
  pathspec collapses under `zsh` — because the abstract rule is what failed to fire.

## 2026-08-07 — prose-and-comment-pass F4: the blast-radius trigger read as adjacency, not behavior
`decision=applied commit=6966001 target=.agents/skills/uvm-plan/SKILL.md`
- **Rationale:** this **narrows a research trigger**, so it went to the human explicitly rather than on
  the finding's say-so; the exception is not an `invariants.md` item or a hard gate, so Safety §3 did
  not bind. Applied the conservative form: it still fires unless the change is *provably* confined to
  comments, message strings or documentation, and message text in a §3 or §7 region keeps a mandatory
  `research/` baseline, since a string those regions print is user-facing behavior. A future run
  proposing to drop that carve-out should treat it as a re-litigation, not a new finding.

## 2026-08-07 — prose-and-comment-pass F5: the fan-out fallback tested availability, not permission
`decision=applied commit=4fe0252 target=.agents/factory/portability.md`
- **Rationale:** subagents existed and worked; the session forbade spawning them unasked, a case the
  condition as written did not cover. Widened at all four sites (`portability.md`, `uvm-plan` twice,
  `uvm-review`) since an agent policy, cost ceiling or sandbox restriction hits every skill that fans
  out. Added that the deliverable is identical either way — without it the fallback reads as a scope
  cut, which is the reason an agent would resist taking it.

## 2026-08-07 — prose-and-comment-pass F6: two skills gave different rules for the commit category
`decision=applied commit=9228a75 target=.agents/skills/uvm-build/SKILL.md`
- **Rationale:** `uvm-build` said `kind`, `uvm-plan` said the shape commit, so a prose cycle had to
  adjudicate `[refactor]` against `[docs]`. `kind` is the lifecycle taxonomy and has no `docs` or
  `harness` member. Took the deferral option over the finding's alternative of a new `TECH.md`
  frontmatter field: the field would have meant touching `_fsm.py`'s `FIELD_ORDER`, `validate` and
  `set_phase.py` to record something already legible in the branch's first commit. `uvm-review` Step 1
  carried the same conflation and moved with it. `uvm-publish` leaves `{category}` undefined rather
  than wrong, so it was left alone — widening to it would have been scope creep past the finding.

## 2026-08-07 — prose-and-comment-pass F7: the prose exception was scoped to the diff hunk
`decision=applied commit=5dd4c7d target=.agents/factory/review-rubric.md`
- **Rationale:** hunk scoping is right for a feature cycle, where untouched prose is somebody else's
  problem and a sweeping reviewer manufactures gaps. It inverts when the prose *is* the deliverable:
  the work is defined as the lines the pass chose not to change, so grading only what moved cannot see
  an omission. The rubric now says which reading wins and keeps the anti-gap-hunting rule everywhere
  else; the `uvm-review` Safety Principles summary points at it.

## 2026-08-07 — prose-and-comment-pass F8: remediation was written as local to one phase
`decision=applied commit=1a83475 target=.agents/skills/uvm-build/SKILL.md`
- **Rationale:** a near miss, not a hypothetical — a one-word fix moved a census total from 7 to 6 and
  a `done` reconciliation phase's gate still hardcoded 7. `next_phase.py` never re-runs a gate, so a
  stale assertion in a `done` phase is invisible to the FSM and the branch ships green over a gate that
  fails on the tree it certifies. Scoped the re-run to `done` phases whose `depends_on` names the
  reopened one, rather than all of them: this factory's plans routinely end in a count-snapshotting
  phase, which is exactly the shape that breaks.

## 2026-08-07 — prose-and-comment-pass F9: the mandatory /bin/sh drive had no supported command
`decision=applied commit=2895fc1 target=.agents/factory/bin/run_verify.py`
- **Rationale:** makes `c5f39f4` executable rather than changing it. Step 4 required the drive and left
  the gate locked in folded YAML, so it was hand-reflowed twice via throwaway readers — and reflowing
  is where a quoting error enters. The empty-string guard is the load-bearing part: `/bin/sh -c ''`
  exits 0, so the first attempt's missing `pyyaml` produced an empty string that *satisfied* "confirm
  it is red". Chose a fourth script over documenting a one-liner because the false green is what needed
  to become impossible, not inconvenient. It `exec`s rather than spawning, so the gate's own status
  reaches the caller — verified at 1, 42 and 0, and the empty gate at 2.

## 2026-08-07 — prose-and-comment-pass F10: commit subjects leaked past the diff's pathspec
`decision=applied commit=b5a9826 target=.agents/skills/uvm-review/SKILL.md`
- **Rationale:** extends `a242486`, which closed the diff channel and handed the same information back
  through the log in the next clause. `severity=high`, but like its predecessor it **strengthens**
  blind-review integrity, so Safety §3's typed override did not apply. Two halves, because one pathspec
  is not enough: review-cycle commits touch only `spec/` and vanish under it, but build subjects name
  the remediated finding id, so cycle ≥ 1 drops subjects entirely. Verified by constructing a spec-only
  review commit and watching it disappear from the filtered log.

## 2026-08-07 — prose-and-comment-pass F11: a verification technique had nowhere blindness-safe to live
`decision=applied commit=65fa2c6 target=.agents/factory/review-rubric.md`
- **Rationale:** took the rubric option and **rejected the finding's alternative** of a
  blindness-safe `REVIEW.md` section paste-forwarded by Step 2. That design makes every cycle judge
  what is safe to carry across the blindness boundary, and the judgment is the risk; the rubric already
  reaches every reviewer and carries no author intent. A future run proposing the `REVIEW.md` channel
  should read this first. Added the `grep`-is-not-`grep` trap next to the `zsh` one — same class, and
  both were reproduced live while applying this run, the `zsh` one by a detection loop written for this
  very ledger entry.

## 2026-08-07 — prose-and-comment-pass: the run's own commit-subject format does not fit
`decision=applied commit=2d5c897 target=.agents/skills/uvm-harness/SKILL.md`
- **Rationale:** noted, not fixed. `uvm-harness` Step 5 prescribes
  `[harness] {summary} ({slug} F#)`, which spends 38 characters on overhead for a slug this long and
  cannot fit `AGENTS.md`'s 72-character subject limit. Two subjects were amended after the fact this
  run. Followed the repository's actual precedent instead — `61574ec`, `fef0ccc` and every prior
  harness commit omit the suffix and let this ledger carry the finding linkage. Left as an observation
  because `uvm-harness` never writes `META.md` findings (Safety §5) and a self-edit mid-run is the
  meta-on-meta the same rule forbids.

## 2026-08-09 — trampoline-ignores-platform-override F1: promotion never read the roadmap entry
`decision=applied commit=56dd89c target=.agents/skills/uvm-feature/SKILL.md`
- **Rationale:** extends `2d5c897`, which moved the `ROADMAP.md` *edit* into Step 4 and left the
  *read* out. The two halves of a deferral carry different things — evidence in the issue, position
  and the reasoning for that position in the entry — and only the entry can say a promotion jumps
  recorded order. Confirmed live before writing: the trampoline entry carries "Deliberately sequenced
  after the test harness", and `issues/trampoline-ignores-platform-override.md` does not. Guidance
  rather than a gate: it surfaces the reordering for sign-off, it does not refuse the promotion.

## 2026-08-09 — trampoline-ignores-platform-override F2: gates were red-tested but never green-tested
`decision=applied commit=e935214 target=.agents/skills/uvm-plan/SKILL.md`
- **Rationale:** extends `87473cc`, which added the red check and treated red as proof. **Rejected
  the finding's blanket form** — green-proving every gate against a scratch copy makes plan time
  rehearse each phase's change, which is the build. Took the cheap always-applicable half instead:
  red is necessary, not sufficient, so read the failure output and confirm the gate died on its
  asserted post-condition rather than on a typo or a mangled string. The scratch copy is named as the
  sanctioned stronger proof when the output does not settle it, with the "Never build" carve-out that
  made it look forbidden. The recipe was executed here before it was written down — `cp -R` to a
  temp dir, `temp_root.sh --offline uv --version` from inside the copy, `uv 9.9.9 (fixture)`.

## 2026-08-09 — trampoline-ignores-platform-override F3: `verify:` is a YAML scalar first
`decision=applied commit=3ce4978 target=.agents/factory/templates/TECH.md`
- **Rationale:** **rejected the finding's second site.** It also wanted the trap in
  `review-rubric.md` § *Verification traps* next to the `zsh` and `grep` entries added by `65fa2c6`,
  but that section's audience is the blind reviewer, who grades a diff and never authors `verify:`
  YAML; `run_verify.py` (`2895fc1`) removed the hand-reflow path that was the only way it could have
  reached them. The template's field reference is read by `uvm-plan` Step 6 and by `uvm-build` when
  it amends a phase, which is everyone the trap can bite. One site, not two.

## 2026-08-09 — trampoline-ignores-platform-override F4: nothing offered the debate variant
`decision=applied commit=b01bbe2 target=.agents/skills/uvm-review/SKILL.md`
- **Rationale:** the rubric named the condition and the skill never told the orchestrator to detect
  it, so `/uvm-review` on the highest-risk diff in the repository ran the same single pass as one on
  a typo fix. Offered, never automatic: the rubric is right that a second reviewer costs twice as
  much, and that is the human's call. A future run proposing to make it fire automatically is
  re-litigating this.

## 2026-08-09 — trampoline-ignores-platform-override F5: a false `cp` fact steering PR bodies
`decision=applied commit=7f65759 target=.agents/skills/uvm-publish/SKILL.md`
- **Rationale:** `0745a25` fixed the claim in `README.md` and left the skill's copy standing, where
  it steers what an agent writes into a PR body — and it did, into the first draft of the 0.4.0
  release notes. Named the platform split rather than dropping the clause, matching the README's
  now-accurate wording and pointing at it, so the next drift is a one-file check. The lesson the
  finding actually carries: a fact corrected in the product documentation has to be chased into the
  harness, which no step currently does.

## 2026-08-09 — trampoline-ignores-platform-override F6: the operational siblings had no intake
`decision=applied commit=bced44f target=.agents/skills/uvm-release/SKILL.md`
- **Rationale:** `uvm-release` routed friction to `/uvm-harness`, which states twice that it never
  writes findings — a consumer named as a destination, so the loop had no writer and F5 survived only
  because the human read it in the final report. Re-derivation turned up what the finding did not
  know: the obvious destination collides with `uvm-roadmap`'s own "Never edit `spec/{slug}/`" and
  with `uvm-release`'s "touches no `spec/`". Resolved by naming `META.md` the explicit exception —
  it is the harness log, not the dated design record — and keeping the write human-gated, which is
  how F5, F6 and F7 actually reached this file (`69434b1`, `93848da`). **Widened to `/uvm-roadmap`**,
  which the finding named as having the same hole. Both skills gain `Write`: the target cycle may
  have no `META.md` yet, and every lifecycle skill's phrasing is "create from the template if absent".

## 2026-08-09 — trampoline-ignores-platform-override F7: `target=` had two spellings for one file
`decision=applied commit=b3650fe target=.agents/factory/templates/META.md`
- **Rationale:** `.claude` is a symlink to `.agents`, so F4 recorded a file that F1, F2, F3 and F5
  recorded under another name, in one `META.md`. Pinned the form in the template's schema paragraph
  and normalized F4's stored value in the same commit. Consistency, not detection: `--all` weighs
  recurrence across jobs and an agent can still see the two match, which is why this stayed `low`.

## 2026-08-09 — roadmap-sweep F1: a skill's own state injection could brick it
`decision=applied commit=4844780 target=.agents/factory/bin/lint.sh`
- **Rationale:** `/uvm-roadmap` could not load at all. `grep -r` over an absent `.security/issues`
  exits 2 while still printing every match, and the harness aborts the whole skill on the status, so
  the correct answer arrived rendered as a fault. Step 2 had documented the `|| true` as load-bearing
  since the skill was written; the injection line on 49 never got it, and the same shape was latent in
  `uvm-harness` and `uvm-release` where no `META.md` and no match are both normal. Fixed in `0a2346b`;
  this entry records the generalization. Empty state is only reachable from a repository in that
  state, so a written rule would not have held — `lint.sh` now executes all 29 injections and fails on
  a non-zero exit, verified red by reintroducing the unguarded grep. The rule is in `portability.md`
  because that is where the injection affordance is already specified.
- **Note on provenance:** this and F2–F4 below have no `spec/{slug}/META.md`. They were raised and
  remediated inside the same sweep at the maintainer's direction, and the only cycle available to file
  them against was one this sweep retired, which would have misfiled them.

## 2026-08-09 — roadmap-sweep F2: the pre-flight STOP was circular for a self-repair
`decision=applied commit=6e93092 target=.agents/skills/uvm-roadmap/SKILL.md`
- **Rationale:** Step 1 STOPs on a dirty tree and named no remedy, so a sweep blocked by a defect in
  this skill could not proceed — repairing it dirties the tree, and the dirty tree stops the sweep.
  Narrowed deliberately: only a repair that made the skill loadable passes, it commits separately, and
  it is never folded into a retirement commit. **Not widened** — `uvm-harness` Step 1 already says
  "commit, stash, or discard first" and does not have the hole.

## 2026-08-09 — roadmap-sweep F3: Step 5 named the breakage but not how to find it
`decision=applied commit=1a816cb target=.agents/skills/uvm-roadmap/SKILL.md`
- **Rationale:** the step described the *kinds* of stale reference a retirement leaves and left
  discovery to whoever thought to grep. This sweep's three real hits — a dangling seed link in
  `issues/test-harness.md` and two skill examples, one asserting a slug the promotion never produced —
  were found that way and could as easily have been missed. Added the search over both the filename
  and the slug, and a triage table, because the hit list is mostly destinations that must be left
  alone: an agent handed one without the table edits `spec/`, which a safety principle forbids.

## 2026-08-09 — roadmap-sweep F4: a conditional non-goal was never verified as landed
`decision=applied commit=faf0bca target=.agents/skills/uvm-roadmap/SKILL.md`
- **Rationale:** the highest-severity of the four, and the only one that loses work. A non-goal that
  discharges itself by pointing elsewhere — "the harness cycle must cover this" — is conditional, and
  the condition is the whole reason the cycle shipped without the work. Step 3 read as "check nothing
  was cut", which passes, rather than "check the named destination has it", which did not:
  `issues/test-harness.md` never carried the trampoline regression case, and the obligation existed
  only in the ROADMAP entry the retirement deletes. Caught by reading the non-goals closely, which is
  not a control. The step now requires opening the named file before deleting anything.

## 2026-08-14 — purge-resilient-run F10: a deferring non-goal was verified only at deletion
`decision=applied commit=86d307f target=.agents/skills/uvm-feature/SKILL.md`
- **Rationale:** extends `faf0bca` rather than repeating it. That entry put the check in
  `/uvm-roadmap` Step 3, which is correct and correctly positioned as a backstop — but it runs at
  deletion, after ship and merge, and only if someone runs the sweep. This puts the obligation where
  the promise is made: land it in the named seed as an acceptance criterion, in the shaping commit, or
  do not write the non-goal that way. **Dropped the finding's `/uvm-publish` half** — `7d5ac9f`
  already considered and rejected folding this class of check into publish, and the draft conceded the
  consequence in its own text (writing the obligation there puts a commit outside `spec/`, where the
  staleness gate fires). Safety §6 flag raised and resolved by narrowing, not layering. Two cycles had
  already discharged "no committed regression test" against `issues/test-harness.md` without writing
  it there.

## 2026-08-14 — purge-resilient-run F11: the rehearsal teardown could not succeed
`decision=applied commit=7aff491 target=.agents/skills/uvm-release/SKILL.md`
- **Rationale:** Step 2.2 mandates applying the bump inside the worktree and Step 2.3 then removed it
  without `--force`, so the two steps guaranteed each other's failure — exit 128 on every release, for
  everyone. Reproduced here before writing. Fixed both sites: the step, and the Safety Principles line
  claiming `git worktree remove` "cleans the rehearsal", which also left the `mktemp` parent behind.
  Kept an inspection before the force rather than forcing blind — reaching for `--force` on a failure
  is the moment you most want to see what is about to be discarded.

## 2026-08-14 — purge-resilient-run F2: the same-commit rule omitted the invariant records
`decision=applied commit=46db87c target=AGENTS.md`
- **Rationale:** applied **narrow**, against the finding's own recommendation, after an adversarial
  pass. The drafted fix also amended `uvm-harness`'s "only skill that writes to `.agents/`" guardrail
  to be conditional and gave `/uvm-build` a standing licence to edit `invariants.md` mid-build. That
  route is ungated in a way the prose could not see: `/uvm-review` is handed `invariants.md` **from
  the branch tree**, so on a branch that edited it the reviewer grades against the loosened standard,
  and it is forbidden `PLAN.md`/`TECH.md`, so it cannot check that a human cleared the bend. Safety §3
  never fires because the route never enters `uvm-harness`. What landed is the record correction in
  `AGENTS.md` and `invariants.md` §12 (lockstep), plus a `uvm-plan` phase item so the overturn travels
  the planned path, which has a human at Step 5. A future run proposing the build-time route should
  read this first and bring the three missing controls with it.

## 2026-08-14 — purge-resilient-run F4: `! cmd` is exempt from errexit
`decision=applied commit=aa2e143 target=.agents/skills/uvm-plan/SKILL.md`
- **Rationale:** third revision of Step 6's gate-authoring guidance after `87473cc` and `e935214`, and
  compatible with both. **Rejected the finding's blanket form** — it wanted `! cmd` forbidden outright,
  which would condemn four committed gates that use it correctly. Verified: `sh -c 'set -e; ! true'`
  exits 1, while `sh -c 'set -e; ! true; echo REACHED'` prints `REACHED` and exits 0. Only a `! cmd`
  with something after it goes inert, so the rule is the trap, not a ban. Paired with the
  first-unmet-clause refinement, which is how the inert assertion becomes visible in a multi-clause
  gate.

## 2026-08-14 — purge-resilient-run F7: a gate's pathspec ignored its criterion's quantifier
`decision=applied commit=93c951d target=.agents/skills/uvm-plan/SKILL.md`
- **Rationale:** applied as **one sentence appended to the existing blind-gate paragraph**, not the
  nine-line section the finding drafted. Step 6 already names the failure ("a gate can also be
  **blind** to one … goes green, and the judgment item ships unchecked"); an enumerated pathspec
  against "wherever it is stated" is that same failure wearing a scope. Safety §4 prefers the example
  over a new rule, and the paragraph was eight lines above the proposed insert.

## 2026-08-14 — purge-resilient-run F5: the injected diffstat was silently truncated
`decision=applied commit=c106d1d target=.agents/skills/uvm-review/SKILL.md`
- **Rationale:** the `--stat` elided its first two rows while its own summary still read "21 files
  changed", so it read as complete — and the two dropped rows were the files carrying two R-IDs.
  Replaced with `--name-only`, which has no summary line to lie with, and carried the reviewer's own
  `':(exclude)spec/'` pathspec onto it so the injected list and the graded diff agree. Removed
  `Bash(tail *)` in the same commit: that pipe was its only consumer, and `uvm-harness` Step 6 audits
  `allowed-tools` against the commands a skill actually calls.

## 2026-08-14 — purge-resilient-run F6: the human gate had no clearance record
`decision=applied commit=7633902 target=.agents/factory/review-rubric.md`
- **Rationale:** **safe half only, by explicit maintainer decision.** The finding argued to narrow the
  trigger so a prose-only finding stops firing the gate; that loosens a non-negotiable guardrail on a
  finding's say-so, which Safety §3 treats as a warning sign, and the maintainer refused it. Both
  trigger bullets are byte-unchanged, verified after applying. What landed is the clearance record:
  cleared by the human and never by the agent's own reading, written into `REVIEW.md` under
  *Human-gate triggers* with who, when and on what grounds, plus the matching field in
  `templates/REVIEW.md`. `uvm-publish` gates on the verdict and the staleness check alone, so that
  section is the only durable evidence a human ever saw the finding. The narrowing stays open.

## 2026-08-14 — purge-resilient-run F9: a scoped cycle contradicted the dropped log subjects
`decision=applied commit=f6748ef target=.agents/skills/uvm-review/SKILL.md`
- **Rationale:** Step 3 sanctioned scoping a later cycle "to verifying the remediation of named
  findings" while Step 2, since `b5a9826`, drops the commit subjects precisely because they name
  findings. The two were written against different concerns and never reconciled, so the natural
  reading of Step 3 defeats Step 2 on the cycle where anchoring is most likely. A scope now reaches
  the reviewer as a range and a graded surface. The R-ID disclosure is kept and labelled as the
  deliberate, bounded exception it is — without it the reviewer reads unchanged implementations as
  unmet. **Cited** the rubric's file-versus-hunk rule rather than restating it.

## 2026-08-14 — purge-resilient-run F8: the blind boundary is drawn at `spec/`, and leaks
`decision=applied commit=b8afd6a target=.agents/factory/review-rubric.md`
- **Rationale:** the third leak channel after `a242486` (the grep sweep) and `b5a9826` (the log
  subjects), and the first that cannot be closed. A cycle that defers work is *required* to write its
  reasoning into `issues/{slug}.md` and `ROADMAP.md`, both outside `spec/` and both legitimately in
  the graded diff, so a correctly filtered pass still reads the plan's conclusions. Named the limit
  and installed skepticism — read the diff's own new prose as a claim to verify — and **rejected
  excluding `issues/`**, which would blind the reviewer to work it must grade. Two mis-citations in
  the draft were caught and dropped: `AGENTS.md` does not require sequencing rationale in the roadmap
  entry (that is `uvm-feature`), and nothing anywhere requires citing `research/` by filename.

## 2026-08-14 — purge-resilient-run F1: prose beside a path was parsed as scope
`decision=applied commit=ab57630 target=.agents/skills/uvm-feature/SKILL.md`
- **Rationale:** follows `56dd89c`'s shape — surface and ask, never refuse. Trimmed to the general
  rule: prose alongside a path shapes that seed and never extends scope, and an ambiguous owner is a
  question for the human. **Dropped the finding's clerical half** (where to file the remark, an extra
  `git add`) as machinery for the job that found it; Step 4's new non-goal rule from F10 already
  carries the routing, and duplicating a shaping rule is what `d3c64ac` shows the cost of.

## 2026-08-14 — purge-resilient-run F3: no route back to shaping from plan time
`decision=deferred commit=— target=.agents/skills/uvm-plan/SKILL.md`
- **Rationale:** real and unfixed. Research establishing that a locked `GOAL.md` cannot be built is a
  *success* of the plan step, and it is the one outcome with no procedure: `uvm-plan` may not edit
  `GOAL.md`, and `uvm-feature` refuses to run off `main` on a clean tree. Deferred rather than applied
  because the fix is a design conversation, not a wording change — it needs a new "bounce to shaping"
  step, `uvm-feature` accepting a branch carrying only `GOAL.md`/`META.md`/`research/`, and a decision
  between re-shaping in place and parking the branch. This cycle worked around it by hand and the
  maintainer chose the narrowing, so the workaround is proven but undocumented.

## 2026-08-15 — doctor-detection-gaps F10: the log's blinding flag was a fragment that no-ops
`decision=applied commit=4fe795a target=.agents/skills/uvm-review/SKILL.md`
- **Rationale:** the fourth pass over this leak channel, after `a242486` (the diff), `b5a9826` (the
  log subjects) and `f6748ef` (scoped cycles). It **completes `b5a9826` rather than reverting it**:
  that entry wrote the correct rule as a flag to append to a command printed earlier, and the only
  correct composition requires moving the flag ahead of the `--`. Appended after it, `--format=%h`
  is a pathspec, `--oneline` survives, and the subjects print at exit 0 — reproduced twice
  independently, and it leaked a prior cycle's finding id into a live review this cycle. Now the
  whole command is printed for `review.cycle` ≥ 1. **Strengthens** blind-review integrity, so Safety
  §3's typed override did not apply, on the same reasoning `b5a9826`'s own entry recorded. A future
  run must not re-compress this into a flag reference; the compression is the defect.

## 2026-08-15 — doctor-detection-gaps F1: `medium` was legal in a seed and illegal in a GOAL
`decision=applied commit=fe45b9b target=.agents/factory/templates/ISSUE.md`
- **Rationale:** took the second of the finding's two directions — drop `medium` from the seed
  template and state the rounding — and **rejected the first**, adding `medium` to `templates/GOAL.md`
  with a phase-cap meaning defined in `methodology.md`. That would invent a third budget tier and
  oblige every downstream scope check to interpret it, to fix a handoff that only needs one
  vocabulary. Rounding up costs a research fan-out; rounding down fails `uvm-review` against a
  contract a human already accepted, so the rule rounds up. **Deliberately did not rewrite the three
  queued seeds still carrying `appetite: medium`** — those are a human's recorded budget judgment on
  repo content outside the factory, and the rounding rule covers them at promotion.

## 2026-08-15 — doctor-detection-gaps F2: `kind` and `appetite` selected opposite research paths
`decision=applied commit=899efd6 target=.agents/skills/uvm-plan/SKILL.md`
- **Rationale:** Step 3's skip bullet listed `kind: fix` beside `appetite: small` as if the two could
  not disagree; a `fix` at `appetite: big` matched both bullets. Took the precedence line — appetite
  governs depth, `kind` does not — over making each bullet's condition conjunctive, because the
  precedence states the intent once where a conjunction restates it per bullet and drifts. The
  diagnostic-fixes exception below is untouched and still overrides both.

## 2026-08-15 — doctor-detection-gaps F3: a gate anchor that wraps out of `git grep`'s reach
`decision=applied commit=9841158 target=.agents/skills/uvm-plan/SKILL.md`
- **Rationale:** same class as the `zsh` pathspec trap already in the paragraph, and likelier here:
  `README.md` and `AGENTS.md` hard-wrap near 100 columns, so a phrase long enough to be unique
  usually spans two lines and a line-based `git grep` never matches it. The failure is a gate
  asserting a sentence is gone that reads green while the sentence stands. Added beside the existing
  hazards rather than as a subsection.

## 2026-08-15 — doctor-detection-gaps F4: a gate covered half a criterion and read as full coverage
`decision=applied commit=1ef1614 target=.agents/skills/uvm-plan/SKILL.md`
- **Rationale:** third sighting, and applied in **half** the form the finding drafted. Took the R-ID
  accounting obligation — the gate encodes the criterion's *Checked by* clause, or the phase body
  names the reviewer as grader for the part no command can decide — plus the multi-location
  generalization. **Dropped the "certify a preservation assertion by tampering" half**, because
  `e935214` already rejected the neighbouring proposal that every gate be green-proved against a
  scratch copy, on the grounds that it makes plan time rehearse the build. A future run tempted by
  tampering should read `e935214` first. Adversarial review caught the first draft dropping the
  "cannot mechanize" bound, which would have turned reviewer-delegation into a free choice at plan
  time and sanctioned the exact R5 miss that produced the finding; the bound is restored, and the
  delegation is stated once in the tail that already carried it rather than twice. Trimmed to two
  hunks, matching the standard `93c951d` set on this same paragraph.

## 2026-08-15 — doctor-detection-gaps F5: "a red gate is a STOP" conflated iterating with failing
`decision=applied commit=0b2caa0 target=.agents/skills/uvm-build/SKILL.md`
- **Rationale:** the durable `attempts` breaker trips at about three, so recording every
  mid-implementation red would trip it on a phase converging exactly as planned, and the signal
  `/uvm-plan` receives from a tripped breaker becomes noise. The STOP is now scoped to a gate that
  *stays* red — no correction left to try, or the predicted one did not clear it — which keeps the
  breaker meaningful rather than granting a licence to loop. **Three sites, not one:** Safety
  Principles stated the unscoped rule twice, and "Every red verify gate is recorded on file" would
  have defeated the fix on its own.

## 2026-08-15 — doctor-detection-gaps F8: refutation certified the observation, never the mechanism
`decision=applied commit=d0dc033 target=.agents/factory/review-rubric.md`
- **Rationale:** a CONFIRMED finding read "exits 1 without repairing anything", measured on a
  one-tool fixture where "fails wholesale" and "repairs the rest, then fails" emit the same rc 1. The
  observation was right and the mechanism was not, and the mechanism is what a remediation encodes —
  had it been trusted, the shipped tool would now talk users out of a command that repairs their
  tree. The protocol renumbers to five steps and adds the distinguishing step, keeping the concrete
  "at least two of whatever the mechanism ranges over" heuristic as the memorable part. The split
  verdict — observation CONFIRMED, mechanism out separately as PLAUSIBLE — is new, and sits inside
  the existing definitions rather than redefining them.

## 2026-08-15 — doctor-detection-gaps F9: REVIEW.md could not retract what a later cycle disproved
`decision=applied commit=9a6d0f4 target=.agents/skills/uvm-review/SKILL.md`
- **Rationale:** append-only is right for an audit trail and wrong for the durable account
  `uvm-publish` surfaces in the PR — the most prominent claims end up the oldest, and a finding a
  later cycle overturned propagates as fact. The convention invented by hand this cycle is now
  named: a `### Correction to cycle {n}` note inside the correcting cycle's own section, never an
  edit to the earlier one. Two sites, skill and template, because the template is what a cycle
  copies. **Left `review-rubric.md`'s own copy of the append-only rule alone** on minimality;
  Step 3 is what the orchestrator executes.

## 2026-08-15 — doctor-detection-gaps F11: the drift sweep was gated on `--all`
`decision=applied commit=8226b47 target=.agents/skills/uvm-roadmap/SKILL.md`
- **Rationale:** extends `1a816cb`, which gave Step 5 its search and triage table. The `--all` bullet
  scoped the sweep by *which seed* held the stale figure when the discriminator that matters is
  whether the sweep is already opening the file: a false figure beside a freshly repaired link is
  worse than one nobody touched, because the repair signals the file was reviewed. Split in two, with
  the bounded half in the default sweep. Second site is the `--all` glossary line in Argument
  Parsing, which an agent reads first and which would otherwise contradict Step 5 about what the flag
  controls.

## 2026-08-15 — doctor-detection-gaps F6: a GOAL may cite a `spec/` record the reviewer may not read
`decision=applied commit=5d65af7 target=.agents/skills/uvm-review/SKILL.md`
- **Rationale:** **applied under an explicit Safety §3 override**, asked for and granted after the
  fix was authored and measured, because blind-review integrity is named on that list and the earlier
  consent was a choice among forms made before anyone knew the narrow form still loosens the text.
  The maintainer granted it on the ground the flag understates: today's practice is hand-pasting the
  cited file's contents into the reviewer prompt, unbounded and unrecorded, so one path named by the
  locked contract is strictly less. **Rejected the finding's own fix** — redrawing the boundary at
  `spec/{slug}/` plus `spec/*/REVIEW.md` — which would pass every sibling cycle's GOAL and research
  by default. Three confinements hold: cited by the locked `GOAL.md`, a landed cycle's record only,
  and a path rather than contents. Adversarial review caught the first draft failing its own
  motivating case, since the `research/` category ban is stated separately and the draft did not
  scope it; the applied text says the ban is on this branch's artifacts and the exception is the
  cited path, not the category. It also dropped a "records are kept for later work to cite"
  justification that `methodology.md` does not support — the same class of mis-citation `b8afd6a`
  removed from a draft one entry back. The rubric's § *What the reviewer sees* still carries the
  blanket ban and was left alone as outside the granted scope; that contradiction is known.

## 2026-08-15 — doctor-detection-gaps F7: the rubric graded four severities and routed on one bit
`decision=applied commit=cd995b4 target=.agents/factory/review-rubric.md`
- **Rationale:** departed from in both directions from the same defect — one cycle blocked on a LOW
  and consumed a cycle of a bounded loop, another approved past two CONFIRMED LOWs because blocking
  was indefensible. **Rejected the finding's own fix**, routing the auto-block on CONFIRMED at MEDIUM
  or above, which turns every verdict into a severity argument and is the loosening Safety §3 treats
  as a warning sign. Took instead the one case that needs no severity judgment and is checkable
  against artifacts already in hand: the behavior predates the diff *and* a criterion requires
  preserving it, so repairing it this cycle would fail that criterion — the recurring shape is a
  parity criterion demanding the rewritten code reach the old code's verdict. If either condition
  fails, it blocks. Adversarial review found the draft's prescribed deferral writes `issues/{slug}.md`
  and `ROADMAP.md`, both outside `spec/`, which trips `uvm-publish`'s staleness gate and returns the
  branch to review — the loop the carve-out exists to prevent; the applied text orders that commit
  before `set_phase.py --reviewed-commit`. Carried into `uvm-review` Step 4 in the same commit,
  which still routed every CONFIRMED to blocked and would otherwise contradict the rubric, the shape
  `9228a75` and `f6748ef` were written to stop.

## 2026-09-07 — lock-acquire-retake F4: the debate variant's two reviewers shared one scratchpad
`decision=applied commit=e74710d target=.agents/skills/uvm-review/SKILL.md`
- **Rationale:** both reviewers wrote a `mkdir` PATH shim to the same path; one reported its fixture
  overwritten mid-pass and re-ran its whole set isolated, and the other returned a HEAD failure rate
  the orchestrator's instrumented re-measurement could not reproduce. The variant's entire claim is
  two independent measurements, and shared mutable fixtures make them dependent in a way neither
  reviewer can detect from inside — the drives still complete and still look like evidence. Does not
  re-litigate `b01bbe2`, which settled that the variant is offered and never automatic; this is
  about isolating the reviewers it launches, not when to launch them.

## 2026-09-07 — lock-ownership-and-hold-time F20: Step 2 inlined a file inside the graded diff
`decision=applied commit=e2d26c3 target=.agents/skills/uvm-review/SKILL.md`
- **Rationale:** `invariants.md` is routinely part of the diff under review, so pasting it hands the
  reviewer an orchestrator-mediated second version of the thing being graded, and doubles a long
  prompt across two reviewers. Both it and `review-rubric.md` sit outside `spec/`, so reading them
  from the working tree costs no blindness. `GOAL.md` stays inline for the reason it always was — it
  lives under `spec/`, which the reviewer must not browse. Independently confirmed before the
  finding was read: this cycle's orchestrator pointed at the paths rather than pasting, for the same
  reason. Leaves `5d65af7`'s cited-`spec/`-record exception untouched.

## 2026-09-07 — lock-ownership-and-hold-time F19: the debate variant defined its input, not its output
`decision=applied commit=1e3b128 target=.agents/factory/review-rubric.md`
- **Rationale:** **departed from the finding's own fix**, which named two modes — overlap as a
  confidence signal, disjointness as a coverage signal. This cycle produced a third its categories do
  not cover: the two reviewers *contradicted* each other, one clean and one CONFIRMED HIGH on the
  same function. Contradiction is a claim about reproducibility rather than about coverage, and the
  applied text says the orchestrator settles it by measuring — with a control proving the
  construction reaches the state at all before a clean result is read as absence, which is the step
  that actually resolved it here and the one neither reviewer performed. Recording which of the three
  occurred is kept from the finding as written.

## 2026-09-07 — lock-ownership-and-hold-time F25: the loop bound had no terminal state
`decision=applied commit=ee91e9f target=.agents/factory/review-rubric.md`
- **Rationale:** "at most two or three" is a range with no tiebreak, ambiguous at exactly the cycle
  where it binds, and a prior cycle-3 verdict had to invent its reading and write it into `REVIEW.md`
  as if it were the rule. Cycle 3 is now the last that may set `changes-requested`, and escalation is
  a named handoff — standing findings, remediation delta, ship/abandon/rescope — rather than an
  unspecified event. Carried into `uvm-review`'s Safety Principles and Final report in the same
  commit, both of which mirrored the old range and would otherwise contradict the rubric: the shape
  `cd995b4` recorded and `9228a75`/`f6748ef` were written to stop.

## 2026-09-07 — lock-ownership-and-hold-time F4 + F14: the checklist was derived from prose, not from the code
`decision=applied commit=cb046f4 target=.agents/factory/invariants.md`
- **Rationale:** a bullet graded as auto-CRITICAL must first be true; the header now requires confirming an invariant holds on `main` before raising one, and makes a false claim a finding against this file. F14 rides here: its content half (§5's contention premise) already landed with that cycle's R7/P7 under the same-commit rule, and its surviving half is exactly this standing rule — this run did not re-fix §5.

## 2026-09-07 — lock-ownership-and-hold-time F5: §6 generalized one failure path's message to all three
`decision=applied commit=0de18eb target=.agents/factory/invariants.md`
- **Rationale:** measured in `uvm_install`: the egress path carries the pre-warm text, the version read-back deliberately omits it (pre-warming would repeat a download that succeeded), and the rename is guarded by nothing. Names all three rather than quantifying the first over the rest; the unguarded rename stays code work in `issues/invariant-audit-gaps.md`.

## 2026-09-07 — lock-ownership-and-hold-time F6: §9 dropped the qualifiers on the overwrite guard
`decision=applied commit=38cf89b target=.agents/factory/invariants.md`
- **Rationale:** the guard is a three-way conjunction, so an unmarked file failing `-s` or `-x` is written over; both files claimed removal-and-overwrite are marker-only. Two-file lockstep with `AGENTS.md`. Written to **name the `-x` gap** and point at the seed rather than quietly dropping the claim — stating a weaker truth plus the gap is a strengthening, so no Safety §3 override was needed.

## 2026-09-07 — lock-ownership-and-hold-time F7: §11's rationale was disproved by uv's CLI
`decision=applied commit=f3d6466 target=.agents/factory/invariants.md`
- **Rationale:** both files asserted anything option-shaped can only follow the subcommand; `uv --cache-dir DIR tool dir` and `--python-preference` refute it on 0.12.4. Kept the real point — the list is not a `uv` CLI model — while naming the two missing entries as a gap. Same lockstep and same no-quiet-weakening shape as F6.

## 2026-09-07 — lock-ownership-and-hold-time F9: the gate re-run rule lived only in remediation mode
`decision=applied commit=f5e3d63 target=.agents/skills/uvm-build/SKILL.md`
- **Rationale:** a forward build that adds a constraint invalidates predecessor gate *setups* by the same mechanism, and `next_phase.py` never re-runs a gate, so a red `done` phase ships green. Moved to Step 4 for both paths. **Went past the finding's own fix**, which said "every `done` phase" and would have missed the pending direction its own "seen again" note recorded; the applied text sweeps every phase and separates retune-and-stay-done from reopen.

## 2026-09-07 — lock-ownership-and-hold-time F21: the re-run rule selected gates by depends_on
`decision=applied commit=eda1e51 target=.agents/skills/uvm-build/SKILL.md`
- **Rationale:** `depends_on` encodes build order, not assertion overlap; in a one-script repository every phase edits the same function, and following the letter re-ran one gate of six. Restated by surface — every `done` phase whose gate exercises a function the edit touched, normally all of them. Complements F9 above rather than duplicating it: F9 covers when to sweep, this covers what to sweep.

## 2026-09-07 — lock-acquire-retake F1: the fix-criteria rule was absolute, its rationale conditional
`decision=applied commit=d782bbd target=.agents/skills/uvm-feature/SKILL.md`
- **Rationale:** **rejected the finding's own fix**, which asked to loosen the prohibition once a landed cycle has measured the mechanism. Safety §4 prefers a clarification to a new rule, and the prohibition is a quality rule as much as an epistemic one — criteria phrased as behavior let the implementation vary. Clarified instead what it never forbade: a WHEN clause naming the triggering state is not a diagnosis, and a measured mechanism belongs in the *Checked by* gate. The cycle that filed this had in fact complied.

## 2026-09-07 — lock-ownership-and-hold-time F1: `shaped` was read as fully settled
`decision=applied commit=8329c01 target=.agents/skills/uvm-feature/SKILL.md`
- **Rationale:** the branch says "do not re-litigate, adopt as written", and a seed had explicitly parked two decisions for promotion, one of them a scope difference (a guard in the dispatch tail versus a comment). Named the distinction rather than adding a step: re-litigation reopens what the seed decided; a parked decision is what it left open. The optional `templates/ISSUE.md` half of the recommendation was not taken — marking parked decisions in the template is a separate change and nothing yet shows prose is failing to carry them.

## 2026-09-07 — lock-ownership-and-hold-time F2: a GOAL's *Checked by* clauses were never executed
`decision=applied commit=a5fa08e target=.agents/skills/uvm-feature/SKILL.md`
- **Rationale:** `/uvm-feature` writes these commands and `/uvm-plan` Step 6 was the first step required to run any, so a clause that can never pass (`git grep -c flock` "returning 0", against a file whose comment names `flock`) reaches `TECH.md` as a permanently red gate walking `--record-attempt` toward the circuit breaker with correct code underneath. Placed in Step 5 rather than as a new step, so the coherence check that already re-reads the GOAL also runs its commands.

## 2026-09-07 — lock-acquire-retake F2: two Step 3 rules claimed the fan-out with no precedence
`decision=applied commit=4592565 target=.agents/skills/uvm-plan/SKILL.md`
- **Rationale:** the diagnostic exception ends "when they disagree with the GOAL, the GOAL wins" and the high-blast-radius exception written immediately after it had no tie-break, so a GOAL resolving the opposite left both binding. Deliberately did **not** mirror the diagnostic tie-break: this exception exists because a human can under-scope an edit in this region, so a GOAL narrows the fan-out's breadth but cannot eliminate it, and the narrowing is recorded in `PLAN.md`.

## 2026-09-07 — lock-acquire-retake F3: the inert-gate rule had no case for a regression gate
`decision=applied commit=7b0de8b target=.agents/skills/uvm-plan/SKILL.md`
- **Rationale:** a collateral criterion demands behavior *not* change, so its gate is green before and after by construction and the "exits 0 today is inert" rule read literally would have discarded four correct gates. Scoped the exemption to the gate and explicitly not to the phase — a phase carrying only regression gates still owes one post-condition red today — so the rule cannot become a route around falsifiability.

## 2026-09-07 — lock-ownership-and-hold-time F8: the invariant gate was never checked against the checklists
`decision=applied commit=8b9476d target=.agents/skills/uvm-plan/SKILL.md`
- **Rationale:** a plan asserted load-time placement was rejected because it would kill `uvm help` on a misconfigured node, then a checklist two sections later specified exactly that; the gate and the phases are written in separate passes and nothing reconciled them. One question per gate bullet — which phase would violate this? — placed at the end of Step 6 where the phases exist, not in Step 5 which runs before them.

## 2026-09-07 — lock-ownership-and-hold-time F3: the gate style reference documented the wrong default
`decision=applied commit=71f1af2 target=.agents/factory/templates/TECH.md`
- **Rationale:** every gate in this project is a multi-line sandbox drive, which double-quoted style cannot express, and the previous cycle's gates round-tripped into near-unreadable folded scalars. Made `|` the documented default for anything longer than one command and kept the escaping warning scoped to the one-liner where it is true. Verified `|` round-trips through `next_phase.py`'s PyYAML with heredocs intact.

## 2026-09-07 — lock-ownership-and-hold-time F16: a drive dying at the wrapper reported nothing
`decision=applied commit=c70d818 target=.agents/factory/templates/TECH.md`
- **Rationale:** the conventions require every *assertion* to print a `FAIL:` line and said nothing about the drive command, so a call that died on a refusal aborted under `set -e` and exited 1 having printed nothing — the diagnostic sat unread in the sandbox and was deleted with it. Stated where gates are authored rather than in the skill, because the idiom that causes it (redirect stderr, assert later) is a template convention.

## 2026-09-07 — lock-ownership-and-hold-time F10: a paragraph cancelled itself and misdirected `pre-release`
`decision=applied commit=8ceb60a target=.agents/skills/uvm-release/SKILL.md`
- **Rationale:** **a deletion, net −3 lines.** "There are no tags yet" died 1h46m after `33a91fb` wrote it and five tagged releases have shipped; worse than inert, it pointed a `pre-release` run at suffix-free tags against the STOP a few lines above. Its live rules already sat in the *Version* bullet, so only the non-duplicated rationale — bare `X.Y.Z` is what makes `git tag -l` and `--version` agree — was grafted there.

## 2026-09-07 — lock-ownership-and-hold-time F11: three scope claims the sibling-intake commit falsified
`decision=applied commit=1592ac7 target=.agents/factory/methodology.md`
- **Rationale:** `bced44f` gave the operational siblings a place to file a finding and left three sentences asserting the old scope, one inside its own diff. Heeded the finding's warning against the obvious rewrite: `methodology.md` names `META.md` as the exception for `/uvm-roadmap` and `/uvm-release` *only*, since a blanket "all three may append" would license the meta-on-meta recursion that document forbids. Corrected the finding's stated reason for the third claim — `/uvm-build` writes `.agents/` not by skill instruction but through `AGENTS.md`'s same-commit rule, as `c44b4c0` did to `invariants.md`.

## 2026-09-07 — lock-ownership-and-hold-time F15: no skill owned a mid-cycle GOAL amendment
`decision=applied commit=c9b72a4 target=.agents/factory/methodology.md`
- **Rationale:** **scoped down from the finding's fix**, which proposed a `/uvm-feature` mode accepting a feature branch. Documented the route the maintainer has now taken twice — one commit on the cycle's branch carrying the whole artifact set, naming the circuit-breakers it crosses and supplying `AGENTS.md`'s sign-off where it overturns an invariant — without building the mode. `harness-log.md` F3 already carried this as `deferred`, "real and unfixed"; documenting what happens is the half that stops it being undocumented, and it does not foreclose the mode.

## 2026-09-07 — lock-ownership-and-hold-time F26: the drift sweep had no trigger for `--all`
`decision=applied commit=a0e4553 target=.agents/skills/uvm-roadmap/SKILL.md`
- **Rationale:** Step 5 bound the sweep to files the retirement already edits, which is uncorrelated with where citations rot — they rot wherever the shipped cycle moved code. Three seeds predating the same growth were never opened, and this repository already paid for it once at cycle 2's F8. Gave `--all` a stated trigger cheap enough to always run, and said the reason is seeds the sweep never opens.

## 2026-09-07 — lock-ownership-and-hold-time F22: a new primitive was designed before it was checked
`decision=applied commit=eabdf83 target=.agents/skills/uvm-build/SKILL.md`
- **Rationale:** a design turned on `rename(2)` being atomic and exclusive — true of the syscall, irrelevant to a script that can only call `mv`, and contradicted by `uvm_point_current`'s documented fallback eleven lines from code read that session. Step 2 points at the region a phase *edits*, never at the region its new mechanism depends on. One grep, in Step 3, before the design rather than after the approval.

## 2026-09-07 — lock-ownership-and-hold-time F23: a mid-build design fork had no route to a human
`decision=applied commit=9a50895 target=.agents/skills/uvm-build/SKILL.md`
- **Rationale:** Step 3 modelled divergence as a free amendment or a contract violation, and a decision that is the maintainer's but does not touch the GOAL fell between them, landing as unchecked boxes in `TECH.md` describing work nobody had agreed to. Gave the middle case the shape `GOAL.md` already uses for a clarification — options, blast radius, ask, dated line — so a reversal has somewhere to attach.

## 2026-09-07 — lock-ownership-and-hold-time F24: a later cycle undercutting a deferral had no rule
`decision=applied commit=a90b6f2 target=.agents/skills/uvm-review/SKILL.md`
- **Rationale:** the correction machinery is scoped to a finding a later cycle overturned; a *disposition* is not a finding, and a deferral a human cleared is not the agent's to reopen. The applied rule routes rather than answers: report the newly-measured part under a new id, correct against the disposition, hand keep-or-reopen to the human as a gate item. Sits beside the deferral exception `cd995b4` added, which is the thing it qualifies.

## 2026-09-07 — lock-ownership-and-hold-time F12: the applier has never said no
`decision=applied commit=45f624c target=.agents/skills/uvm-harness/SKILL.md`
- **Rationale:** re-measured today rather than trusting the finding's figures: 4429 lines added under `.agents/` against 174 deleted, 63 applied against **0** rejected, and Safety §6's previously-rejected branch never once written to. Every run now owes a rejection or a deletion candidate, or a plain statement it found neither. Tightens rather than loosens, so Safety §3 does not bite — but it edits the applier's own procedure, so it went in front of the maintainer regardless. This run supplied both: F10 is a deletion, F13 the first recorded rejection.

## 2026-09-07 — lock-ownership-and-hold-time F13: ledger entries ran 3x the documented format
`decision=applied commit=8d888a0 target=.agents/factory/harness-log.md`
- **Rationale:** **split — the backfill of 63 entries is rejected, the forward rule applied.** Measured before acting: 62 of 63 entries are already a single `**Rationale:**` bullet, so the drift is length, not format, and the length is what a later run actually reads — `b8afd6a` spends its nine lines on two cross-referenced commits, an explicit rejection and its ground, and two mis-citations caught before landing. Compressing that to three lines deletes precisely the anti-thrash memory Safety §6 consults, and the finding's own GATE FLAG argues for keeping it ("rejection history has no horizon"). Inverted the fix: the spec was wrong, not the entries — the format block and Step 7's "one-line rationale" now describe what an entry must carry, length earned rather than counted. **The payload complaint is real and stays open:** 624 lines read end to end every run, whose two cheap remedies — pruning and relaxing that read — the GATE FLAG forbids without a typed override that was not requested.

## 2026-09-12 — lock-break-instance-identity F1: the roadmap rewrite stopped at the promoted seed
`decision=applied commit=187436e target=.agents/skills/uvm-feature/SKILL.md`
- **Rationale:** **half the recommendation rejected as a no-op.** It asked for `ROADMAP.md` beside Step 7's `git add issues/{other-slug}.md`; line 230 already stages it whenever promoting, so the gap is real only on the path the finding did not consider — a GOAL that writes into a sibling under the non-goal rule without promoting anything, where the promotion-conditional add never fires. That is what landed, and it is what `ab57630` means by dropping a clerical half that the surrounding rules already carry. Extends `2d5c897` and `56dd89c`, which between them moved the entry's read and its edit into Step 4 for one seed and left the quantifier at one.

## 2026-09-12 — lock-break-instance-identity F2: gate rehearsal contradicted itself on a fix cycle
`decision=applied commit=c38c324 target=.agents/skills/uvm-feature/SKILL.md`
- **Rationale:** the same defect `7b0de8b` fixed in `uvm-plan` Step 6 one cycle earlier, sitting unfixed in the step `a5fa08e` had just given the command-running rule to — a fix cycle's criteria are broken→fixed behavior by Step 4's own instruction, so "a clause that cannot pass is not a criterion" deletes exactly the red states the cycle exists to establish. Sorted by what the criterion claims rather than by which way the gate went, which is the shape `7b0de8b` settled on. Did not carry over that entry's gate-versus-phase scoping: phases do not exist at shaping time.

## 2026-09-12 — lock-break-instance-identity (off-finding): uvm-feature could not run its own Step 5
`decision=applied commit=a65f816 target=.agents/skills/uvm-feature/SKILL.md`
- **Rationale:** not from a `META.md` finding — found re-deriving F2 against the target under Safety §1, which is the check that earns its keep here. `a5fa08e` made Step 5 run every literal *Checked by* command and the `templates/GOAL.md` default for those is a `temp_root.sh` drive, but the grant had no `Bash(.agents/factory/bin/*)`, no `Bash(git grep *)` and no `Bash(bash -n *)`; Step 4's own `git ls-tree main --name-only spec/` was ungranted too, against a present `Bash(git ls-files *)`. Filed here rather than as a finding because `uvm-harness` never writes `META.md` (Safety §5); it reached the maintainer as a Step 3 question and was taken.

## 2026-09-12 — lock-break-instance-identity F3: a brief's recommended idiom reached the design unrun
`decision=applied commit=2ae848a target=.agents/skills/uvm-plan/SKILL.md`
- **Rationale:** measured before writing rather than taking the finding's word: `[[ "/tmp/probe/lock" -ot "/tmp/probe/mark" ]]` with the lock absent returns **true** on bash 3.2.57, the portability floor itself — fail-open in the one state a lock-age guard exists to catch, and two independent briefs recommended it. Step 6 has run every gate since `87473cc`; the artefact copied verbatim into a §1 high-blast-radius function had no equivalent. Placed in Step 4 where the design is recorded, so `PLAN.md` carries what the idiom returned. The build-side twin is `eabdf83`, which greps for a primitive before designing on it — this runs the idiom the brief already wrote.

## 2026-09-12 — lock-break-instance-identity F4: a red gate waiting on an earlier phase had no category
`decision=applied commit=d83498c target=.agents/skills/uvm-plan/SKILL.md`
- **Rationale:** **placed against the finding's own suggestion.** It said "name the third category in Step 6", which reads as the red-requirement paragraph; the miscategorization actually happens in the *how to read a red* paragraph two below, whose list — "a typo, a missing flag, a tool that is not installed" — is where `tests/lock-race.sh: No such file or directory` lands and is condemned. Put beside that list so "the missing tool above" resolves. Seventh revision of this step's gate guidance after `87473cc`, `e935214`, `aa2e143`, `93c951d`, `1ef1614`, `9841158` and `7b0de8b`; compatible with all, and with `7b0de8b` in particular, which exempts a gate and never its phase.

## 2026-09-12 — lock-break-instance-identity F5: the trap list omitted `git grep -c`
`decision=applied commit=6a6923e target=.agents/skills/uvm-plan/SKILL.md`
- **Rationale:** the finding carried no *Recommended fix* line, so the substitute is authored here and was verified first: `git grep -c flock -- bin/uv-manager` prints `bin/uv-manager:1` and `grep -c flock bin/uv-manager` prints `1`, so a census comparing the former to a bare number is false on a correct tree and walks `--record-attempt` toward the breaker. Added beside the existing hazards rather than as a subsection, the standard `9841158` set on this paragraph. The shipped P1 gate in `spec/lock-break-instance-identity/TECH.md` already uses the `grep -c` spelling, so the rule describes what the repository converged on by hand.

## 2026-09-12 — lock-break-instance-identity F8: a researched rate was adopted without re-measurement
`decision=applied commit=95ffa55 target=.agents/skills/uvm-build/SKILL.md`
- **Rationale:** the skill treats `research/` as settled and the gate as the check, which is right for a post-condition and wrong for a distribution — a brief's zero over 3648 ranks has a confidence interval, and the mechanism wedged 1 burst in 40 once it sat beside the other guards. Does not re-litigate `e935214`, which rejected green-proving gates at *plan* time on the ground that it makes planning rehearse the build; at build time the rehearsal is the job. One collateral edit: the paragraph below read "the middle case between those two" across one intervening paragraph and would now read across two, so its poles are named.

## 2026-09-12 — lock-break-instance-identity F6 + F9: "red before the fix" had no form for a retune
`decision=applied commit=c87f346 target=.agents/skills/uvm-build/SKILL.md`
- **Rationale:** one commit, on `61574ec`'s ground — both amend the same rule, and splitting them means the second rewriting the first's sentence. **F9's premise is false and was corrected before landing:** it states Step 4 "describes proving a gate red by copying the repo out of tree", but that recipe is `uvm-plan` Step 6 and `uvm-build` Step 4 carries none, so the partial revert lands as an addition rather than a replacement — the mis-citation class `b8afd6a` and `5d65af7` both record catching in drafts. F6 is the load-bearing half: Step 1's `set_phase.py --verify` and a mid-phase correction both run after the code exists, so the skill directs you into a flow where its own check is structurally unavailable.

## 2026-09-12 — lock-break-instance-identity F7: the trace rendered an intact gate as a collapsed one
`decision=applied commit=09600ca target=.agents/factory/bin/run_verify.py`
- **Rationale:** `2895fc1` built this tool because hand-reflowing a wrapped gate is where a quoting error enters, then printed `{gate!r}`, which escapes every newline to a literal `\n` — the exact rendering of the failure it exists to catch, so a reader trusting the trace concludes the opposite of the truth. Sharper than the finding knew: the gates in this repository are stored as double-quoted YAML scalars carrying `\n` escapes, so the old trace was byte-similar to the unparsed source and could not show that decoding had worked. Verified both states rather than only the fix — an intact gate prints four `|` lines, a collapsed one a single line carrying literal `\n` — with rc=42 passthrough, the empty-gate exit 2, and a real P4 drive green.

## 2026-09-12 — lock-break-instance-identity F10: cycle 1 handed the reviewer the plan's phase structure
`decision=applied commit=179e431 target=.agents/skills/uvm-review/SKILL.md`
- **Rationale:** **a net deletion** — the cycle conditional and one of the two log commands both go. Fifth pass on this channel after `a242486` (the diff), `b5a9826` (the subjects), `f6748ef` (scoped cycles) and `4fe795a` (the flag that no-ops), and it **completes `b5a9826` rather than reverting it**: that entry dropped subjects for the findings they name, which is a cycle ≥ 1 property, while `uvm-build` Step 7 puts `{id}: {phase name}` in every build subject by convention — confirmed live, `[fix] Build lock-break-instance-identity P3: age, pin, verify, remove`. `4fe795a`'s prohibition on re-compressing this into a flag reference is honored; the whole command and its reason both survive. **Strengthens** blind-review integrity, so Safety §3's typed override did not apply, on the reasoning `a242486`, `b5a9826` and `4fe795a` each recorded. Checked for the second site `9228a75` and `f6748ef` were written to stop: Step 1's two `--oneline` calls are the orchestrator's own pre-flight, which reads `TECH.md` and is not blind, so they stay.

## 2026-09-12 — lock-break-instance-identity F12: the completeness sub-pass had no disposal rule
`decision=applied commit=857f642 target=.agents/skills/uvm-review/SKILL.md`
- **Rationale:** the sub-pass reads `PLAN.md` and `TECH.md` by design, which is exactly what makes a correctness aside it returns unusable — promoting one launders plan-informed suspicion into the verdict the blindness exists to protect, dropping it discards a real observation, and Step 5 named neither. Quarantine is the third disposal, and it is the one this cycle invented by hand. Sits with `9a6d0f4`'s correction notes and `a90b6f2`'s routing rule as the third thing `REVIEW.md` records without promoting.

## 2026-09-12 — lock-break-instance-identity F11: a scratchpad copy of `GOAL.md` for blind reviewers
`decision=rejected commit=— target=.agents/skills/uvm-review/SKILL.md`
- **Rationale:** **rejected by the maintainer on the Safety §6 collision, and the second rejection this ledger records after `8d888a0`.** `e2d26c3`, five weeks old, moved `invariants.md` and `review-rubric.md` out of the inline set and kept `GOAL.md` inline in the same breath — "it lives under `spec/`, which the reviewer must not browse" — so the finding proposes precisely what that decision declined. Two things the finding did not know, surfaced at Step 3 and put to the maintainer with them: `e74710d` already sanctions per-reviewer scratchpad subdirectories in this same step, so the machinery exists, and `uvm-review`'s `allowed-tools` grants no `Bash(cp *)`, so the fix is larger than one sentence. The applier's recommendation was to apply; the maintainer's call was that a recent deliberate decision on a §3 guardrail is not reopened on a finding's say-so. **The transcription risk is real and stays open:** an 18 KB contract re-emitted into two prompts is not a copy, and a future run proposing this again should bring evidence that a reviewer graded a corrupted `GOAL.md`, not a restatement of the argument refused here.
