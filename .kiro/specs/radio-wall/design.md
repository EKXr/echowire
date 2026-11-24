# Design — The Wire (Public Message Wall)

## Data model (Supabase Postgres)

```sql
create table public.messages (
  id          uuid primary key default gen_random_uuid(),
  channel     text not null check (channel in ('sos-practice', 'daily-phrase', 'open-wire')),
  nickname    text not null check (char_length(nickname) between 1 and 24),
  body        text not null check (char_length(body) between 1 and 280),
  morse       text not null,
  client_token uuid not null,
  is_seed     boolean not null default false,
  created_at  timestamptz not null default now()
);
create index messages_channel_created_idx on public.messages (channel, created_at desc);

create table public.blocklist (
  term        text primary key,          -- stored normalized (lowercase)
  created_at  timestamptz not null default now()
);
```

Notes:

- `morse` is computed by the client with the shared codec at post time and stored, so readers never re-encode and the stored pair (body, morse) can be audited against the codec. Display uses the stored value.
- `client_token` is never rendered in the UI; it exists for rate limiting and abuse response.
- `is_seed` marks rows inserted by the demo-seed hook (`.kiro/hooks/README.md`, Hook 2). Seed rows also carry the reserved nickname prefix `SEED-`; either marker is sufficient to delete the seed set in one statement.

## Access control (RLS)

- RLS enabled on both tables.
- `messages`: public `SELECT` for the `anon` role; `INSERT` for `anon` with the check constraints above; no `UPDATE` or `DELETE` policies (messages are immutable from the client).
- `blocklist`: public `SELECT` for `anon` (the client needs the terms to pre-check); no client writes. Maintained via the Supabase dashboard/SQL by the project owner.

## Backend enforcement (trigger)

Client checks are for user experience; enforcement lives in a `BEFORE INSERT` trigger on `messages`:

1. **Filter**: normalize `new.body` and `new.nickname` (lowercase; collapse 3+ repeated letters to 1; map common leetspeak digits/symbols to letters) and reject the insert if any `blocklist.term` appears as a substring.
2. **Rate limit**: reject if a row with the same `client_token` exists with `created_at > now() - interval '10 seconds'`, or if the count for that token in the last hour is ≥ 30.

Rejections raise an exception with a stable error code the client maps to telegraph-style copy (FILTERED / TOO FAST + wait seconds). The normalization function is shared logic: the SQL version and the TypeScript version in `src/lib/wire/filter.ts` implement the same rules and are tested against the same fixture cases, so client and backend verdicts agree.

## Realtime

- Enable Realtime on `messages` (add the table to the `supabase_realtime` publication in the migration).
- The client opens one subscription per selected channel: `postgres_changes` with `filter: channel=eq.<channel>`, `INSERT` events only. On channel switch, unsubscribe and resubscribe.
- New rows are prepended. Because the posting client also receives its own insert event, the client de-duplicates by message `id` (optimistic local entry is replaced, not duplicated).
- On subscription error or close, the page shows the degraded indicator and enables manual refresh (requirements 4.5). No polling loop in the normal path.

## Client modules (`src/lib/wire/`)

| Module | Responsibility |
|---|---|
| `client.ts` | Supabase client creation from public env vars (URL + anon key). |
| `nickname.ts` | Callsign generation, edit validation (≤ 24 chars), local persistence. |
| `filter.ts` | Blocklist normalization and matching — the TypeScript twin of the SQL function; blocklist fetch + in-memory cache. |
| `post.ts` | Post flow: local validation (length, filter pre-check, 10-second local throttle), insert, error-code mapping, draft preservation on failure. |
| `feed.ts` | Initial page load (50, newest first), load-older pagination by `created_at` cursor, realtime subscription management, de-duplication. |

## Posting flow

1. User composes (typed directly, or pre-filled from the shared output channel per requirements 3.3) and explicitly presses POST.
2. `post.ts` validates locally: length, filter pre-check, local throttle. Failures stop here with copy and keep the draft.
3. Insert via Supabase client. Trigger verdicts return as errors; map the code, show the reason, keep the draft.
4. On success, clear the composer. The message appears via the realtime event (de-duplicated), not via a manual refetch.

## Seed data

Seed insertion (Hook 2) uses the service role from a local script only — never from the browser — and writes rows with `is_seed = true` and `SEED-` nicknames, with staggered `created_at` values. Seeded bodies pass the same blocklist.
