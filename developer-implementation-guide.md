# Developer Implementation Guide: Local Community Features

## Purpose

This guide turns the feature plan into an implementation sequence for developers. Implement the features in the release order below and keep each step traceable to its Spec Kit story and acceptance criteria.

All components run on the user's computer or trusted intranet. Local browser-to-application-server requests are allowed. The application must not depend on public hosting, cloud storage, external APIs, hosted model inference, public STUN/TURN, or internet relays. The text filter uses a local keyword/phrase list and requires no model weights, model download, or LFS asset. Install other required runtime dependencies locally before use; do not fetch them at runtime.

## Release order

| Release | Work | Story mapping |
| --- | --- | --- |
| 1 | Community dashboard, polls, announcements, text warning, LAN voice room pilot | US-3, US-4, US-5, US-7, US-11, US-13 |
| 2 | Encrypted community/workspace group messaging | US-14 |
| 3 | Workspaces and shared Markdown documents | US-15 |

Keep existing US-12 one-to-one messaging. Add the dashboard to community management (US-3), optional content tags to discussions/resources (US-4/US-5), polls to community participation, moderation assistance to reporting/moderation, and announcements to notifications. Do not create extra stories for those additions.

## Common implementation rules

1. Implement server-side membership/role checks before adding the UI. Every request that reads or changes private-community data must verify active membership on the server.
2. Reuse the application's configured local database, API client, validation, response envelope, error handling, authentication, and notification patterns.
3. Keep transient toasts separate from persisted in-app notifications.
4. Return explicit validation errors and display accessible, keyboard-operable status/error messages in the UI.
5. Make local-only operation observable: document local dependency and relay setup, ensure the text rules load from the application bundle, and confirm there is no external runtime request or remote model resolution.
6. Keep test implementation consistent with the project test policy. The steps below describe acceptance verification; they do not authorize adding test suites if the project policy requires separate instruction.

## Release 1

### A. Community insights dashboard — US-3 extension

**Initial behavior:** Integrate the community overview into the existing community landing page; do not create a separate dashboard route. On wide screens, show compact insight cards and mini charts in a sidebar alongside the community feed. On narrow screens, place them in a collapsible section below the community header. Members can choose the last 7 days, 30 days (default), 90 days, or all time. All values are computed from the local application's existing records; do not introduce external analytics, tracking pixels, or a persisted analytics warehouse.

**Implementation sequence:**

1. Add optional tags to discussions and resources: 1–5 tags per item, 24 characters maximum per tag, trim whitespace and a leading `#`, and compare duplicates case-insensitively. Preserve a display label while using a normalized value for matching. Include tags in the local keyword/phrase warning.
2. Add a community-scoped insights service and read-only endpoint such as `GET /api/communities/:communityId/insights?range=30d`. Verify current membership server-side on every request. Return only aggregate cards and chart series: current member count; period counts for discussions, comments, resources, likes, and poll ballots; activity by type (daily for 7/30 days, weekly for 90 days, monthly for all-time); top 10 tags counted by distinct content items; content mix for discussions/resources/polls created in the period; and most-engaged discussion/resource links ranked by likes plus comments, with recent items first for ties.
3. Aggregate in the local database using the selected community and range. Do not load all content rows into the browser, expose ballot choices, rank individual members, or combine data across communities. Reuse existing community membership and content permissions for links and tag filters.
4. Build a `CommunityInsightsPanel` with existing design tokens and mount it in the existing community landing page, not as a separate page. On wide layouts, use a right sidebar beside the feed; on narrow layouts, use a collapsible section below the community header. Include compact metric cards, an activity trend chart, top-tags horizontal bar chart with click-to-filter, content-mix chart, and a short list of engaging content. Use responsive cards, clear chart titles/units, useful hover and focus details, skeleton/loading, error, and no-activity states.
5. Make every chart keyboard and screen-reader accessible, with a concise textual summary and a tabular data equivalent. Do not use color as the only way to distinguish activity types. Render charts with the existing local UI stack; use inline SVG/CSS if no chart package is already installed, and do not load chart code from a CDN.
6. Verify date-range boundaries, aggregate counts, tag normalization/counts, tag-click filtering, member access and non-member denial for private communities, aggregate-only poll data, empty/loading/error states, responsive layout, and keyboard/screen-reader access. Keep the dashboard read-only.

### B. Polls — US-4 extension

**Initial behavior:** A community member creates a single-choice poll with a question, 2–6 options, and an optional closing time. Each active member gets one ballot and may change it until close. Polls are non-anonymous in this release. Results appear after the member votes or after closing.

**Implementation sequence:**

