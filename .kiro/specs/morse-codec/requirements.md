# Requirements — Morse Codec and Keying Input

Feature: bidirectional Morse codec (text ↔ Morse) and keying input with timing classification.
Owned modules: `src/lib/morse/`. UI: keyer page (`src/app/page.tsx`).

Requirements use EARS notation. Timing values referenced as "the parameter table" are defined in section 4 and are normative.

## 1. Character codec

1.1. THE SYSTEM SHALL encode and decode the character set A–Z, 0–9, and the punctuation `. , ? ' ! / ( ) & : ; + - _ " $ @` using the ITU Morse assignments.

1.2. WHEN text is encoded, THE SYSTEM SHALL normalize input to uppercase, collapse runs of whitespace to a single word separator, and drop characters outside the supported set while reporting how many were dropped.

1.3. WHEN a Morse sequence is decoded, THE SYSTEM SHALL map element groups separated by letter gaps to characters and word gaps to single spaces, producing uppercase text.

1.4. IF a completed element group matches no supported character, THEN THE SYSTEM SHALL emit the placeholder `�` for that group and continue decoding subsequent groups.

1.5. THE SYSTEM SHALL expose encode and decode as pure functions with no DOM, audio, or framework dependencies.

## 2. Keying input

2.1. THE SYSTEM SHALL accept keying input from three sources: pointer press/release on the on-screen key, the spacebar (keydown/keyup), and touch press/release on the key surface.

2.2. WHILE the spacebar is held, THE SYSTEM SHALL ignore operating-system key-repeat events and treat the hold as one continuous press.

2.3. WHEN a press ends, THE SYSTEM SHALL classify it as a dot or a dash by comparing its duration with the dot/dash threshold from the parameter table, using the currently effective unit (fixed or adapted, per section 5).

2.4. WHEN silence after a press reaches the letter gap, THE SYSTEM SHALL commit the pending element group as one decoded character and append it to the current output.

2.5. WHEN silence after a committed letter reaches the word gap, THE SYSTEM SHALL append one space to the current output.

2.6. IF a press is shorter than the bounce floor in the parameter table, THEN THE SYSTEM SHALL discard it as contact bounce and SHALL NOT alter the pending element group or reset the gap timers.

2.7. IF the input surface loses focus or the pointer leaves the key area mid-press, THEN THE SYSTEM SHALL end the press at that moment and classify it normally, so no press is ever left open.

2.8. WHILE keying is in progress, THE SYSTEM SHALL show the pending element group (dots and dashes as keyed) and the decoded output in real time, and SHALL announce committed characters through an ARIA live region.

2.9. THE SYSTEM SHALL provide a backspace action that removes the last committed character (or the pending element group first, if one is pending) and a clear action that empties the current output, both operable from the keyboard.

## 3. Playback

3.1. WHEN the user requests playback of text or of a Morse sequence, THE SYSTEM SHALL synthesize the tones with the Web Audio API at the configured tone frequency, with element and gap durations taken from the parameter table at the configured playback speed.

3.2. WHILE playback is running, THE SYSTEM SHALL highlight the character currently being sounded and SHALL provide a stop control that silences output immediately.

3.3. IF the AudioContext is suspended (autoplay policy), THEN THE SYSTEM SHALL resume it in response to the user's playback gesture and SHALL NOT report playback as started until audio is actually running.

## 4. Timing parameter table (normative)

All durations in milliseconds. `unit` is the base dit duration. Defaults assume a sender at roughly 15 WPM.

| Parameter | Default | Range / rule | Notes |
|---|---|---|---|
| Base unit (`unitMs`) | 80 ms | 40–240 ms, from speed setting (WPM = 1200 / unitMs) | Fixed-mode reference and adaptive-mode seed. |
| Dot/dash threshold | 2 × unit (160 ms at default) | Press < threshold → dot; press ≥ threshold → dash | Single comparison point; no dead zone. |
| Letter gap | 3 × unit (240 ms at default) | Silence ≥ gap commits the pending letter | Measured from press release. |
| Word gap | 7 × unit (560 ms at default) | Silence ≥ gap commits letter (if pending) and appends one space | Measured from press release; letter commit happens first at its own gap. |
| Bounce floor | 20 ms | Presses shorter than this are discarded | See requirement 2.6. |
| Tone frequency | 600 Hz | 400–900 Hz, user setting | Applies to keying sidetone and playback. |
| Playback speed | 15 WPM (unit 80 ms) | 5–30 WPM, user setting | Independent of the sender's keying unit. |
| Adaptive window | Last 24 classified elements | — | See section 5. |
| Adaptive step limit | ±10% per committed letter | Adapted unit clamped to 40–240 ms | Prevents oscillation. |

## 5. Adaptive threshold estimation

5.1. THE SYSTEM SHALL offer two threshold modes, fixed and adaptive, with fixed as the default; the mode is a user setting.

5.2. WHILE adaptive mode is active, THE SYSTEM SHALL maintain a rolling window of the last 24 classified element durations and, after each committed letter, re-estimate the sender's unit as the median duration of the elements currently classified as dots in that window.

5.3. IF the window contains fewer than 4 dots, THEN THE SYSTEM SHALL keep the configured base unit as the effective unit.

5.4. WHEN the estimate changes, THE SYSTEM SHALL move the effective unit toward the estimate by at most the adaptive step limit per committed letter, clamped to the unit range in the parameter table, and SHALL derive the dot/dash threshold and both gaps from the effective unit using the table's multipliers.

5.5. THE SYSTEM SHALL expose the currently effective unit in the settings area so the sender can see what the decoder believes their speed to be.

## 6. Settings

6.1. THE SYSTEM SHALL persist threshold mode, base unit (or WPM), playback speed, and tone frequency across sessions in local storage.

6.2. WHEN any timing setting changes, THE SYSTEM SHALL apply it from the next press or playback onward and SHALL NOT retroactively reclassify already committed output.
