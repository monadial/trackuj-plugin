---
name: where-did-time-go
description: Answers questions about the User's tracked work in trackuj with the Summary's totals, by Client, Task Type, Activity, day and, for an Owner, by User. Use when the User asks how much time went where ("where did my week go?", "how much did I do for Acme this month?", "how long did the logo take?").
---

# Where did the time go

You work in trackuj as the User, through its tools. The **Summary** totals the work of a Period: in all, by Client, by Task Type, by Client and Task Type, by Client, Task Type and Activity, by day, and for an Owner by User. Its totals are the ones the app shows. Answer from them, as they are.

## How the tools answer

- Each tool but `list_organisations` takes `organisation`, the Organisation's address. Name it in every call once you know it.
- A refusal comes back as a tool error whose JSON names its `reason` and a `message` in the User's Language, as set in trackuj. Answer in the language of the User's request, even when the message and trackuj's names are in another. Tell the User the message.
- Moments come in the Organisation's Time zone, with their offset, and durations in seconds. Write a moment you send in the same form, with the same offset, and tell the User durations in hours and minutes.

## 1. The Organisation

Call `list_organisations`. Work in the Organisation the User named, by its name or address, or in their only one; when they have several and named none, ask which.

## 2. The question

1. **The Period**, from the question: `from` the first midnight it covers, `before` the midnight after its last day. "Today", "yesterday", "this week", "last month" each name one. Call `read_settings` for the Organisation's `timeZone`, which the midnights are in, and its `firstDayOfWeek`, the day a week starts on, from 1 for Monday to 7 for Sunday. When the question names no time, ask for it.
2. **What it narrows to**, when the question names it:
   - a Client: `clientId`, from `list_clients`, matched by its name, letter case aside;
   - a Task Type: `taskTypeId`, from the Client's `effectiveTaskTypes`;
   - an Activity: `activityId`, from `offered_activities` with its Client's `id`, matched to the User's words; when several fit, ask which;
   - for an Owner, a User (`userEmail`) or a Role (`role`).

## 3. Read the Summary

Call `summary` with the Period and the filters.

An Owner's Summary totals everyone's work. When an Owner asks about their own work, ask which of the Users in `byUser` they are, even when it names one, unless they told you, and call `summary` again with that User's `email` as `userEmail`. When they are none of them, they have no work in the Period.

## 4. Answer

Answer the question with the Summary's numbers, in hours and minutes:

- "Where did it go": the total, then the biggest Clients, and within them their Task Types and Activities (`byClientTaskTypeAndActivity`).
- "How much for a Client" or "for a Task Type": that total, and how it splits.
- About one Activity: its total, from the Summary narrowed to its `activityId`.
- By day: `byDay`.

Say which Period the numbers cover. When the Summary holds no work, say so.
