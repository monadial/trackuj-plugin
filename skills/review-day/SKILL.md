---
name: review-day
description: Reviews the User's work today in trackuj, finds the time no Timer or Time Entry covers, and adds a Manual Time Entry for each gap the User confirms. Use when the User wants to check or complete their day's tracking ("what did I miss today?", "fill in my day").
---

# Review the day in trackuj

You work in trackuj as the User, through its tools. Their work today is in their Time Entries and in their unfinished Timers, Running or Paused, each with its **Work**: a Client, one of the Client's Task Types, and an Activity. Each measured its work in Segments, stretches with a start and an end. Work they forgot to track goes in as a **Manual Time Entry**, with a start, an end and its Work.

## How the tools answer

- Each tool but `list_organisations` takes `organisation`, the Organisation's address. Name it in every call once you know it.
- A refusal comes back as a tool error whose JSON names its `reason` and a `message` in the User's Language, as set in trackuj. Answer in the language of the User's request, even when the message and trackuj's names are in another. Tell the User the message, unless a step below says more.
- Moments come in the Organisation's Time zone, with their offset, and durations in seconds. Tell the User times as they read on a clock there, and durations in hours and minutes. Write a moment you send in the same form, with the same offset.
- On `organisation_lapsed`, the Organisation's Subscription has ended: nothing can be added in it until an Owner renews it in **Nastavenia → Predplatné** (Settings → Subscription in the English app). Tell the User so; reading their work still works.

## 1. The Organisation

Call `list_organisations`. One Organisation: work in it. Several: work in the one the User named, by its name or address; when they named none, ask which, listing the names.

## 2. The Settings

Call `read_settings`. Its `workingHours`, a `start` and an `end` on a 24-hour clock, are the Organisation's Working hours, and its `timeZone` is the Time zone today is a day of. The Working hours only bound which gaps are worth asking about: work outside them counts all the same. When the User says they worked other hours today, take theirs.

## 3. Read the day

Today runs from midnight to the next midnight in the Time zone.

1. Call `current_timers`: the User's unfinished Timers, each with its Work and Segments. The open Segment of a Running Timer runs until now.
2. Call `list_time_entries` with `from` today's midnight and `before` the next one. While it answers `hasMore`, call it again with `after` set to the last Time Entry's `id`. An Assistant sees only their own. An Owner sees everyone's, each naming its User (`userName`): unless they told you who they are, ask which of the listed Users they are, even when the list names one, and call it again with that User's `userEmail`. When they are none of them, they have no Time Entries today.

Done when you hold every Segment of the User's work today.

**Today's total** is the server's, not a sum of your own: call `summary` for today, with the User's `userEmail` when they are an Owner, and add each unfinished Timer's seconds so far. Call it again after adding Manual Time Entries.

## 4. Find the gaps

A **gap** is a stretch inside the Working hours, and before now, that none of today's Segments covers, of at least 15 minutes. Shorter ones are breaks. A Paused Timer's pause is a gap too: the User may have worked on something else meanwhile.

No gaps: tell the User their day is covered, with today's total, and stop here.

## 5. Propose Manual Time Entries

List each gap with its start, its end and its length. For each, propose a Manual Time Entry with its Work, from what the User says they did then, or else from the Work just before or after it, saying it is a guess.

Make each proposed Work real:

- the Client from `list_clients`, matched by its name, letter case aside;
- the Activity from `offered_activities` with the Client's `id`: an offered Activity that is that work, which also gives its Task Type (`taskTypeId`), or else a new Activity name in the User's words, short and on one line;
- the Task Type, when the Activity didn't give it, from the Client's `effectiveTaskTypes`.

When a Client, a Task Type or an Activity could be one of several, ask.

Then ask the User to confirm, change or drop each proposal. Add nothing until they confirm it.

## 6. Add what the User confirms

For each confirmed proposal, call `add_manual_time_entry` with `clientId`, `taskTypeId`, `startedAt` and `endedAt` as the User confirmed them, and either `activityId` or `activityName`.

- `overlaps`: it overlaps other work of the User's. Tell them, and propose the part of the gap that is still free.
- `client_archived`, `task_type_not_offered`, `activity_not_offered`, `activity_archived`: tell the User the message, and choose the Work again with them.

Done when every confirmed proposal is added or the User dropped it. Tell the User what was added, and today's total now.
