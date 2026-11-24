# Requirements — The Wire (Public Message Wall)

Feature: an anonymous, no-signup public message wall — the shared "radio channel" of EchoWire.
Owned modules: `src/lib/wire/`, schema in `supabase/migrations/`. UI: wire page (`src/app/wire/page.tsx`).

Requirements use EARS notation.

## 1. Identity

1.1. THE SYSTEM SHALL NOT require an account, password, or any sign-up to read or post.

1.2. WHEN a visitor first opens the Wire, THE SYSTEM SHALL assign an anonymous callsign-style nickname (for example `EW-7K2Q`) and store it locally.

1.3. THE SYSTEM SHALL allow the visitor to edit their nickname (maximum 24 characters) and SHALL persist the edited nickname locally across sessions.

## 2. Channels

2.1. THE SYSTEM SHALL provide exactly three fixed channels: `sos-practice` (SOS Practice), `daily-phrase` (Daily Phrase), and `open-wire` (Open Wire). Users SHALL NOT be able to create channels.

2.2. WHEN the visitor switches channels, THE SYSTEM SHALL show that channel's messages and subscribe to that channel's new messages only.

## 3. Posting

3.1. WHEN the visitor submits a message, THE SYSTEM SHALL store it with: channel, nickname, text body, the Morse encoding of the body (computed with the codec from the `morse-codec` spec), and a server timestamp.

3.2. THE SYSTEM SHALL accept message bodies of 1–280 characters after trimming; IF a body is empty or over the limit, THEN THE SYSTEM SHALL reject the submission locally with a telegraph-style explanation and SHALL NOT send it to the backend.

3.3. WHEN a composed message arrives on the shared output channel (from the keyer or the scanner), THE SYSTEM SHALL offer it as a pre-filled Wire post in the currently selected channel, still requiring an explicit post action by the user.

3.4. IF the backend rejects a post (filter or rate limit), THEN THE SYSTEM SHALL show the reason in telegraph-style copy and SHALL keep the draft text so the user does not lose it.

## 4. Display and playback

4.1. WHEN the Wire page opens, THE SYSTEM SHALL load the 50 most recent messages of the selected channel, newest first, with a way to load older messages in batches of 50.

4.2. WHILE the page is open, WHEN a new message is inserted in the selected channel, THE SYSTEM SHALL prepend it to the list in real time without a page reload.

4.3. THE SYSTEM SHALL display every message in both forms: the text body, and its Morse encoding rendered as dots and dashes.

4.4. WHEN the user activates a message's play control, THE SYSTEM SHALL play that message's Morse encoding as audio using the codec player, and SHALL visually track progress through the message.

4.5. IF realtime updates are unavailable (connection failure), THEN THE SYSTEM SHALL indicate the degraded state and provide a manual refresh that reloads the latest messages.

## 5. Moderation: keyword filter

5.1. WHEN a message is submitted, THE SYSTEM SHALL check the body and nickname against the blocklist (section 7) before the message becomes visible, normalizing case, repeated letters, and common leetspeak substitutions before matching.

5.2. IF a submission matches the blocklist, THEN THE SYSTEM SHALL reject it (the message is never stored as visible content) and inform the user with telegraph-style copy that does not quote the blocked term.

5.3. THE SYSTEM SHALL enforce the filter on the backend as well as in the client, so bypassing the client does not publish blocked content.

## 6. Moderation: anonymous rate limiting

6.1. THE SYSTEM SHALL identify anonymous posters by a random client token generated on first visit and stored locally.

6.2. THE SYSTEM SHALL reject posts from a client token that has posted within the last 10 seconds, and posts beyond 30 per rolling hour per client token, enforcing this on the backend.

6.3. IF a post is rate-limited, THEN THE SYSTEM SHALL tell the user how long to wait before posting again.

## 7. Blocklist management

7.1. THE SYSTEM SHALL keep the blocklist as data (a database table editable without a code deploy), seeded with a small initial list; the client-side check SHALL use a cached copy refreshed at page load.
