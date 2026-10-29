# Spec Kit Amendment Plan: Community Collaboration Features

## Objective

Amend the Spec Kit constitution, specification, plan, data model, contracts, and task list to include the features below. Keep the feature behavior and acceptance criteria consistent across every artifact. This is a documentation change plan; describe the work for later implementation without writing application code.

All functionality must run on the user's machine or trusted local network. Local browser-to-application-server communication is allowed. Do not require public hosting, internet access at runtime, cloud storage, external APIs, hosted model inference, hosted signaling, public STUN/TURN servers, or external relays. The text filter uses a bundled keyword/phrase list and requires no model weights, model download, or LFS asset. Install other dependencies locally before use; do not fetch them at runtime.

## Recommended scope and rollout

The complete feature set is too large and security-sensitive for one implementation pass. Keep all feature ideas in the Spec Kit documents, but split them into releases and label later work as deferred so it is not treated as a first-release requirement.

- **First release:** a community insights dashboard; single-choice polls; a simple pinned announcement board; a local English keyword/phrase warning for configured profanity and abusive phrases; and a small LAN voice-room pilot only if the IT approval and two-device test pass.
- **Second release:** encrypted community/workspace group messaging, after choosing and reviewing a maintained protocol/library. Begin with one active device per account; do not add multi-device key synchronization or server-side key recovery.
- **Third release:** workspaces with one shared Markdown document and a linked group chat/voice room. Keep online document edits simple with versioned saves and explicit conflict resolution. Defer offline multi-device merging, rich-text editing, and E2EE document content.
- **Defer image moderation** until text moderation is measured and stable. In the first release, create only a disabled image-filter scaffold/interface marked `not implemented`; it must not claim to inspect images, assign fake safety scores, or affect upload visibility. Do not make image analysis a dependency of the first release.

Copilot should mark the release boundaries and dependencies explicitly in stories, plan, and tasks. Do not expand the first release with optional advanced behavior.

## Constitution updates

Amend the scope and moderation principles to permit the following features while preserving access control, privacy, accessibility, validation, and human moderation safeguards:

- A local keyword/phrase warning is allowed as author assistance. Human moderators and user reports remain available; the warning is not semantic analysis and must not block, punish, or delete content.
- Community group messaging and shared workspaces are allowed in addition to existing one-to-one messages.
- Voice rooms and collaborative documents are community features. They inherit the parent community's visibility and membership checks.
- E2EE is required for community/workspace chat message bodies. Only ciphertext is stored server-side. The client-side keyword/phrase check runs before encryption and may warn the author; the server and administrators cannot inspect or filter encrypted message content unless a user submits a report with the relevant decrypted content.
- Collaborative documents are server-readable in the third release and protected by workspace membership checks. Defer E2EE document encryption to a later amendment after a separate design review; do not promise encrypted document storage in the initial workspace version.
- Local network services are permitted; external services and external network dependencies are prohibited at runtime.

Record a constitution version change and a Sync Impact Report. Update any Spec Kit guidance needed so the new stories remain required inputs to plan and task generation.

## New user stories and required behavior

Keep the total at **15 user stories**: retain existing US-1–US-12 and add only US-13–US-15. Fold related additions into existing stories: the community insights dashboard into US-3; discussion/resource tags into US-4/US-5 as applicable; local content filtering into US-11 (reporting/moderation); polls into US-4 (community participation); and announcements into US-7 (notifications). Add matching functional requirements, data fields/API contracts only where needed, edge cases, assumptions, and measurable success criteria, and keep all additions traceable to their parent story.

### US-11 extension — Local English keyword/phrase warning

Give authors an immediate, private warning when configured English profanity or abusive phrases appear in supported text. Use a small deterministic keyword/phrase matcher. This is not semantic analysis and cannot reliably recognize meaning, paraphrases, sarcasm, or context. Human reports and moderator decisions remain authoritative.

