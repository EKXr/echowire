# Design — Morse Codec and Keying Input

## Module layout (`src/lib/morse/`)

| Module | Responsibility | Dependencies |
|---|---|---|
| `codec.ts` | Character map, `encode(text)`, `decode(groups)`, normalization. Pure functions. | none |
| `config.ts` | `TimingConfig` type and defaults mirroring the requirements parameter table; unit↔WPM conversion. | none |
| `timing.ts` | `TimingClassifier`: press/gap state machine turning raw press durations into element, letter, and word events. | `config.ts` |
| `adaptive.ts` | `UnitEstimator`: rolling-window unit estimation for adaptive mode. | `config.ts` |
| `player.ts` | `TonePlayer`: Web Audio tone scheduling for sidetone and full-sequence playback with per-character progress callbacks. | `codec.ts`, `config.ts` |
| `keyer.ts` | Input-source adapters (pointer, spacebar, touch) normalizing raw DOM events into press start/end with `performance.now()` timestamps; handles key repeat, bounce, and focus loss. | `timing.ts` |

React components consume these through hooks in `src/hooks/`; components contain no decoding or classification logic (per tech steering).

## Codec

- `CHAR_TO_MORSE: Readonly<Record<string, string>>` holds the ITU assignments for the supported set; the reverse map is derived once at module load.
- `encode(text: string): { morse: string; dropped: number }` — normalize (uppercase, whitespace collapse), map each character, join letters with a single space and words with ` / ` in the string representation. The string form is for display and storage; internal logic uses token arrays.
- `decode(elements: TimedElement[]): string` is not the codec's job — decoding from *timing* belongs to the classifier. The codec decodes from *committed groups*: `decodeGroup(group: string): string` maps one dot/dash group to a character or `�`, and word assembly is done by the classifier emitting word-boundary events.
- Round-trip property: for any string `s` over the supported set, decoding the groups of `encode(s)` yields the normalization of `s`. This is the primary fast-check property.

## Timing classification (state machine)

States: `IDLE`, `KEY_DOWN`, `GAP` (with two pending deadlines).

```
IDLE --press start--> KEY_DOWN
KEY_DOWN --press end(d)--> GAP        [classify d vs threshold -> dot|dash, append to pending group]
GAP --silence reaches letterGap--> GAP [commit pending group -> character event; feed durations to estimator]
GAP --silence reaches wordGap--> IDLE [emit word-boundary event]
GAP --press start--> KEY_DOWN         [cancel pending gap deadlines]
```

Implementation notes:

- One classifier instance per keyer session. Timers are driven by `setTimeout` scheduled at press end for the letter and word deadlines, cancelled on the next press start. Deadline math uses the *effective* unit snapshot taken at press end, so a mid-gap adaptation never moves a deadline already in flight.
- Classification is a single comparison against `2 × effectiveUnit` — no hysteresis band, matching the spec table. The bounce floor is applied in `keyer.ts` before the classifier sees the press.
- Events emitted: `onElement('dot'|'dash')`, `onLetter(char)`, `onWordBoundary()`, `onPendingChange(group)`. The UI renders pending group and output from these events; the ARIA live region announces `onLetter`.

## Adaptive estimation

`UnitEstimator` keeps the last 24 classified element durations with their classifications. On each letter commit:

1. Collect durations classified as dots. If fewer than 4, return the current effective unit unchanged.
2. Estimate = median of dot durations.
3. Move the effective unit toward the estimate by at most ±10%, then clamp to [40, 240] ms.

Using dots only is deliberate: dash durations vary more between senders (many senders stretch dashes), while dot duration is the most stable personal tempo signal. Reclassification of window elements under a new threshold is *not* performed — the window is a record of what was classified at the time, which keeps the estimator stable and the behavior explainable.

## Playback

`TonePlayer` schedules a full sequence ahead of time on the AudioContext clock:

- Convert text (via `encode`) or a raw group sequence into a list of (startOffset, duration, charIndex) tone segments using the playback unit; gaps are silence segments (intra-letter 1 unit, letter 3 units, word 7 units).
- A single oscillator + gain envelope is re-triggered per segment (gain ramps of ~5 ms to avoid clicks). Progress callbacks fire from a lookahead timer so the UI can highlight the current character.
- `stop()` cancels scheduled segments and silences the gain immediately.
- Sidetone during keying is the same player in manual mode: tone on at press start, off at press end, at the keying tone frequency.

## Error and edge handling

- AudioContext creation is lazy; `player.ensureRunning()` is called from the first user gesture and playback requests await it (requirements 3.3).
- A press still open when the surface unmounts or loses focus is force-ended by `keyer.ts` (requirements 2.7).
- Unknown groups decode to `�` and never throw (requirements 1.4).
