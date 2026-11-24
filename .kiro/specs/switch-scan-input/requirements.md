# Requirements — Single-Switch Scanning Input

Feature: text composition using exactly one switch, via an automatically cycling letter wheel.
Owned modules: `src/lib/scan/`, output via `src/lib/output/`. UI: scan page (`src/app/scan/page.tsx`).

This is an accessibility input method for users who can operate one binary input and nothing else (see product steering). Every requirement below is written against that constraint: if a flow needs a second control, it is a defect.

Requirements use EARS notation.

## 1. The switch

1.1. THE SYSTEM SHALL treat any of the following as the same single switch: spacebar press, Enter press, pointer click anywhere on the scan surface, or touch tap anywhere on the scan surface.

1.2. THE SYSTEM SHALL NOT require any input other than the switch for starting a scan, selecting, composing, correcting, or sending. Pointer position and hover SHALL carry no meaning in this mode.

## 2. Scanning behavior

2.1. WHEN the user activates the switch on the idle scan screen, THE SYSTEM SHALL begin cycling the highlight through the scan items at the configured scan interval.

2.2. THE SYSTEM SHALL organize items as a frequency-ordered letter grid (rows grouped by letter frequency, most frequent first) plus a fixed control row containing at minimum: SPACE, DELETE, CLEAR, and SEND.

2.3. WHILE scanning is in the row phase, THE SYSTEM SHALL cycle the highlight through rows; WHEN the user activates the switch, THE SYSTEM SHALL move to the item phase and cycle through the items of the selected row.

2.4. WHILE scanning is in the item phase, WHEN the user activates the switch, THE SYSTEM SHALL apply the highlighted item (append the letter, or execute the control) and return to the row phase.

2.5. IF a full cycle of the item phase completes with no selection, THEN THE SYSTEM SHALL return to the row phase without changing the composed text.

2.6. IF three full row-phase cycles complete with no selection, THEN THE SYSTEM SHALL pause scanning and display a resume prompt; WHEN the user next activates the switch, THE SYSTEM SHALL resume from the row phase.

2.7. THE SYSTEM SHALL offer a linear scan mode as a user setting, in which the highlight cycles through all letters and controls in one flat sequence and a single activation selects directly.

## 3. Timing and settings

3.1. THE SYSTEM SHALL provide a scan interval setting with a default of 1200 ms and a range of 600–3000 ms, adjustable in 100 ms steps, persisted across sessions.

3.2. WHEN the scan interval setting changes, THE SYSTEM SHALL apply the new interval from the next highlight step onward.

3.3. THE SYSTEM SHALL provide an optional audio tick on each highlight change and an optional spoken-free confirmation sound on selection, each independently mutable, and SHALL respect the global mute settings from the product steering.

## 4. Composition and correction

4.1. WHILE composing, THE SYSTEM SHALL display the composed text prominently and SHALL announce each appended character through an ARIA live region.

4.2. WHEN the user selects DELETE, THE SYSTEM SHALL remove the last character of the composed text; IF the composed text is empty, DELETE SHALL do nothing.

4.3. WHEN the user selects CLEAR, THE SYSTEM SHALL empty the composed text. CLEAR SHALL be placed so that reaching it requires a deliberate row and item selection (it is never adjacent to SEND in the control row order: SPACE, DELETE, SEND, CLEAR).

4.4. THE SYSTEM SHALL impose no length limit on composed text within this mode; limits of any destination (for example the Wire's message limit) are enforced by the destination, not here.

## 5. Shared output channel

5.1. WHEN the user selects SEND, THE SYSTEM SHALL emit the composed text as a message on the shared output channel (`src/lib/output/`) with source tag `scan`, in the same message shape used by keyed input (source tag `keyer`).

5.2. WHEN a message is emitted, THE SYSTEM SHALL clear the composition area and confirm the send in telegraph-style copy.

5.3. IF the composed text is empty, THEN SEND SHALL do nothing and SHALL NOT emit an empty message.

5.4. THE SYSTEM SHALL also offer, on the same screen, a "play as Morse" action reachable by scanning, which plays the composed text using the Morse codec player without emitting a message.