- Apply the check locally on the author’s device to discussion/resource titles, descriptions, and tags, comments, direct messages, community/workspace chat, poll questions/options, and shared-document edits where those compose fields exist. Run it before encryption for E2EE messages. Never send text to a moderation service or add a network request for this check.
- Implement a small pure JavaScript utility and a versioned, editable English keyword/phrase list. Normalize case and Unicode, collapse repeated characters, remove selected punctuation between letters, and map a small documented set of common substitutions. Match whole words or phrases with token boundaries; use a short explicit exception list to reduce obvious false positives.
- Return only matched rule IDs/categories and severity (`notice` or `strong-warning`). Do not calculate or display a semantic confidence score. A match shows an author-only warning with options to edit or explicitly continue. It never blocks, holds, removes, reports, or punishes content. No match is not proof that content is safe.
- Keep the check synchronous and bounded by existing text-length limits. If it fails, show a brief notice and let the existing submission and user-report flow continue. Do not add model files, model downloads, LFS assets, external APIs, moderation queue entities, or a three-second pending workflow.
- Preserve existing user reporting and human admin review. Do not store the checked text or rule matches. Image analysis remains deferred; any placeholder must remain disabled and must not affect image visibility.
- Add acceptance cases for case/Unicode normalization, common obfuscation, token boundaries, benign or quoted text, no-match behavior, checker failure, and confirmation that the check makes no network request. Do not claim semantic understanding or comprehensive detection.

### US-3 extension — Community insights dashboard and tags

Give current community members a polished overview of activity and topics, using only data already stored by the local application. Fold this into US-3; do not add a user story or a separate analytics service.

- Add optional tags to discussions and resources so members can describe topics. Allow 1–5 tags per item, trim whitespace and a leading `#`, normalize case for duplicate detection, limit each tag to 24 characters, and reject empty or duplicate tags. Preserve a display form while matching/searching case-insensitively. Include tags in the local keyword/phrase warning.
- Integrate the insights into the existing community landing page, scoped to that community and visible only to its current members; do not add a separate dashboard route. On wide screens, place compact insight cards and mini charts in a sidebar beside the community feed. On narrow screens, show them in a collapsible section below the community header. Default to the last 30 days; offer 7-day, 30-day, 90-day, and all-time ranges. Every request must verify current membership server-side, including private communities.
- Show summary cards for current member count and selected-period discussions, comments, resources, likes, and poll ballots. Show an activity chart broken down by event type (daily for 7/30 days, weekly for 90 days, monthly for all-time), a horizontal bar chart of the 10 tags used by the most distinct content items, and a content-mix chart comparing discussions, resources, and polls created in the selected period. Include a compact list of the most-engaged discussions/resources, ranked by likes plus comments with recent items first for ties, linking to their existing pages.
- Clicking a tag filters the community's existing discussion/resource lists to that tag. Charts use counts from existing records; do not add tracking pixels, cross-community analytics, a new analytics database, or external chart/telemetry services. Do not rank or expose individual members by activity.
- Use aggregate queries over the selected community and date range rather than loading every content record into the browser. Keep poll participation aggregate-only; never disclose a member's ballot choice from dashboard data.
- Use the existing design tokens and responsive layout. Provide clear labels, hover/focus details, loading/error/empty states, and a keyboard-accessible tabular equivalent for every chart. The dashboard must remain understandable without color alone and work on narrow screens.
- Add acceptance cases for date ranges, correct counts and tag aggregation, tag-click filtering, no-activity empty state, private-community/non-member denial, poll-vote privacy, keyboard/screen-reader access, and responsive layout. Keep the charts read-only; existing content permissions continue to govern linked items.
### US-13 — Community voice rooms

Members can join an ephemeral, live audio room associated with a community or workspace and speak with multiple members simultaneously.

