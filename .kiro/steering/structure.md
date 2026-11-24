# Structure Steering — EchoWire

Repository layout and the mapping between specs and code. New files go where this document says; if a needed file fits nowhere, amend this document in the same change.

## Repository layout

```
/
├── .kiro/
│   ├── steering/            # product.md, tech.md, structure.md (this file)
│   ├── specs/               # one directory per feature, see mapping below
│   └── hooks/               # agent hook documentation
├── src/
│   ├── app/                 # Next.js App Router routes
│   │   ├── page.tsx         # Keyer (home): keying input + text-to-Morse playback
│   │   ├── scan/page.tsx    # Single-switch scanning input
│   │   ├── wire/page.tsx    # The Wire (public message wall)
│   │   └── practice/page.tsx# Listening practice
│   ├── components/          # UI components, organized by feature
│   │   ├── keyer/
│   │   ├── scan/
│   │   ├── wire/
│   │   └── practice/
│   ├── lib/                 # Framework-free logic (unit/property testable)
│   │   ├── morse/           # codec, timing classifier, player
│   │   ├── scan/            # scanning state machine
│   │   ├── wire/            # Supabase client, keyword filter, rate limiting
│   │   ├── output/          # shared output channel (composed-message bus)
│   │   └── copy.ts          # user-visible system copy (telegraph style)
│   └── hooks/               # React hooks wrapping lib modules for components
├── tests/
│   ├── morse/               # codec + timing tests, incl. property-based
│   ├── scan/
│   └── wire/
├── supabase/
│   └── migrations/          # Wire schema and RLS policies, in apply order
├── public/
├── BUILD-PLAN.md
├── README.md
└── LICENSE
```

## Spec-to-code mapping

Each spec directory owns one feature area. Code generated from a spec lands in the matching modules, and tests for it land in the matching test directory.

| Spec (`.kiro/specs/`) | Logic (`src/lib/`) | UI (`src/app/`, `src/components/`) | Tests (`tests/`) |
|---|---|---|---|
| `morse-codec` | `lib/morse/` (codec, timing, player) | `app/page.tsx`, `components/keyer/` | `tests/morse/` |
| `switch-scan-input` | `lib/scan/`, sends via `lib/output/` | `app/scan/page.tsx`, `components/scan/` | `tests/scan/` |
| `radio-wall` | `lib/wire/`, schema in `supabase/migrations/` | `app/wire/page.tsx`, `components/wire/` | `tests/wire/` |

Listening practice has no separate spec: it is a consumer of `lib/morse/` (playback + codec) and lives in `app/practice/` and `components/practice/`, with any tests under `tests/morse/`.

## Cross-cutting rules

- **Shared output channel.** Both input methods (keyer and scanner) emit composed messages through `src/lib/output/`. The Wire composer and any future consumer subscribe to that channel; input components never import Wire code directly.
- **Settings live in one place.** Timing, scan, audio, and display settings are read through a single settings module backed by `localStorage`. Components do not read `localStorage` directly.
- **One migration per change.** Supabase schema changes are new files in `supabase/migrations/`, never edits to an applied migration.
- **Specs and code move together.** A behavior change starts in the spec's `requirements.md`, then `design.md` if the approach changes, then code and tests. A pull request that changes behavior without touching the spec is incomplete.
