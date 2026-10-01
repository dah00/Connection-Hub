# Software Requirements Specification (SRS)

**Product:** Connection Hub
**Version:** 0.1 (Draft), September 29, 2026
**Author:** Obe Velonjatovo
**Related document:** PRD-connection-hub.md

---

## 1. Introduction

### 1.1 Purpose
This document specifies the functional and non-functional requirements for the Connection Hub MVP (v1). It is the reference for design, implementation, and testing.

### 1.2 Scope
Connection Hub is a mobile-friendly web app for verified adult students at one campus (De Anza College). It lets students join interest groups, commit to small in-person meetups, and get placed into weekly small intro groups that an AI facilitator helps start. Features marked **[v2]** are out of MVP scope and are included only so the design leaves room for them.

### 1.3 Definitions

| Term | Meaning |
|---|---|
| **Group** | A persistent interest community (e.g., Psychology) |
| **Meetup** | A specific in-person plan: activity, time, campus location, min/max participants |
| **Commit** | A user tapping "I'm down" on a meetup |
| **Open to Meet (OTM)** | Opt-in badge meaning "place me in a weekly intro group" |
| **Intro group** | A small group (3–4) of OTM users created by the weekly matching job |
| **Facilitator** | The AI-assisted bot that posts in intro group chats |
| **Free-time grid** | A user's weekly availability, stored as time blocks |
| **Slot** | A 30-minute block within the free-time grid |

---

## 2. Overall description

### 2.1 User classes

| Class | Description |
|---|---|
| **Student** | Verified, 18+ user. All standard features |
| **Admin** | Reviews reports, manages groups and campus locations, can suspend users |
| **System** | Scheduled jobs and the facilitator |

### 2.2 Operating environment
- Mobile-first responsive web app on current Chrome, Safari, and Firefox (desktop and mobile).
- Hosted on a managed platform (see 2.4).

### 2.3 Constraints
- Users must be 18 or older.
- Only verified school email addresses may register.
- No direct messages between users outside of intro groups and confirmed meetups (v1).
- AI calls must not receive private data (see 6.3).

### 2.4 Suggested stack (non-binding)

| Layer | Suggestion | Notes |
|---|---|---|
| Frontend + API | Next.js (App Router) + TypeScript + Tailwind | Matches existing experience |
| Database | PostgreSQL | With an ORM or query builder (Prisma or Drizzle) |
| Auth | Email one-time code or magic link | Restrict to school domain |
| Realtime chat | **Open decision** | Vercel serverless doesn't hold WebSockets. Options: polling (simplest for v1), a hosted realtime service (Pusher, Ably, Supabase Realtime), or a separate Node server |
| Scheduled jobs | **Open decision** | Weekly matching and facilitator timers. Options: Vercel Cron + database-driven job table, or a queue (e.g., BullMQ with Redis) on a separate worker |
| AI | Claude API | Message generation only (see Section 6) |
| Email | Transactional email service | Verification codes and notifications |
| File storage | Object storage | Profile photos |

### 2.5 Assumptions
- Students are willing to fill out a weekly free-time grid.
- A small number of groups (1–3) is enough for a pilot.
- The school email domain can be used to verify enrollment.

---

## 3. Functional requirements

Priority: **M** = must (MVP), **S** = should (MVP if time allows), **L** = later (v2).

### 3.1 Authentication and account (AUTH)

| ID | Requirement | Pri |
|---|---|---|
| AUTH-1 | The system shall allow registration only with an email address on an allowed school domain (configurable list). | M |
| AUTH-2 | The system shall verify email ownership with a one-time code or magic link that expires after 15 minutes. | M |
| AUTH-3 | The system shall require users to confirm they are 18 or older and accept the terms before creating an account. | M |
| AUTH-4 | The system shall allow users to log out and to delete their account and all personal data. | M |
| AUTH-5 | The system shall rate-limit verification requests per email and per IP. | M |

### 3.2 Profile (PROF)

| ID | Requirement | Pri |
|---|---|---|
| PROF-1 | Users shall have a profile with: display name, 1–3 photos, major, age range (18–21, 22–25, 26+), interests (from a fixed list), hobbies (free text, short), and up to 2 prompt answers. | M |
| PROF-2 | Exact age and date of birth shall not be stored or displayed. | M |
| PROF-3 | Users shall be able to edit their profile at any time. | M |
| PROF-4 | Profiles shall be visible only to logged-in, verified users. | M |
| PROF-5 | Photos shall be limited in size and type (JPEG/PNG/WebP, ≤ 5 MB) and resized on upload. | M |
| PROF-6 | Users shall choose which profile fields the facilitator may reference (default: interests, major, hobbies). | S |