- **Network feasibility gate:** Before enabling voice across office devices, ask the office IT/network administrator whether audio between devices on the intranet is permitted and which local ports/protocols are allowed. Do not scan the office network or change firewall rules without authorization. If approved, run a two-device test on the office intranet using local signaling and audio. Record whether microphone access, signaling, and peer audio succeed. If IT does not allow it, or the approved test cannot establish a connection, do not attempt public STUN/TURN or an internet relay. Keep the feature local and use the single-computer demo described below.
- Any authorized community member may join an open room; a room moderator may mute a participant, remove a participant, lock the room, or end it. Show participant names, speaking/muted state, and a clear local microphone-active indicator. Require an explicit browser microphone permission and provide mute, deafen, leave, and keyboard-accessible controls.
- Rooms are ephemeral. Do not record or persist audio. Persist only room lifecycle metadata needed for current membership, moderation, and cleanup; automatically close empty rooms and expire abandoned rooms.
- Support at least 10 participants per room. Use an audio relay/SFU that runs on the same local app host or trusted intranet to keep each browser from sending separate audio streams to every other participant. The relay must require no cloud account or internet connection; do not configure external STUN/TURN, cloud SFU, or public relay. Verify that firewall/network rules allow the local signaling and media ports before the intranet pilot. If connections fail, show a clear error and do not fall back to the internet.
- For the demonstration, run the app and local audio relay on one computer. Sign in as two distinct test users in separate browser profiles, join the same room, and show the live participant count as `2/10`, speaking/muted indicators, and working mute/leave controls. Use headphones to prevent feedback; if the two sessions need separate audio inputs, use a local virtual audio device or a generated test microphone source. Verify that one session's microphone audio reaches the other and that no external service is contacted. This demonstrates a real local call with two participants; the 10-person capacity is the room limit and must be validated separately with 10 local test clients/devices on an approved LAN.
- Access is checked on room creation and every join. A user removed, banned, or no longer a member is disconnected and cannot rejoin. Private-community rooms are visible only to current members.
- A workspace may link a room to its shared document. Participants can keep viewing/editing the document while speaking. Voice remains available if the document is read-only.
- Reject the eleventh participant with a clear “room is full” message. Do not silently degrade existing calls.
- Add acceptance cases for the two-user same-computer demo, 10-member capacity, full-room rejection, permission denial, reconnect, room lock/end, member removal, private communities, blocked LAN media ports, and simultaneous speakers.

### US-14 — End-to-end encrypted group messaging

Community members can use persistent community/workspace group conversations, with message bodies readable only by authorized member devices. Existing US-12 one-to-one conversations remain available.

- For the first encrypted messaging release, support one active device per account. Use an established, reviewed cryptographic protocol/library; do not create custom cryptographic algorithms. Generate keys on the client. Private keys remain on the user's device and are protected with a local user-controlled recovery phrase/passphrase. Never send or store private keys or plaintext message bodies on the server. Defer multi-device key synchronization.
- Encrypt each message on the sender's device before transmission. The server stores ciphertext plus only necessary routing metadata (community/conversation ID, sender ID, timestamps, ciphertext size, delivery/read state). Explain this metadata to users. Do not log message plaintext, ciphertext keys, or recovery secrets.
- On member addition, provision access for the new member to future messages. On removal/ban, rotate the community conversation key so the removed member cannot decrypt messages sent after removal. Members retain plaintext they already received. A member who leaves and rejoins gets access to new messages only.
- Support multiple simultaneous senders and deterministic chronological ordering. Persist messages locally on the server as ciphertext so disconnected members can sync encrypted messages after reconnect. Do not add offline draft storage in this release.
- If a user loses their device and recovery secret, prior messages are unrecoverable; they may re-enroll with new keys but cannot decrypt old history. Do not provide server-side key recovery.
- The local keyword/phrase warning runs on the sender's client before encryption. The server cannot inspect E2EE content. A user may report a message by explicitly submitting its decrypted text and context to moderators; the report flow must tell the reporter what will be disclosed. Moderators may act on submitted reports but cannot browse private message plaintext.
- Enforce community membership and conversation authorization server-side on every read/write even though payloads are encrypted. Revoke access promptly when membership ends. Direct messages continue to use their existing eligibility rule unless the user story explicitly expands it.
- Add acceptance cases for concurrent sends, reconnect delivery, key rotation, removed member, lost key, changed membership, private community isolation, and user-submitted reports. Defer multi-device enrollment.

### US-4 extension — Community polls

Members can create and vote in polls within communities and workspaces.

