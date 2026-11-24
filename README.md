# EchoWire

EchoWire revives the telegraph and Morse code as a modern web application: a working communication channel for people and situations where a keyboard, a touchscreen, or a network connection is too much to ask for.

## The problem

Two groups still need a lowest-bandwidth human interface, and mainstream software serves neither:

- **People who can operate only a single switch.** For users with severe motor impairments (for example ALS), one reliable binary input — a key, a button, a tap — may be the only available channel. Morse code turns that one channel into full text.
- **People in degraded conditions.** Morse code survives in emergency communication precisely because it works at extremely low bandwidth and can be sent or received by ear. When data networks fail, a tone and a timing pattern still get through.

EchoWire treats Morse not as nostalgia but as a protocol: key it in, scan it in with one switch, listen to it, and post it to a public wire that anyone can hear.

## Features

- **Morse keying input** — Key with mouse, spacebar, or touch. The decoder classifies dots and dashes, commits letters and words from silence gaps, and adapts its timing thresholds to the sender's hand.
- **Two-way codec** — Type text and hear it played as Morse; key Morse and watch it decode to text in real time.
- **Single-switch scanning input** — A letter wheel cycles automatically; one switch press selects, composes a full sentence, and sends it through the same output channel as keyed input.
- **The Wire (public message wall)** — An anonymous, no-signup message wall in three channels (SOS Practice, Daily Phrase, Open Wire). Every message is shown as text and as Morse, and can be played back as audio. New messages arrive in real time.
- **Listening practice** — An ear-training mode that plays characters and words and scores recognition, for learners building receive speed.

Out of scope by design: accounts, private messaging, and native apps.

## Tech stack

- Next.js + Tailwind CSS, deployed on Vercel
- Web Audio API for tone generation and playback (no audio assets)
- Supabase (free tier: Postgres + Realtime) for the message wall
- fast-check for property-based tests of the codec and timing logic

No paid APIs. The core logic is deterministic algorithms, not model calls.

## How Kiro was used

This project was built spec-first with Kiro:

- **Specs** (`.kiro/specs/`) — Three features, each with EARS-format requirements, a design document, and a task list: `morse-codec`, `switch-scan-input`, `radio-wall`. Timing parameters for Morse decoding are pinned in the codec spec, not left to taste.
- **Steering** (`.kiro/steering/`) — Product, tech, and structure documents fix the non-negotiables: telegraph-style copy, accessibility rules, stack conventions, and the requirement that the codec carry property-based tests.
- **Property-based testing** — The codec is verified by generated tests: encode/decode round-trip identity, and decodability under ±30% timing noise.
- **Agent hooks** (`.kiro/hooks/`) — Documented automations: run the matching property tests when a spec is saved, and generate demo seed traffic for the Wire before a presentation build.

## Running locally

```bash
npm install
cp .env.example .env.local   # fill in Supabase URL and anon key
npm run dev
```

The keyer, scanning input, and practice mode work without Supabase credentials; the Wire requires a Supabase project (free tier).

## License

MIT — see [LICENSE](LICENSE).
