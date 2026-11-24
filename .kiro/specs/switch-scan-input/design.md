# Design — Single-Switch Scanning Input

## Module layout (`src/lib/scan/`)

| Module | Responsibility |
|---|---|
| `layout.ts` | The scan grid definition: frequency-ordered letter rows and the control row. Pure data plus helpers (row lookup, flat sequence for linear mode). |
| `scanner.ts` | `ScanController`: the scanning state machine (phases, highlight position, pause/resume), driven by an injectable clock for testing. |
| `composer.ts` | Composed-text model: append, delete, clear; emits change events for the UI and ARIA announcements. |

The controller knows nothing about the DOM or React. The page component renders from controller state snapshots and forwards the single switch event.

## Scan layout

Letters are grouped into rows by approximate English frequency so the most common letters are reached fastest:

```
Row 1: E T A O I N        (plus, in linear mode, sequence continues)
Row 2: S H R D L U
Row 3: C M W F G Y
Row 4: P B V K J X
Row 5: Q Z 0 1 2 3
Row 6: 4 5 6 7 8 9
Control row: SPACE, DELETE, SEND, CLEAR
```

Control row order is deliberate (requirements 4.3): CLEAR sits after SEND with DELETE between the destructive and constructive actions, so a one-step timing error lands on the least harmful neighbor. The layout is data, not markup, so it can be unit-tested (row contents, control order, every supported codec character reachable exactly once).

## State machine

States: `IDLE`, `ROW_SCAN`, `ITEM_SCAN`, `PAUSED`.

```
IDLE --switch--> ROW_SCAN (highlight row 0)
ROW_SCAN --tick--> ROW_SCAN (next row, wrap)
ROW_SCAN --switch--> ITEM_SCAN (highlight item 0 of selected row)
ROW_SCAN --3 full cycles, no switch--> PAUSED
ITEM_SCAN --tick--> ITEM_SCAN (next item, wrap)
ITEM_SCAN --switch--> apply item --> ROW_SCAN
ITEM_SCAN --1 full cycle, no switch--> ROW_SCAN (no change)
PAUSED --switch--> ROW_SCAN
(any state) --SEND applied--> emit message, clear composer --> ROW_SCAN
```

Implementation notes:

- One timer per active scan, scheduled from the configured interval; the interval is re-read at each step so a settings change applies on the next step (requirements 3.2) without restarting the scan.
- Cycle counting for the pause rule counts completed wraps of the row phase since the last selection or resume.
- All timing goes through an injectable `now/schedule` pair so the state machine is testable with a fake clock — the same discipline as the timing classifier in the codec spec.
- The highlight is rendered with position, shape, and text emphasis, never color alone (product steering). The current highlighted item is exposed via `aria-activedescendant`-equivalent live announcement at a rate that does not flood screen readers: row changes are announced, item changes within a row are announced only in linear mode or on selection. (If field testing shows this is wrong, the spec — not the component — is where the rule changes.)

## Composition and output

`composer.ts` holds the text and is the single source of truth for the composition area. On SEND, the controller calls `outputBus.emit({ text, source: 'scan' })` from `src/lib/output/` — the identical call the keyer makes with `source: 'keyer'`. Consumers (Wire composer, practice) subscribe to the bus; the scan page itself also offers "play as Morse" by handing the composed text to the codec's `TonePlayer`.

## Relationship to the keyer

The scanner and the keyer are alternative front ends over the same codec and output channel. They share: the supported character set (anything the scanner offers must be encodable), the settings store (audio mute), and the output message shape. They share no input-handling code — merging them would compromise the scanner's single-switch purity.