- A poll has a question, 2–6 options, creator, community, created/close timestamps, and status. First release polls are single-choice and non-anonymous, with an optional closing time.
- One ballot per active member per poll. Members may change their ballot until the poll closes. The creator cannot edit options after the first ballot; they may close the poll early. Closed polls are read-only and retain final aggregate results.
- Only current members may read, create, or vote in a private community. Users who leave or are removed cannot vote or retrieve poll details afterwards. Polls and result counts never appear in search or notifications to unauthorized users.
- Show aggregate results only after a member votes or the poll closes, whichever happens first. Defer anonymous polls to a later release that can specify and review the privacy guarantees.
- Notify members of new polls using their existing notification preferences. Do not notify the poll creator about their own poll.
- Add acceptance cases for duplicate ballots, changed ballot, poll close, deleted/removed member, aggregate result visibility, and private-community access.

### US-15 — Community workspaces and collaborative documents

Communities may create optional workspaces for projects, study groups, event planning, and shared interests. A workspace combines a shared document, group chat, and linked voice room while leaving the existing community feed as the default experience.

- A community owner or moderator can create, rename, archive, or delete a workspace. A workspace inherits parent community visibility and membership: no workspace can grant access to a non-member of its parent community. Workspace roles are `owner`, `editor`, and `viewer`; community moderators retain moderation authority. Workspace owners can assign editor/viewer roles only among community members.
- Each workspace has one primary shared Markdown document with title, content, version history, author attribution, and last-edited metadata. Editors save versioned updates; viewers can read and comment but cannot change document content. If two editors save against the same prior version, preserve both edits and ask an editor to resolve the conflict. Defer offline multi-device merge and rich-text editing.
- In the first workspace release, documents are visible only to authorized workspace members and protected by the local application's normal access controls. E2EE applies to workspace chat messages, not document content. Disclose that the local server can read document text. Defer E2EE document sync until a separate design review.
- The workspace group chat supports multiple concurrent members, message history, unread indicators, and E2EE using US-14. Workspace chat membership always equals current workspace membership. The group chat is separate from one-to-one DMs.
- A member can start or join a linked US-13 voice room from the workspace. The document remains usable during the call. Voice is ephemeral and is not recorded.
- The editor requires a connection to the local application to save. Warn before navigating away from unsaved edits. Preserve server-saved versions; do not promise offline collaboration in this release.
- Archiving makes the workspace read-only. Deleting a workspace requires owner confirmation and soft-deletes its document/chat for a defined retention period before cleanup; voice sessions close immediately. Deleting the parent community follows the community deletion policy. Removed members lose access immediately to server-hosted workspace data and future keys.
- Add acceptance cases for concurrent edits, stale-version conflict resolution, viewer restrictions, role changes, archived workspace, deletion, private parent community, and voice/document simultaneous use.

### US-7 extension — Community announcement board

Community owners/moderators can publish pinned announcements so members can reliably find important updates without them being buried in discussion or chat.

- Each announcement has a title, body, author, community, created/updated timestamps, optional expiration, pinned state, and status (`draft`, `published`, `archived`). Only community owners/moderators can create, edit, pin/unpin, archive, or delete announcements. Members can read published announcements only while they are active members of that community.
- Show current pinned announcements in a dedicated board and a compact community header area. Order pinned items by moderator-selected order, then other published items newest first. Clearly label edited announcements and their update time.
- On publication, create an in-app notification for current community members according to their existing notification preferences; exclude the author. Editing an announcement does not create a second notification by default. Never send email, push, or external notifications. Do not notify former members or disclose private-community announcement content outside the membership boundary.
- Support an optional “acknowledge” action so moderators can see which active members have read an important notice. Acknowledgment is visible only to community moderators and the acknowledging member. Do not require acknowledgment for ordinary announcements.
- Validate title/body limits, prevent duplicate notifications on retries, rate-limit excessive posting, and record moderator actions for accountability. Removing a member immediately removes board access; rejoining restores access only to announcements still published and not expired.
- Add acceptance cases for permission failures, publish/edit/pin/archive, notification preferences, retry idempotency, expiration, member leave/rejoin, private community isolation, and acknowledgment privacy.

## Spec Kit artifact changes

