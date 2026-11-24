# Tech Steering — EchoWire

Technical conventions for all generated code. Where this document and a feature spec disagree on a parameter value, the spec's parameter table wins for that feature; this document governs everything else.

## Stack

- **Framework**: Next.js (App Router) with TypeScript in strict mode.
- **Styling**: Tailwind CSS. No additional CSS-in-JS or component libraries; build the small set of components by hand so the accessibility behavior is fully under our control.
- **Audio**: Web Audio API directly. No audio files, no Tone.js. One shared `AudioContext`, created lazily and resumed on first user gesture.
- **Backend**: Supabase free tier only — Postgres for Wire messages, Realtime (`postgres_changes`) for live updates, Row Level Security for access control. No custom server code beyond Next.js route handlers if a spec calls for one.
- **Deployment**: Vercel. The app must remain fully functional on free tiers of Vercel and Supabase.
- **Testing**: Vitest for unit tests, **fast-check** for property-based tests.

## Hard constraints

- **No paid APIs. No hosted AI models.** The codec, timing classifier, scanning engine, and content filter are deterministic local code. Do not introduce an API key requirement for any core feature.
- **Client-first.** The keyer, scanner, codec, and practice mode run entirely in the browser and must work with the network disconnected. Only the Wire needs the network.
- **No new runtime dependencies without a note in the relevant spec's design.md.** The dependency list stays short enough to audit.

## Timing parameters

All Morse timing behavior is governed by the parameter table in `.kiro/specs/morse-codec/requirements.md` (defaults, ranges, and the adaptive estimation rule). Code must read timing values from a single typed configuration object (`TimingConfig`) whose defaults mirror that table exactly. Do not hard-code millisecond thresholds at call sites. If a default changes, change the spec table and the config in the same commit.

## Testing requirements

- The codec (`encode`, `decode`, timing classifier) **must** carry property-based tests, at minimum:
  - round-trip identity: for any string over the supported character set, `decode(encode(s)) === s` (after defined normalization);
  - noise robustness: decode still succeeds when every element duration is jittered by up to ±30%;
  - classifier monotonicity: a longer press never classifies as a shorter element.
- Bug fixes in codec or timing code require a regression test that fails without the fix.
- Scanning state machine transitions and the Wire keyword filter get ordinary unit tests; property-based tests are welcome but not required there.
- Run tests before committing changes to `src/lib/morse/`.

## Code conventions

- Pure logic (codec, timing, scanning, filtering) lives in framework-free TypeScript modules under `src/lib/` and must be importable in a test without a DOM.
- React components never implement decoding, classification, or filtering themselves; they call `src/lib/`.
- All user-visible strings come from the copy conventions in `product.md` (telegraph style for system copy). Do not inline ad-hoc phrasing in components for status messages; keep them in a single copy module.
- Time is always milliseconds as a number; durations from `performance.now()` deltas, never `Date.now()` arithmetic in the timing path.
- Handle the unglamorous cases in the same change as the happy path: key repeat on a held spacebar, pointer leaving the key area mid-press, tab losing focus mid-scan, AudioContext suspended.
