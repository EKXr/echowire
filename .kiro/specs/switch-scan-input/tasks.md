# Tasks — Single-Switch Scanning Input

Execute in order. Requirements references are to this spec's `requirements.md`.

- [ ] 1. Scan layout
  - [ ] 1.1 Implement `src/lib/scan/layout.ts`: frequency-ordered letter rows, control row in the order SPACE, DELETE, SEND, CLEAR, and the flat sequence for linear mode. (Req 2.2, 2.7, 4.3)
  - [ ] 1.2 Unit tests: every character in the codec's supported set is reachable exactly once; control row order matches the spec; no row is empty. (Req 2.2, 4.3)

- [ ] 2. Scan controller state machine
  - [ ] 2.1 Implement `src/lib/scan/scanner.ts` with an injectable clock: IDLE / ROW_SCAN / ITEM_SCAN / PAUSED transitions, wrap-around cycling, item-phase timeout back to row phase, three-cycle pause and resume. (Req 2.1–2.6)
  - [ ] 2.2 Unit tests with a fake clock: row selection enters item phase; item timeout returns without changes; three idle row cycles pause; a switch press resumes; interval changes apply on the next step. (Req 2.3–2.6, 3.2)
  - [ ] 2.3 Implement linear scan mode in the controller (flat sequence, single-activation select). (Req 2.7)

- [ ] 3. Composer
  - [ ] 3.1 Implement `src/lib/scan/composer.ts`: append, delete-last (no-op when empty), clear, change events. (Req 4.1, 4.2, 4.3)
  - [ ] 3.2 Unit tests: delete on empty is a no-op; clear empties; event sequence matches applied items. (Req 4.2, 4.3)

- [ ] 4. Settings
  - [ ] 4.1 Add scan settings to the shared settings store: interval (default 1200 ms, 600–3000 ms, 100 ms steps), linear/grouped mode, tick sound on/off, selection sound on/off. (Req 3.1, 3.3)

- [ ] 5. Shared output integration
  - [ ] 5.1 Implement `src/lib/output/` message bus if not already present from keyer work: `emit({ text, source })` and subscription; wire SEND to it with source `scan`; empty composition emits nothing. (Req 5.1–5.3)
  - [ ] 5.2 Add "play as Morse" via the codec `TonePlayer` for the composed text, reachable by scanning. (Req 5.4)

- [ ] 6. Scan page integration
  - [ ] 6.1 Build the scan page and components: grid rendering with non-color highlight cues, composition display, start/pause states, settings controls reachable outside an active scan. (Req 1, 2, 4.1)
  - [ ] 6.2 Normalize all switch sources (spacebar, Enter, click, tap anywhere on the scan surface) into the single controller input; verify pointer position has no effect. (Req 1.1, 1.2)
  - [ ] 6.3 Accessibility pass: ARIA live announcements for appended characters and phase changes per the design's announcement rule, visible focus, full operability with the switch alone, reduced-motion respected. (Req 4.1; product steering non-negotiables)

- [ ] 7. End-to-end manual check
  - [ ] 7.1 Using only the spacebar, compose and send the sentence `HELLO WORLD` from a cold start; record the number of switch presses and elapsed time as a baseline for later tuning. Compose a second sentence using DELETE to correct a deliberate error.
