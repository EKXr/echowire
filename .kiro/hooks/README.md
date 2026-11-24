# Agent Hooks — EchoWire

This document specifies the agent hooks planned for this project. It is documentation only: the hooks are described here so they can be configured in Kiro (Hooks panel) with exactly this behavior. No executable hook configuration is committed yet.

## Hook 1: Run property tests on spec save

- **Trigger**: A file under `.kiro/specs/morse-codec/` is saved (`requirements.md`, `design.md`, or `tasks.md`). The same trigger applies, with their unit test suites, to saves under `.kiro/specs/switch-scan-input/` and `.kiro/specs/radio-wall/`.
- **Action**: Run the test suite mapped to that spec (see the spec-to-code mapping in `.kiro/steering/structure.md`):
  - `morse-codec` → `tests/morse/` (unit + fast-check property-based tests)
  - `switch-scan-input` → `tests/scan/`
  - `radio-wall` → `tests/wire/`
- **Why**: The codec's correctness claims (round-trip identity, ±30% timing-noise robustness) are the technical core of the project. A spec edit that silently invalidates them must surface immediately, while the editor still remembers the intent of the change — not at the next manual test run.
- **Behavior on failure**: Report the failing properties and the shrunk counterexamples in the editor. Do not auto-edit the spec or the code; a human (or a deliberately prompted agent task) decides whether the spec or the implementation is wrong.
- **Scope note**: This hook runs tests only. It never commits, pushes, or modifies files.

## Hook 2: Generate demo seed traffic before a presentation build

- **Trigger**: Manual invocation only (run from the Hooks panel before recording the demo video or producing a presentation build). Never on save, never on a schedule.
- **Action**: Generate a seed set of Wire messages and insert them into the Supabase `messages` table across all three channels (SOS Practice, Daily Phrase, Open Wire):
  - Messages are realistic short telegraph-style traffic: callsign-style anonymous nicknames, brief messages, a mix of plain text and messages whose Morse playback is interesting to hear (for example SOS drills and the daily phrase).
  - Timestamps are staggered over the preceding hours so the wall does not look staged at a single instant.
  - Every seeded row is marked as seed data (reserved nickname prefix `SEED-`) so it can be identified and deleted in one query after the presentation.
- **Why**: The demo shows the Wire as a living channel, and judges opening the live URL should not land on an empty wall. Seed data makes the realtime arrival of a genuinely new message visible against existing traffic.
- **Guardrails**: Seed only against the project’s own Supabase instance; refuse to run if the target is not the EchoWire project. Seeded content passes through the same keyword filter as user posts. Delete the seed set after the competition judging period if the wall is kept online.