### 3.3 Availability (AVAIL)

| ID | Requirement | Pri |
|---|---|---|
| AVAIL-1 | Users shall set a weekly free-time grid of 30-minute slots, Monday–Friday, 8:00am–8:00pm (configurable). | M |
| AVAIL-2 | The free-time grid shall be required to complete onboarding. | M |
| AVAIL-3 | Users shall be able to update the grid at any time; changes apply to future matching only. | M |
| AVAIL-4 | Availability shall be stored in the campus time zone (America/Los_Angeles). | M |
| AVAIL-5 | The system shall prompt users to review their grid at the start of each quarter. | S |

### 3.4 Groups (GRP)

| ID | Requirement | Pri |
|---|---|---|
| GRP-1 | Users shall be able to browse, join, and leave interest groups. | M |
| GRP-2 | A group page shall show upcoming meetups first, then members. | M |
| GRP-3 | Only admins shall create groups in v1. | M |
| GRP-4 | Groups shall have a discussion board. | L |

### 3.5 Meetups (MEET)

| ID | Requirement | Pri |
|---|---|---|
| MEET-1 | A group member shall be able to create a meetup with: title, optional description, date/time, duration, campus location (from admin list), min participants (≥ 3), max participants (≤ 6). | M |
| MEET-2 | Users shall be able to commit ("I'm down") to and withdraw from a meetup until its start time. | M |
| MEET-3 | A meetup shall move to **Confirmed** when commits reach the minimum. | M |
| MEET-4 | Commits shall stop at the maximum; further users see "Full." | M |
| MEET-5 | A meetup that has not reached its minimum 2 hours before start shall move to **Expired** and notify committed users neutrally ("This one didn't come together"). | M |
| MEET-6 | Users shall not see which specific users withdrew or didn't commit. | M |
| MEET-7 | The creator shall be able to cancel a meetup; committed users are notified. | M |
| MEET-8 | The Home tab shall list upcoming meetups in the user's groups, ranked by overlap with the user's free-time grid. | M |
| MEET-9 | After a meetup, members shall be asked "Did you meet?" (one tap). | S |
| MEET-10 | After a meetup, members shall be asked "Meet again next week?"; if a majority says yes, a new meetup is created at the same time and place. | L |

**Meetup states:** `open → confirmed → completed`, or `open → expired`, or `open/confirmed → cancelled`.

### 3.6 Chat (CHAT)

| ID | Requirement | Pri |
|---|---|---|
| CHAT-1 | A chat shall exist for each confirmed meetup and each intro group, visible only to its members. | M |
| CHAT-2 | Chats shall support text messages and emoji; no images or files in v1. | M |
| CHAT-3 | Chats shall support a poll message type (used by the facilitator for time votes). | M |
| CHAT-4 | Facilitator messages shall be visually distinct and labeled as a bot. | M |
| CHAT-5 | New messages shall appear within 5 seconds (polling acceptable in v1). | M |
| CHAT-6 | Chats shall become read-only 7 days after the meetup or after the intro group closes. | S |

### 3.7 Open to Meet and weekly matching (OTM)

| ID | Requirement | Pri |
|---|---|---|
| OTM-1 | Users shall be able to turn the OTM badge on and off. | M |
| OTM-2 | The badge shall turn off automatically 4 weeks after being turned on, with a prompt to renew. | M |
| OTM-3 | A matching job shall run every Monday at 8:00am campus time. | M |
| OTM-4 | The job shall place OTM users into groups of 3–4 (target 4). | M |
| OTM-5 | Matching shall require every group to share at least two common 1-hour free windows in the coming week (Mon–Fri). | M |
| OTM-6 | Matching shall prefer groups with more shared interests, then shared major. | M |
| OTM-7 | Matching shall not place users together who have blocked each other. | M |
| OTM-8 | Matching should avoid regrouping users who were in the same intro group within the last 4 weeks. | S |
| OTM-9 | Users who cannot be matched shall be carried over to the next week and told neutrally ("We couldn't find a group that fits your schedule this week"). | M |
| OTM-10 | Each user shall be in at most one active intro group at a time. | M |
| OTM-11 | Waves: mutual opt-in between two OTM users; a 1:1 chat opens only if both wave. | L |

> Implementation note: matching is a deterministic algorithm (a greedy approach is fine for v1). The AI is **not** used for matching decisions.

### 3.8 Facilitator (FAC)