Apply the following updates in the Spec Kit paradigm; keep every artifact consistent and traceable:

1. **Constitution:** amend scope/moderation/privacy principles and constraints for local keyword/phrase warnings, local LAN voice, E2EE, group chat, workspace collaboration, and moderator announcements. Version the amendment and include a Sync Impact Report.
2. **Specification:** retain US-1–US-12 and add only US-13–US-15. Integrate the community insights dashboard into US-3 and specify optional normalized tags on discussions/resources under US-4/US-5; return dashboard counts as a computed API response without persisting an analytics entity. Integrate the stateless client-side keyword/phrase warning requirements into US-11 without adding a moderation-decision entity; polls and entities (`Poll`, `PollOption`, `Ballot`) go into US-4; the announcement board and entities (`Announcement`, `AnnouncementAcknowledgment`) go into US-7. Add the remaining entities (`VoiceRoom`, `CommunityConversation`, `EncryptedMessage`, `CommunityKeyEpoch`, `Workspace`, `CollaborativeDocument`, `DocumentUpdate`, `WorkspaceMembership`) under US-13–US-15. Reconcile existing one-to-one messaging, moderation, group-chat, notification, and attachment scope statements so they do not contradict the additions.
3. **Plan/research:** keep first-release moderation work bounded to the local keyword/phrase rules and its false-positive behavior, alongside IT-approved LAN voice feasibility and local 10-participant media relay and basic poll/announcement behavior. Do not select or package an ML text model for this release. Before second release, select and review an established group-encryption protocol/library and document its key lifecycle. Before third release, document versioned Markdown save/conflict behavior. Defer CRDT selection and E2EE document sync.
4. **Data model/contracts:** define only the entities and API/event contracts needed per release: community insights aggregation and optional discussion/resource tags; room lifecycle/join/leave; encrypted group-message delivery and key epoch rotation; poll ballot operations; announcement publication/notification; workspace roles and versioned Markdown saves/conflict resolution. The dashboard response is computed from existing records and does not create an analytics entity. The local text warning creates no server entity or API contract. Include membership checks on every endpoint/event and specify which fields are ciphertext versus metadata.
5. **Tasks:** create ordered phases with unique task IDs and parent-story traceability. First release tasks: US-3 community dashboard and tag aggregation; local keyword/phrase rules and author warning; membership/role authorization; US-11 reporting and local warning; US-4 simple polls; US-7 announcement board; US-13 local relay setup and 10-person voice capacity; IT authorization and approved two-device LAN test; and two-user same-computer demo. Do not add model weights, model-download tasks, or LFS assets. Second release: US-14 protocol review, key lifecycle, and E2EE group messaging. Third release: US-15 workspaces and versioned Markdown conflict handling. Add UI, accessibility, error handling, privacy/security review, and documentation tasks for each release. Do not mark work complete in advance. Keep automated test-writing tasks consistent with the project’s existing test policy; include acceptance criteria and verification steps without silently changing that policy.
6. **Traceability:** link every dashboard/tag task to US-3 or its existing content story (US-4/US-5), and every other task to US-13–US-15 or its existing parent story (US-4, US-7, US-11). Define measurable criteria per release: membership-isolated dashboard data; correct tag/activity aggregation and chart accessibility; local-only operation; author warning behavior; 10-person voice capacity and the two-user same-computer demonstration; group-message encryption and access revocation; poll correctness; announcement delivery; and preservation of conflicting document edits. Do not add image-processing criteria to the first release.

## Acceptance constraints applying to every feature

- No runtime request to an external service; no public hosting requirement. Local HTTP/API calls between the browser and the local application are allowed.
- Every read, write, search, notification, room join, poll ballot, and document sync checks active membership and role at the server boundary.
- All user-facing states have accessible labels, keyboard operation, screen-reader announcements, and clear success/error feedback.
- Fail closed for authorization and encryption; if the local keyword checker fails, show a notice and keep the existing submit/report flow available. Show a clear failure if the local voice connection fails.
- Document observable local-only verification criteria and avoid claims that E2EE protects metadata or that automated moderation is perfectly accurate.