1. Add `Poll`, `PollOption` (embedded or separate), and `Ballot` data structures. Add indexes/constraints that prevent duplicate ballots for the same poll/member.
2. Add service methods for create, list/get, vote, change vote, close, and results. Validate that poll and parent community exist and that the caller is an active member. Reject voting after close and option edits after the first vote.
3. Add authenticated API routes and request validation. Apply authorization in the service/controller boundary as well as route middleware; never rely on UI hiding.
4. Add client API methods and a community-scoped UI for creating a poll, voting, changing a vote, and viewing results. Show closed state and close time clearly.
5. Verify duplicate-vote rejection, vote change, close behavior, removed-member access denial, and private-community isolation.

### C. Announcements — US-7 extension

**Initial behavior:** Community owners/moderators publish, edit, pin/unpin, archive, or delete announcements. Active members can view published announcements on a board. Publication creates one in-app notification per eligible member, respects notification preferences, and does not notify the author. No email, push, or external delivery.

**Implementation sequence:**

1. Add an `Announcement` entity with community, author, title, body, status, pinned state, and timestamps. Add a stable sort order for pinned items.
2. Add service methods for create, update, publish, pin/unpin, archive, delete, and member list. Enforce owner/moderator permissions for writes and active membership for reads.
3. Connect publish to the existing notification service. Make notification creation idempotent so request retries do not duplicate notifications. Respect notification settings and exclude the author/former members.
4. Add moderator controls and a member board. Announcements from private communities must never appear in a global feed or to non-members.
5. Verify role failures, duplicate notification retries, notification preference behavior, archive behavior, and access after leaving a community.

### D. Local text rule check — US-11 extension

**Initial behavior:** Run a small deterministic English keyword/phrase check locally on the author's device while composing. On a match, show a private warning and let the author edit or explicitly continue. This is not semantic moderation: it cannot reliably detect meaning, paraphrases, sarcasm, or context. Human reports and admin review remain the moderation authority. Do not include a neural model, model weights, model downloader, LFS asset, external API, or runtime network dependency.

**Implementation sequence:**

1. Implement a small pure JavaScript rule module in the frontend. Store the versioned, editable English keyword/phrase rules in a local source/config file. Normalize case and Unicode, collapse repeated characters, strip selected punctuation between letters, and map a small documented set of common digit/symbol substitutions before matching. Use whole-token or phrase boundaries to reduce accidental substring matches.
2. Rules return only matched rule IDs/categories and severity (`notice` or `strong-warning`); do not invent a semantic confidence score. Keep allow-list/context exceptions explicit and small. Treat quoted text and reclaimed terms as likely false-positive cases; offer the author a way to dismiss the warning and continue.
3. Run the check locally in the compose flow for discussion titles/bodies/tags, resource titles/descriptions/tags, comments, direct messages, community/workspace messages, poll questions/options, and shared-document edits where those features exist. Do not send extra plaintext to the server for checking. For E2EE messages, run the check before encryption on the sending client.
4. On a match, show the category and let the author edit or explicitly continue. Do not automatically block, hold, publish, remove, report, or punish content. A no-match result is not a claim that the text is safe. Existing user-report hiding and human admin review behavior remains unchanged.
5. Keep the routine bounded by existing input length limits and synchronous; show a clear error if the local checker fails, while allowing the existing submit/report flow. No model startup, asynchronous inference, three-second moderation wait, or model-failure fallback is needed.
6. Record no message body, match text, or private key for this local check. If telemetry is later introduced, it requires a separate privacy review and must not include raw user text.
7. Verify normalization, token boundaries, common obfuscation, false positives in benign/quoted text, no-match behavior, checker failure, and confirmation that the checker sends no network request. Do not claim semantic accuracy, comprehensive detection, or protection from harmful content.

**Image-filter scaffold only:** Keep the disabled interface explicitly marked `not_implemented`; do not return a safety score or affect image visibility. Do not add an image model or transfer its assets.
### E. Voice rooms — US-13

**Initial behavior:** Temporary member-only voice rooms support up to 10 participants. The local application server handles room membership/signaling, and a local audio relay/SFU handles group media. Rooms are not recorded. No external STUN/TURN or internet relay is configured.

**Implementation sequence:**

1. Ask the office IT/network administrator whether the required local signaling and media traffic between intranet devices is permitted. Do not scan the network or alter firewall settings without authorization.
2. Select a locally runnable media relay/SFU and document how it starts with the application, required local ports, authentication, health checks, and shutdown. It must run on the same app host or trusted intranet and operate without a cloud account or internet connection.
3. Add a `VoiceRoom` lifecycle service: create/open, join, leave, lock, remove participant, end, and automatic cleanup. Mint short-lived room credentials only after verifying active community membership. Enforce the 10-person cap server-side and reject the 11th join clearly.
4. Add the browser audio UI: explicit microphone permission, participant list, speaking/muted indicator, mute, deafen, and leave controls. Stop microphone capture when leaving or ending the room. Do not record audio.
5. Configure the WebRTC client to use only the local relay and local signaling. Do not include public ICE/STUN/TURN addresses or remote fallback URLs. Show useful connection errors if local media ports are blocked.
6. For office intranet verification, use two authorized devices after IT confirms the test is permitted. Confirm signaling and audio in both directions, then check room access/removal and retry behavior.

