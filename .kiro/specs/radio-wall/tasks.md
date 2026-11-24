# Tasks — The Wire (Public Message Wall)

Execute in order. Requirements references are to this spec's `requirements.md`.

- [ ] 1. Supabase schema and access control
  - [ ] 1.1 Write migration 1: `messages` and `blocklist` tables with check constraints and the channel/created_at index, per the design. (Req 2.1, 3.1, 3.2, 7.1)
  - [ ] 1.2 Write migration 2: enable RLS; public SELECT and anon INSERT policies on `messages`; public SELECT only on `blocklist`. (Design: Access control)
  - [ ] 1.3 Write migration 3: add `messages` to the `supabase_realtime` publication. (Req 4.2)
  - [ ] 1.4 Seed the blocklist with the initial term list via migration or a documented SQL step. (Req 7.1)

- [ ] 2. Backend enforcement trigger
  - [ ] 2.1 Implement the normalization function in SQL (lowercase, repeated-letter collapse, leetspeak mapping) and the BEFORE INSERT trigger enforcing blocklist matching on body and nickname. (Req 5.1–5.3)
  - [ ] 2.2 Extend the trigger with rate limiting: 10-second minimum interval and 30 posts per rolling hour per `client_token`, with stable error codes for both rejection kinds. (Req 6.2)
  - [ ] 2.3 Test at the database level: a blocked term in leetspeak form is rejected; the 31st post in an hour is rejected; a normal post succeeds. (Req 5, 6)

- [ ] 3. Client filter (TypeScript twin)
  - [ ] 3.1 Implement `src/lib/wire/filter.ts` with the same normalization and matching rules as the SQL function, plus blocklist fetch and caching at page load. (Req 5.1, 7.1)
  - [ ] 3.2 Shared fixture tests: one fixture file of (input, expected verdict) cases run against the TypeScript implementation in Vitest; the same fixtures verified against the SQL function in task 2.3. Verdicts must agree on every fixture. (Req 5.3)

- [ ] 4. Identity
  - [ ] 4.1 Implement `src/lib/wire/nickname.ts`: callsign-style nickname generation, local persistence, editing with the 24-character limit. (Req 1.2, 1.3)
  - [ ] 4.2 Generate and persist the anonymous `client_token` on first visit. (Req 6.1)

- [ ] 5. Posting
  - [ ] 5.1 Implement `src/lib/wire/client.ts` (Supabase client from public env vars) and `src/lib/wire/post.ts`: local length validation, filter pre-check, 10-second local throttle, Morse encoding via the codec, insert, error-code mapping to telegraph-style copy, draft preserved on any failure. (Req 3.1–3.4, 6.3)
  - [ ] 5.2 Subscribe to the shared output channel: a composed keyer/scanner message pre-fills the composer in the current channel, posting still requires the explicit POST action. (Req 3.3)

- [ ] 6. Feed and realtime
  - [ ] 6.1 Implement `src/lib/wire/feed.ts`: initial 50 newest-first, load-older cursor pagination, per-channel realtime subscription with unsubscribe on switch, de-duplication by message id against optimistic entries. (Req 2.2, 4.1, 4.2)
  - [ ] 6.2 Implement the degraded path: subscription failure shows the indicator and enables manual refresh. (Req 4.5)
  - [ ] 6.3 Two-browser test on different networks: a post in one appears in the other without reload; switching channels stops the previous channel's stream. (Req 2.2, 4.2)

- [ ] 7. Wire page integration
  - [ ] 7.1 Build the wire page and components: channel tabs, message list showing text and Morse for every message, per-message play control with progress via the codec player, composer with character count, nickname editor. (Req 2, 4.3, 4.4)
  - [ ] 7.2 Accessibility pass: channel tabs and play controls fully keyboard-operable with accessible names; new-message arrivals announced politely without stealing focus; no color-only distinction between channels or states. (Product steering non-negotiables)

- [ ] 8. Seed support
  - [ ] 8.1 Provide the local seed script used by Hook 2 (service role, local only): inserts staggered, filter-passing seed messages across all three channels with `is_seed = true` and `SEED-` nicknames, plus a matching delete-seed script. (Design: Seed data)
