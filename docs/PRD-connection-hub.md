# Product Requirements Document (PRD)

**Product:** Connection Hub
**Author:** Obe Velonjatovo
**Status:** Draft v0.1, September 29, 2026
**Launch campus:** De Anza College

---

## 1. Summary

Connection Hub helps college students meet each other in person by turning shared interests into small, scheduled, low-pressure meetups. Instead of a feed or a large group chat, the app's core unit is a **meetup**: a specific activity, at a specific time and place, with 3–5 people. An **AI facilitator** removes the "who goes first" problem by opening every new small-group chat with an icebreaker and a concrete plan the group can vote on.

## 2. Problem

Many students want to meet people on campus, but each waits for someone else to make the first move. As a result, few connections form.

- **Pluralistic ignorance:** each student privately wants to connect but assumes others don't.
- **Research support:** people underestimate how much strangers want to talk to them (Epley & Schroeder, 2014) and how much others liked them after a conversation (Boothby et al., 2018, "the liking gap").
- **Bystander effect in large groups:** in big group chats, everyone assumes someone else will speak first, so the chat stays silent.
- **Founder observation:** at both De Anza and San Jose State, students were rarely the first to reach out. The groups that worked, like an informal Friday soccer game, had a shared activity, a fixed time and place, low stakes, and repetition.

> The observation is anecdotal for De Anza specifically. Phase 0 (Section 10) is designed to test it.

## 3. Goals and non-goals

### Goals
1. Get students to their **first in-person meetup** within two weeks of signing up.
2. Make starting a connection **take one tap**: no cold messages, no visible rejection.
3. Turn one-time meetups into **recurring** ones, like the soccer game.
4. Serve as a portfolio project showing full-stack and applied AI engineering skills.

### Non-goals (for now)
- A general social feed, posts, likes, or followers.
- Dating or romantic matching.
- Open direct messages between strangers.
- Official club management (rosters, dues, officer elections).
- Campuses other than De Anza.
- Users under 18.

## 4. Target users

| Persona | Description | Main need |
|---|---|---|
| **The newcomer** | New to De Anza, knows almost nobody, often a transfer or returning student | Meet a few people without the pressure of making the first move |
| **The commuter** | On campus only for classes, leaves right after | Meet people in the gaps between classes they already have |
| **The interest-seeker** | Has friends but wants people who share a specific interest (psychology, chess, soccer) | Find a regular, low-effort activity around that interest |
| **The organizer** | Happy to start things but doesn't know who would come | See who is interested and free before committing to a plan |

## 5. Product principles

1. **Meetups over feeds.** Every feature should lead toward meeting in person.
2. **The app makes the first move.** Users respond to invitations and suggestions; they never have to break silence alone.
3. **Small groups.** 3–5 people, to avoid the bystander effect.
4. **No visible rejection.** Unfilled meetups expire quietly; unanswered waves aren't shown as declined.
5. **Repetition builds friendships.** Make it easy to meet the same people again.
6. **Safety first.** Verified students, adults only, no open DMs, public campus locations.
7. **AI where it helps, code everywhere else.** AI writes messages and suggestions. Deterministic code handles scheduling, matching rules, and timing.

## 6. Features

### 6.1 MVP (v1)

| # | Feature | Description |
|---|---|---|
| F1 | **School email signup** | Verified De Anza student email, 18+ confirmation |
| F2 | **Profile** | Name, photo(s), major, age range, interests, hobbies, 2 short prompt answers |
| F3 | **Free-time grid** | Weekly grid of when the student is on campus and free |
| F4 | **Interest groups** | Browse and join groups such as Psychology, Soccer, Chess, CS |
| F5 | **Meetups** | Post a meetup (activity, time, campus location, min/max spots); others tap "I'm down" |
| F6 | **Meetup confirmation** | Meetup is confirmed when the minimum number commits; unfilled meetups expire quietly |
| F7 | **Meetup group chat** | Opens for committed members once a meetup is confirmed |
| F8 | **"Open to Meet" badge** | Opt-in badge; turns off automatically after 4 weeks unless renewed |
| F9 | **Weekly small-group intros** | Every Monday, students with the badge are placed into groups of 3–4 by shared interests and overlapping free time |
| F10 | **AI facilitator** | Opens each intro group with an icebreaker plus a meetup plan with time options to vote on (details in 6.3) |
| F11 | **Safety tools** | Block, report, and a basic admin review queue |
| F12 | **Notifications** | Email (v1) for new intro groups, confirmed meetups, and reminders |