The facilitator follows fixed rules in code. The AI is used only to write message text (see Section 6).

| ID | Requirement | Pri |
|---|---|---|
| FAC-1 | When an intro group is created, the facilitator shall post an **opening message** within 1 minute containing: (a) an intro naming shared interest(s), (b) a one-line icebreaker, (c) a poll with **two** time options from the group's shared free windows at a campus location, plus "Can't make it." | M |
| FAC-2 | Time options shall be at least 24 hours in the future and within the current week. | M |
| FAC-3 | When **all** members have voted, or a time option reaches a majority of members, the facilitator shall post a **confirmation** and create a confirmed meetup. | M |
| FAC-4 | If fewer than 3 members can attend any option, the facilitator shall offer one new pair of time options (at most once). | M |
| FAC-5 | If there are **no human messages or votes** 24 hours after the opening message, the facilitator shall post **one nudge**. | M |
| FAC-6 | If there are still no human messages or votes 48 hours after the nudge, the group shall close quietly and its members return to the matching pool. | M |
| FAC-7 | The facilitator shall **not** post while humans are actively chatting: no facilitator message within 30 minutes of the last human message, except confirmations and reminders. | M |
| FAC-8 | If humans have exchanged at least 5 messages but no plan is confirmed after 24 hours, the facilitator may post one **plan suggestion** (new time poll). | S |
| FAC-9 | The facilitator shall post a **reminder** 3 hours before a confirmed meetup, optionally with a light icebreaker. | M |
| FAC-10 | The facilitator shall post no more than **4 messages** per intro group per week, excluding the confirmation and reminder. | M |
| FAC-11 | The facilitator shall reference only profile fields the members allowed (PROF-6; default interests, major, hobbies). | M |
| FAC-12 | After the meetup, the facilitator shall ask "Meet again next week?" | L |

**Intro group states:**

```
created → opened (opening message posted)
opened → planned (meetup confirmed)            [FAC-3]
opened → nudged (no activity 24h)              [FAC-5]
nudged → planned                                [FAC-3]
nudged → closed (no activity 48h after nudge)   [FAC-6]
planned → completed (meetup time passed)
any → closed (all members left / admin action)
```

### 3.9 Safety and moderation (SAFE)

| ID | Requirement | Pri |
|---|---|---|
| SAFE-1 | Users shall be able to block any user. Blocked users can't see each other's profiles, can't be matched together, and can't join the same meetup (the later one to commit is prevented). | M |
| SAFE-2 | Users shall be able to report a user or a message with a reason category and optional note. | M |
| SAFE-3 | Admins shall have a review queue for reports with actions: dismiss, warn, suspend, remove content. | M |
| SAFE-4 | Suspended users shall be logged out and unable to log in. | M |
| SAFE-5 | Meetup locations shall be chosen from an admin-managed list of public campus spots. | M |
| SAFE-6 | Terms and a short safety guide shall be shown during onboarding. | M |
| SAFE-7 | AI-assisted flagging of harmful messages for admin review. | L |

### 3.10 Notifications (NOTIF)

| ID | Requirement | Pri |
|---|---|---|
| NOTIF-1 | The system shall send email for: new intro group, meetup confirmed, meetup expired or cancelled, reminder (3h before). | M |
| NOTIF-2 | Users shall be able to turn off each notification type except safety and account emails. | M |
| NOTIF-3 | An in-app notification list shall show the same events. | S |
| NOTIF-4 | Push notifications. | L |

### 3.11 Admin (ADM)

| ID | Requirement | Pri |
|---|---|---|
| ADM-1 | Admins shall manage groups, the interest list, and campus locations. | M |
| ADM-2 | Admins shall view basic metrics: signups, OTM users, intro groups by state, meetups by state. | S |

---

## 4. Data model (conceptual)

