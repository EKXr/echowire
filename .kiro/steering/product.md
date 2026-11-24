# Product Steering — EchoWire

This document fixes the product decisions for EchoWire. Generated code and copy must conform to it. If a feature request conflicts with this document, the document wins until it is deliberately amended.

## What EchoWire is

EchoWire revives the telegraph and Morse code as a working communication tool, not a novelty. It serves people for whom Morse is, or could be, a practical channel:

1. **Single-switch users.** People with severe motor impairments (for example ALS, high spinal cord injury) who can operate exactly one binary input. EchoWire gives that one input two full text-entry paths: keyed Morse, and a scanning letter wheel that needs nothing but timed single presses.
2. **Emergency and low-bandwidth contexts.** Morse survives in emergency practice because it works at minimal bandwidth and can be received by ear. EchoWire's public wire and audio playback are built for degraded conditions: small payloads, no account, no setup.
3. **Morse learners and radio hobbyists.** The listening practice mode and the public wire give learners real traffic to copy and a reason to build speed.

## Product principles

- **One input is enough.** Every core task — composing, sending, navigating between the keyer, the scanner, the wire, and practice — must be completable with a single switch (plus, at most, the same switch used at different times). If a design needs a second input, redesign it.
- **No accounts, ever.** Identity on the Wire is an anonymous, locally stored nickname. There is no sign-up, no password, no profile. Do not add authentication "for later."
- **The ear is a first-class display.** Anything shown as text that originated as Morse must be playable as audio. Anything decodable must also be listenable.
- **Calm, factual tone about disability.** The product copy describes capabilities and constraints. It does not use pity framing, inspiration framing, or rescue narratives about its users.

## Copy and voice

- **Telegraph style for user-visible system copy**: short uppercase lines, terse wording, `STOP` as a sentence separator where a period would appear in a telegram (for example: `MESSAGE SENT STOP`). Telegraph style applies to headings, status messages, empty states, and confirmations.
- Body text that explains how to do something (instructions, settings labels, help) uses plain sentence case for readability. Accessibility beats style: never sacrifice clarity of an instruction for telegraph flavor.
- No marketing superlatives. State what the thing does.

## Non-negotiables (accessibility)

These are release blockers, not enhancements:

1. **Full keyboard operability.** Every interactive element is reachable and operable by keyboard alone, with a visible focus indicator. The keyer must work from the spacebar without trapping focus.
2. **ARIA correctness.** Decoded output and the scanning highlight are announced through ARIA live regions at a politeness level that does not interrupt the user's own input rhythm. Icon-only controls have accessible names.
3. **No color-only signalling.** Dot vs. dash, correct vs. incorrect, selected vs. scanning — every state distinguished by color must also be distinguished by shape, text, position, or sound.
4. **Timing is user-adjustable.** Scan interval, keying thresholds mode (fixed/adaptive), playback speed, and tone frequency all have settings with sane defaults. A user whose press duration is long must be able to make the decoder accept it.
5. **Reduced motion and sound control.** Respect `prefers-reduced-motion`; scanning and playback can be muted independently of each other.

## Scope guardrails

In scope: Morse codec and keying input, single-switch scanning input, the public Wire (three fixed channels, anonymous), listening practice.

Out of scope: user accounts, private/direct messages, channel creation, file or image transfer, native mobile apps, any paid API or hosted AI model.