### 6.2 Later (v2+)

- **Waves:** mutual opt-in between two "Open to Meet" students; a chat opens only if both wave.
- **Recurring meetups:** "Meet again next week?" turns a meetup into a weekly one.
- **AI meetup suggestions for groups:** "5 people in Psychology are free Thursday at 2pm. Start a meetup?"
- **AI moderation:** flag harmful messages for admin review.
- **Group discussion board** (secondary to meetups).
- **Push notifications / mobile app.**
- **More campuses.**

### 6.3 AI facilitator behavior (summary)

The facilitator is a clearly labeled bot in small-group chats. It:

1. **Opens the chat immediately** when an intro group is created, with one message containing:
   - a short intro naming the shared interest(s),
   - a one-line icebreaker,
   - a coffee/meetup plan at a campus spot with **two time options** plus "Can't make it."
2. **Stays quiet while people are chatting.**
3. **Suggests a plan** if people are chatting but no plan has been agreed on.
4. **Confirms the plan** once the vote settles.
5. **Sends one reminder** before the meetup, optionally with a light icebreaker.
6. **Asks "Meet again?"** after the meetup (v2).
7. **Sends at most one nudge** if nobody responds, then quietly closes the group and rematches those students the next week.

It uses only information members made visible on their profiles. Full rules are in the SRS.

## 7. Key user flows

**Onboarding:** Sign up with school email → confirm 18+ → choose interests → fill free-time grid → build profile → choose whether to turn on "Open to Meet."

**Meetup loop:** Open a group → see upcoming meetups → tap "I'm down" → meetup confirms when minimum is reached → group chat opens → meet in person.

**Open to Meet loop:** Turn on badge → Monday: placed in a group of 3–4 → facilitator posts icebreaker and time vote → group votes → plan confirmed → reminder → meet.

## 8. App structure

| Tab | Purpose |
|---|---|
| **Home** | Upcoming meetups that match your interests and free time; pending intro groups |
| **Groups** | Browse and join interest groups |
| **Open to Meet** | Your weekly intro group and badge status |
| **Plans** | Meetups you've joined and their chats |
| **Profile** | Profile, free-time grid, settings |

## 9. Success metrics

| Metric | Target for MVP pilot |
|---|---|
| Signup → completed profile + free-time grid | ≥ 70% |
| Intro groups where at least one member replies within 24h | ≥ 60% |
| Intro groups that confirm a meetup | ≥ 40% |
| Confirmed meetups where most members attend (self-reported) | ≥ 60% |
| Users who attend 2+ meetups within 4 weeks | ≥ 30% |
| Groups that meet again without the app suggesting it | Tracked; this is the "soccer test" |

## 10. Launch plan

**Phase 0: Manual test (2–3 weeks, almost no code)**
- One interest (Psychology). Sign-ups through flyers and a QR code linking to a form (interests, free time, 18+ confirmation).
- Obe matches groups of 3–4 by hand and sends the icebreaker and time options (drafted with Claude) by text or email.
- Measure sign-ups, replies, show-up rate, and repeat meetups.
- **Go/no-go:** build the MVP if at least ~40% of groups meet and some want to meet again.

**Phase 1: MVP build** (features F1–F12), then pilot with 2–3 groups.

**Phase 2:** v2 features based on what the pilot shows.

## 11. Risks

| Risk | Mitigation |
|---|---|
| **Cold start:** not enough users for matching | Launch with 1–3 groups, not the whole campus; run Phase 0 first |
| **Commuter campus:** little overlapping free time | Free-time grid is required; match on overlap; meetups between classes |
| **Harassment or dating-app behavior** | No open DMs, mutual opt-in, block/report, friendship-focused profiles |
| **Minors on campus** (dual enrollment) | 18+ requirement at signup; state it in terms |
| **"Open to Meet" stigma** | Badge, not a room; framed as open to anyone; expires automatically |
| **AI message quality or tone** | Short, template-guided prompts; fallback templates if the AI call fails |
| **College policy** | Talk to De Anza student life before any official promotion |

## 12. Open questions

1. Name check: domain and app-name availability for "Connection Hub."
2. De Anza's exact student email domain and whether the college allows student-built apps to use it for verification.
3. Which campus locations to offer as default meetup spots.
4. Web-only (mobile-friendly) for v1, or a mobile app later?
5. How to collect attendance: a one-tap "Did you meet?" after each meetup, or skip for v1?