| Entity | Key fields |
|---|---|
| **User** | id, email, email_verified_at, role (student/admin), status (active/suspended), created_at |
| **Profile** | user_id, display_name, major, age_range, hobbies, prompt_answers (json), facilitator_visible_fields |
| **Photo** | id, user_id, url, position |
| **Interest** | id, name |
| **UserInterest** | user_id, interest_id |
| **AvailabilitySlot** | user_id, weekday (1–5), start_time (30-min block) |
| **Group** | id, name, description, interest_id |
| **GroupMember** | group_id, user_id, joined_at |
| **Location** | id, name, description, active |
| **Meetup** | id, group_id (nullable), intro_group_id (nullable), creator_id (nullable for facilitator), title, starts_at, duration_min, location_id, min_people, max_people, state |
| **Commitment** | meetup_id, user_id, created_at |
| **OtmStatus** | user_id, enabled, enabled_at, expires_at |
| **IntroGroup** | id, week_start, state, created_at, opened_at, nudged_at, closed_at |
| **IntroGroupMember** | intro_group_id, user_id |
| **Chat** | id, meetup_id or intro_group_id |
| **Message** | id, chat_id, sender_id (null = facilitator), type (text/poll/system), body, created_at |
| **Poll / PollOption / PollVote** | poll options with time + location; votes per user |
| **Block** | blocker_id, blocked_id |
| **Report** | id, reporter_id, target_user_id, message_id, reason, note, status |
| **FacilitatorEvent** | id, intro_group_id, kind (open/nudge/suggest/confirm/remind), prompt_version, used_fallback, created_at |
| **Notification** | id, user_id, kind, payload, read_at |

---

## 5. Non-functional requirements

### 5.1 Security and privacy
- NFR-1: All traffic over HTTPS.
- NFR-2: Session tokens in secure, HTTP-only cookies.
- NFR-3: Authorization checked on the server for every request (chat membership, meetup membership, admin role).
- NFR-4: Photos served through URLs that only logged-in users can access.
- NFR-5: Account deletion removes profile, photos, availability, and memberships within 30 days; messages are anonymized.
- NFR-6: Secrets (API keys) stored in environment variables, never in the client.

### 5.2 Performance
- NFR-7: Main pages load in under 2 seconds on a mid-range phone over 4G.
- NFR-8: The weekly matching job finishes in under 1 minute for 2,000 OTM users.

### 5.3 Reliability
- NFR-9: If an AI call fails or times out (10s), the facilitator uses a fallback template so messages are never skipped.
- NFR-10: Scheduled jobs are idempotent: rerunning them doesn't create duplicate groups or messages.

### 5.4 Usability and accessibility
- NFR-11: Onboarding can be completed in under 3 minutes.
- NFR-12: Meets WCAG 2.1 AA for color contrast, labels, and keyboard navigation.

### 5.5 Cost
- NFR-13: AI cost stays under $0.01 per facilitator message (short prompts, small model where possible).

---

## 6. AI requirements

### 6.1 What the AI does
Writes the **text** of facilitator messages: intro line, icebreaker, nudge, plan suggestion wording, reminder.

### 6.2 What the AI does not do
Choose times, locations, group members, or when to speak. Those come from code (Sections 3.7–3.8) and are passed into the prompt as fixed values.

### 6.3 Inputs
- Message kind (open, nudge, suggest, remind).
- For each member: first name and only the allowed fields (FAC-11).
- Shared interests computed by code.
- Time options and location computed by code.
- **Never:** email, photos, full availability, reports, blocks, or other members' hidden fields.

### 6.4 Output
- Structured JSON, e.g. `{ "intro": "...", "icebreaker": "..." }`. Code assembles the final message and poll.
- Validated: length limits (intro ≤ 200 characters, icebreaker ≤ 120), required fields present. Invalid output → fallback template.

### 6.5 Tone and content rules (in the system prompt)
- Friendly, brief, casual; at most one emoji per part.
- Icebreakers answerable in one line; no personal, romantic, political, religious, health, or embarrassing topics.
- No mention of loneliness, having no friends, or being new in a way that singles anyone out.
- Never pretend to be a human.

### 6.6 Evaluation
- A set of 20+ test member profiles and expected constraints, run whenever the prompt changes.
- Log prompt version and fallback use in FacilitatorEvent.

---

## 7. External interfaces

| Interface | Use |
|---|---|
| Claude API | Facilitator message text |
| Email service | Verification codes, notifications |
| Object storage | Profile photos |
| Realtime provider (if chosen) | Chat updates |

---

## 8. Open decisions

1. Realtime approach for chat (polling vs. hosted service vs. own WebSocket server).
2. Job runner for matching and facilitator timers.
3. Auth method (one-time code vs. magic link).
4. Final school email domain(s) for verification.
5. Default campus locations list.
6. Whether to collect "Did you meet?" in v1 (MEET-9).

---

## 9. Suggested build order

1. Auth + school email verification
2. Profile + photos + interests
3. Free-time grid
4. Groups + meetups + commit/confirm/expire
5. Chat (polling first)
6. Open to Meet + weekly matching job
7. Facilitator: state machine and timers with **fallback templates only**
8. Facilitator: add Claude API for message text
9. Block/report + admin queue
10. Email notifications