**Single-computer demonstration:**

1. Start the local application and local audio relay on one computer. Serve the microphone-enabled UI from `localhost` (or use locally configured HTTPS for an intranet hostname); browsers require a secure context for microphone access.
2. Open two separate browser profiles, sign in as two different test accounts, and join the same room. The UI should show `2/10` participants and identify each stream by its test account.
3. Use the built-in demo audio source to generate a different short tone for each profile. Send each tone through the normal WebRTC audio track and local relay; confirm the other profile receives it and can mute it independently. This verifies real local signaling/media transport without needing two physical people or external audio services.
4. For a real speech check, use headphones. If each profile needs a distinct input, use a locally installed virtual audio device. Avoid routing speaker output back into the microphone because it creates echo.
5. State clearly in the demo that two local profiles prove the local call path, not 10-person capacity. Validate the 10-person limit separately with 10 local browser clients/devices on an approved intranet and reject participant 11.

## Release 2: encrypted group messaging — US-14

**Initial behavior:** Community/workspace group message bodies are end-to-end encrypted. Existing one-to-one messaging remains unchanged. First release supports one active device per account; multi-device synchronization is deferred.

**Implementation sequence:**

1. Before coding, select a maintained, reviewed E2EE protocol/library that supports group membership changes and key rotation. Document its threat model, license, local packaging, interoperability constraints, and recovery behavior. Do not implement a custom cryptographic protocol.
2. Define a client-generated account key and recovery flow. Private keys stay on the device, protected by a user-controlled recovery secret. The server stores neither private keys nor plaintext messages.
3. Define a `CommunityConversation`, encrypted `Message`, and key epoch/rotation records. Store ciphertext plus the minimum routing metadata (conversation/community, sender, timestamps, ciphertext size, delivery/read state).
4. Add server endpoints for conversation membership and encrypted payload storage/retrieval. Check current membership on every operation. The server may route ciphertext but cannot decrypt it.
5. Encrypt message text on the sender before sending. Decrypt only on authorized recipients' clients. The local keyword/phrase warning runs on the sender's client before encryption. A report explicitly submits decrypted content and tells the reporter what moderators will receive.
6. Rotate group keys when someone joins or is removed. A removed/left member cannot read messages sent after removal; do not grant old history to a newly enrolled key by default. If the device and recovery secret are lost, prior messages are unrecoverable.
7. Verify concurrent sends, offline recipient sync, key rotation, removed-member denial, lost-key handling, and that server records/logs contain ciphertext rather than message bodies.

## Release 3: workspaces and shared documents — US-15

**Initial behavior:** A community may have optional workspaces. Each workspace has owner/editor/viewer access and one shared Markdown document. Workspace group chat uses Release 2 encryption. Documents are access-controlled but readable by the local server in this release. Offline collaborative merge, rich text, and E2EE documents are deferred.

**Implementation sequence:**

1. Add `Workspace`, `WorkspaceMembership`, and versioned `CollaborativeDocument` entities. Workspace visibility inherits from the parent community; workspace roles cannot grant access beyond community membership.
2. Add owner/moderator create/archive/delete operations and editor/viewer assignment, restricted to current community members.
3. Implement document read and save APIs using optimistic versioning. Each save includes the version the editor started from. If that version is stale, preserve both versions and return a conflict response; never overwrite the newer version silently.
4. Add a Markdown editor and read-only viewer with author/time/version display, conflict review, and restore-previous-version action. Warn before navigating away from unsaved changes.
5. Link workspace group chat and voice room. Reuse membership authorization, E2EE messaging, and the local voice service; do not duplicate these systems.
6. Archive makes the workspace read-only. Deletion closes its voice room and follows the retention policy documented in the specification. Removing a member revokes access immediately.
7. Verify simultaneous stale saves, conflict preservation/resolution, viewer restrictions, workspace removal, parent private-community isolation, and voice/chat access consistency.

## Completion checklist per story

- Story, requirement, plan decision, data model, API contract, and task IDs agree.
- Membership and role checks are enforced server-side for every private resource and action.
- Local runtime has no external service dependency; the keyword/phrase list is bundled with the client and relay assets load from local paths.
- Errors are explicit and safe; no raw stack traces, plaintext encrypted messages, or private keys are logged.
- UI controls work by keyboard and screen reader; microphone state and encryption state are visible where relevant.
- Acceptance verification covers the positive flow, access denial, failure/reconnect, and deletion/removal lifecycle.
