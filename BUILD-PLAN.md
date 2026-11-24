# EchoWire — Build Plan

Competition: Kiroween 2026, Resurrection track.
Submission deadline: **2026-12-04 14:00 PT** (= 2026-12-05 06:00 Asia/Shanghai). No extensions, late submissions not accepted.
Plan window: 2026-11-24 to 2026-12-04 (11 calendar days; Day 11 is submission day with buffer).

Hard external checkpoints:

- **Kiro credits request** must be submitted well before its own deadline of 2026-12-01 12:00 PT, and approval can take up to 48 business hours. Submit on Day 1.
- **Devpost registration** is open until the submission deadline, but the draft submission should exist by Day 8 so the final day is not spent on forms.

## Schedule

| Day | Date | Milestone | Notes / risks |
|---|---|---|---|
| 1 | 11-24 | Repository scaffold committed: LICENSE, README, steering docs, three specs, hooks doc, this plan. Devpost account registered and hackathon joined. Kiro credits request submitted. | Credits approval latency is out of our control — everything else on Day 1 must not wait for it. Free-tier 50 credits are enough for spec work. |
| 2 | 11-25 | Specs finalized (requirements/design/tasks reviewed end to end). Kiro generates codec core: character mapping, encode, decode. Property-based round-trip test written and passing. | If a spec ambiguity survives Day 2 it will be re-litigated in code all week. Freeze specs at end of day; changes after this go through the spec file first. |
| 3 | 11-26 | Keying input: press capture (mouse / spacebar / touch), timing state machine, letter and word commit. Adaptive threshold estimator per spec parameter table. | **Primary risk: timing "feel".** Classification thresholds that look right on paper can feel wrong in the hand. Start hands-on tuning today, not later. |
| 4 | 11-27 | Playback: Web Audio tone player, text-to-Morse playback with speed control. Keyer page assembled end to end (key in → decoded text → play back). | AudioContext must resume after a user gesture; handle this on first interaction or playback silently fails in some browsers. |
| 5 | 11-28 | Switch-scan input mode: scanning state machine, adjustable interval, compose / delete / clear, shared output channel with keyer. | Scan interval defaults will be wrong for someone; ship the settings control with the feature, not after it. |
| 6 | 11-29 | The Wire: Supabase schema and RLS policies, posting with anonymous nickname, realtime subscription, dual text + Morse display with per-message playback. | Realtime on the free tier is generally reliable, but test with two browsers on different networks, not just two tabs. |
| 7 | 11-30 | Listening practice mode (play, answer, score). Wire moderation: keyword filter and anonymous rate limiting. Seed traffic generator (hook) run to populate demo data. | Keyword list is a blunt instrument; keep it small and log rejections so false positives are visible. |
| 8 | 12-01 | Integration day: all four features reachable from one navigation, accessibility pass (full keyboard operation, ARIA live regions for decoded output and scan highlight, no color-only signalling). Devpost draft submission created with screenshots and write-up outline. | Accessibility is a scoring-relevant product claim for this project, not polish. If it slips, cut practice-mode scope before cutting accessibility. |
| 9 | 12-02 | **Buffer day.** Deploy to Vercel; verify the production URL end to end (keyer, scan, wire from a second device). Timing-feel tuning round 2 against real usage. Fix only what the deployed build reveals. | This buffer absorbs the timing-tuning risk. If Day 3–8 went cleanly, spend it on demo rehearsal instead of new features. No new features after today. |
| 10 | 12-03 | Demo video: script, record (single-switch scene first, practice scene, live Wire scene), edit to under 3 minutes, upload to YouTube as public. Finalize all English submission materials: Devpost description, Kiro usage write-up (specs / hooks / steering / property-based testing), test instructions for judges. | Video must show the app actually running on a real device. Leave editing headroom — do not record on Day 11. |
| 11 | 12-04 | Submission day: final production deploy in the morning (Beijing), submit on Devpost, then verify the submitted entry (repo link, video link, live URL all open in a private window). Hard stop: submit by 22:00 Beijing, eight hours ahead of the deadline. | The deadline is 06:00 Beijing on 12-05. Submitting on Beijing evening of 12-04 keeps a full night of margin for a failed upload or a broken link. |

## Risk register

| Risk | Where it bites | Mitigation |
|---|---|---|
| Timing classification feels wrong in real hands | Days 3–4, discovered again Day 9 | Parameter table pinned in the codec spec; adaptive estimator; two scheduled tuning rounds (Day 3, Day 9 buffer). |
| Kiro credits approval is slow | Days 1–4 | Request on Day 1; spec and steering work (low credit burn) front-loaded; free-tier and already-available credits cover the codec phase. |
| Supabase Realtime misbehaves on judge networks | Demo and judging | Wire degrades gracefully: a manual refresh path and polling fallback are acceptable; the other three features are fully local and unaffected. |
| Scope creep into accounts / private messaging | Any day | Out of scope is written in README and product steering. New ideas go to a parking list, not the spec. |
| Video slips to the last day | Days 10–11 | Recording is scheduled on Day 10 with Day 11 reserved for submission only. |
