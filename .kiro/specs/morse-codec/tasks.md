# Tasks — Morse Codec and Keying Input

Execute in order. Each task is done when its acceptance check passes. Requirements references are to this spec's `requirements.md`.

- [ ] 1. Timing configuration module
  - [ ] 1.1 Implement `src/lib/morse/config.ts`: `TimingConfig` with defaults exactly matching the requirements parameter table (unit 80 ms, threshold multiplier 2, letter gap 3×, word gap 7×, bounce floor 20 ms, tone 600 Hz, playback 15 WPM), plus unit↔WPM conversion helpers. (Req 4, 6.1)
  - [ ] 1.2 Add a settings store (localStorage-backed) for threshold mode, base unit, playback speed, and tone frequency. (Req 6.1, 6.2)

- [ ] 2. Character codec
  - [ ] 2.1 Implement `src/lib/morse/codec.ts`: ITU character map for the full supported set, `encode` with normalization and dropped-character count, `decodeGroup` with `�` fallback. (Req 1.1–1.5)
  - [ ] 2.2 Unit tests: every supported character encodes to its ITU assignment; representative punctuation; normalization cases (lowercase, extra whitespace, unsupported characters). (Req 1.2, 1.4)

- [ ] 3. Property-based tests for the codec (fast-check)
  - [ ] 3.1 **Round-trip identity**: for arbitrary strings over the supported character set, decoding the encoded groups reproduces the normalized string. (Req 1.3)
  - [ ] 3.2 **Noise robustness**: generate a valid timed element stream for a random string, jitter every element and gap duration by a uniform factor in ±30%, run it through the timing classifier (fixed mode, unit set to the stream's unit), and assert the decoded text equals the original. (Req 2.3–2.5)
  - [ ] 3.3 **Classifier monotonicity**: for any two press durations d1 < d2 with the same config, if d2 classifies as a dot then d1 classifies as a dot. (Req 2.3)
  - [ ] 3.4 **Gap invariants**: streams with gaps below the letter gap never commit a letter early; a gap at or beyond the word gap produces exactly one word boundary. (Req 2.4, 2.5)

- [ ] 4. Timing classifier
  - [ ] 4.1 Implement `src/lib/morse/timing.ts` state machine (IDLE / KEY_DOWN / GAP) emitting element, letter, word-boundary, and pending-change events per the design. (Req 2.3–2.5)
  - [ ] 4.2 Unit tests with a fake clock: dot vs. dash at threshold boundary (threshold − 1 ms, threshold, threshold + 1 ms), letter commit timing, word commit timing, press during GAP cancels deadlines. (Req 2.3–2.5)

- [ ] 5. Adaptive estimator
  - [ ] 5.1 Implement `src/lib/morse/adaptive.ts` rolling-window estimator (window 24, dot-median, <4 dots keeps current unit, ±10% step, clamp 40–240 ms). (Req 5.2–5.4)
  - [ ] 5.2 Unit tests: a synthetic sender at 120 ms unit converges from the 80 ms default within a bounded number of letters; a sender who speeds up mid-message is tracked without oscillation; effective unit is exposed for display. (Req 5.2–5.5)

- [ ] 6. Input adapters
  - [ ] 6.1 Implement `src/lib/morse/keyer.ts` for pointer, spacebar, and touch: press start/end with `performance.now()`, key-repeat suppression, bounce-floor filtering, force-end on blur / pointer-leave / unmount. (Req 2.1, 2.2, 2.6, 2.7)
  - [ ] 6.2 Unit tests: held spacebar with repeat events yields one press; a 10 ms press is discarded; blur mid-press ends and classifies the press. (Req 2.2, 2.6, 2.7)

- [ ] 7. Playback
  - [ ] 7.1 Implement `src/lib/morse/player.ts`: lazy AudioContext, sidetone (manual on/off), full-sequence scheduling with per-character progress callbacks, immediate stop, click-free gain envelopes. (Req 3.1–3.3)
  - [ ] 7.2 Manual verification checklist: playback resumes correctly after autoplay suspension; stop silences within one element; highlight tracks the sounded character. (Req 3.2, 3.3)

- [ ] 8. Keyer page integration
  - [ ] 8.1 Build the keyer page and components: key surface, pending group display, decoded output, backspace/clear, text-to-Morse playback entry point, settings controls (mode, speed, tone). (Req 2.8, 2.9, 6)
  - [ ] 8.2 Accessibility pass on the page: full keyboard operation, visible focus, ARIA live announcements of committed characters, no color-only distinction between dot and dash indicators. (Req 2.8; product steering non-negotiables)

- [ ] 9. Spec-compliance sweep
  - [ ] 9.1 Walk every numbered requirement in `requirements.md` against the implementation and tests; file gaps as new tasks here rather than fixing silently.
