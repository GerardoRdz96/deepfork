# DeepFork — project context for Claude Code

**DeepFork** is an agent skill with one idea: *don't fork the code, fork the design.* Point it at any open-source repo and it produces a clean explanation of how the repo actually works, a rebuildable behavioral **blueprint**, and a **clean-room** reimplementation in your stack with your changes. Public OSS, **MIT**, repo `GerardoRdz96/deepfork`. Powered by [graphify](https://github.com/safishamsi/graphify).

This file is authoritative project context. Verify claims against the code before acting; do not improvise facts about this repo.

---

## What this repo is

DeepFork is **not** an application — it is a **Claude Code skill distributed as a repo** (works with any agent that reads [skills](https://github.com/anthropics/skills)). The whole product is the skill prompt plus its examples. There is no build, no package, no test suite to run here — the "execution" is an agent following `skills/deepfork/SKILL.md` against a *target* repo.

Repo layout (small by design — 7 tracked files):
- **`skills/deepfork/SKILL.md`** — the skill. The entire pipeline lives here. This is the file you edit to change DeepFork's behavior.
- **`examples/`** — a worked run (`micrograd`).
- **`assets/`** — the banner. `README.md` — the public pitch. `LICENSE` — MIT.
- **`deepfork-out/`** — sample output artifact (the skill writes here at runtime; keep it out of any rebuilt repo).

---

## The pipeline (what SKILL.md encodes — keep these invariants)

A license-gated reverse-engineering pipeline. Every phase's product is a file the user keeps; **all outputs go to `deepfork-out/<target-name>/`**.

- **Phase 0 — License gate (non-negotiable, never skip).** Check the target's license (`gh api repos/<owner>/<repo> --jq .license.spdx_id` or the LICENSE file). **Proceed only for OSI-approved licenses.** No license = all-rights-reserved → stop (an *understanding* report from public code is still OK; **no rebuild**). Copyleft (GPL/AGPL) → warn that a closely-derived rebuild may carry obligations; recommend the rebuild also be open source. Record the verdict in `LICENSE-GATE.md`.
- **Phase 1 — Acquire.** `git clone --depth 1 <target-url> deepfork-out/<target>/source`.
- **Phase 2 — Comprehend (the graph pass).** Prefer `graphify` (`graphify .` is free/local tree-sitter AST; `graphify cluster-only`, `graphify . --wiki`). Fallback if absent: map entry points, manifest, dir tree, top-5 most-imported modules by hand. Then write **`UNDERSTANDING.md`** — load-bearing pieces (graphify's "god nodes"), data flow, the core trick, surprising couplings.
- **Phase 3 — Interrogate.** Answer a rebuilder's questions via `graphify query / path / explain` (or direct reading); append to `UNDERSTANDING.md` § Interrogation. Read real code for anything still `[INFERRED]` the rebuild depends on.
- **Phase 4 — Blueprint.** Write **`BLUEPRINT.md`** — a spec someone could build from **without ever seeing the original source**.
- **Phase 5 — Rebuild (clean-room).** Build from the blueprint, **tests first** for the core mechanisms, milestone by milestone. Ship with **`ATTRIBUTION.md`** ("Design informed by reverse-engineering &lt;original&gt; (&lt;license&gt;). Implementation is original, built clean-room from a behavioral blueprint.").

---

## Standing rules

- **Phase 0 is non-negotiable.** No rebuild of a target without an OSI-approved license; honor copyleft obligations. This is the legal spine of the tool — never let a run skip or soft-pedal it.
- **Clean-room means clean-room.** Phase 5 builds from `BLUEPRINT.md`, **not** from the source tree. The blueprint is the firewall between "learned the design" and "copied the code." Don't paste original source into the rebuild.
- **Attribution always.** Every rebuild ships `ATTRIBUTION.md`.
- **The skill is the product.** Improvements are edits to `skills/deepfork/SKILL.md`. Keep it self-contained and portable (no assumptions beyond a shell, `gh`, optional `graphify`).
- **graphify is preferred, not required.** Always provide the hand-mapping fallback so DeepFork works where graphify isn't installed.

## Keeping this file honest

This file is re-read every turn — keep it accurate. When the pipeline phases or outputs change in `SKILL.md`, update this file in the same change and re-verify against it. Stale context here causes a DeepFork run that skips a safeguard.
