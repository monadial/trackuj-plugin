---
name: track
description: Starts, switches, pauses and stops the User's trackuj Timer around a piece of work. Use when the User starts working on something ("let's work on Acme's checkout"), goes back to work they tracked earlier ("back to the logo"), takes or ends a break, or is done.
---

# Track work in trackuj

You work in trackuj as the User, through its tools, with what their Role allows. A Timer measures one piece of work, its **Work**: a Client, one of the Client's Task Types, and an Activity, the concrete work ("Checkout flow", "Logo"). In an Organisation, one Timer of the User's runs at a time: the Running one. Others may be Paused. A stopped Timer leaves a Time Entry, unless it measured less than the Grace period: then it is Discarded and leaves none. Only the stop's answer tells which: never say it before.

## How the tools answer

- Each tool but `list_organisations` takes `organisation`, the Organisation's address. Name it in every call once you know it.
- A refusal comes back as a tool error whose JSON names its `reason` and a `message` in the User's Language, as set in trackuj. Answer in the language of the User's request, even when the message and trackuj's names are in another. The sections below say what to do for each reason; for any other, tell the User the message.
- Moments come in the Organisation's Time zone, with their offset, and durations in seconds. Tell the User times as they read on a clock there, and durations in hours and minutes. Write a moment you send in the same form, with the same offset.
- On `organisation_lapsed`, the Organisation's Subscription has ended: nothing in it can be started, stopped or changed until an Owner renews it in **Nastavenia → Predplatné** (Settings → Subscription in the English app). Tell the User so, and that reading their work still works. Retry nothing.

## 1. The Organisation

Call `list_organisations` once in the conversation. It lists each Organisation's `address`, `name`, the User's `role` there, and whether a Timer of theirs is Running there (`timerRunning`).

- One Organisation: work in it.
- Several: work in the one the User named, by its name or address. When they named none, ask which, listing the names, and start nothing until they answer.

Done when you hold one address.

## 2. The Work

1. **Client.** Call `list_clients`. Match what the User said to a Client's `name`, letter case aside; a part of the name counts ("Acme" for "Acme s.r.o."). Exactly one fits: take it. None or several fit: ask, naming the ones that come closest, and start nothing yet.
2. **Activity.** Call `offered_activities` with the Client's `id`. Each offered Activity has its `name`, the Task Type it is under (`taskTypeId`) and when the User last tracked it (`youLastTracked`). Find the ones that are the work the User described, in their words or in others ("checkout" fits "Checkout flow"). Keep to the Task Type the User named, if they named one.
   - Exactly one fits: it is the Activity, by its `id`, and its Task Type is the Work's.
   - Several fit: ask which, listing them with their Task Types.
   - None fits: the Work gets a new Activity, named in the User's words as it would read in a Report: short, on one line, in the language they wrote in. Reusing an offered Activity keeps the User's work together, so a new name is for new work only.
3. **Task Type.** When the Activity didn't give it: the Client's `effectiveTaskTypes` (from `list_clients`) that fits what the User said, or the only one the Client offers. When several fit, ask which.

Done when the Work has a Client, a Task Type and either an offered Activity's `id` or a new Activity name, each chosen by a match or by the User's answer.

## 3. Start the Timer

Call `start_timer` with `clientId`, `taskTypeId`, and either `activityId` or `activityName`. Tell the User what is running: the Client, the Task Type and the Activity.

When it is refused:

- `another_timer_running`: the refusal carries the Running Timer (`runningTimer`) and its Work.
  - Its Work is the one the User asked for: it is already running. Say so, and start nothing.
  - Otherwise ask the User whether to **pause** or **stop** that Timer, naming its Work and nothing of what a stop would leave. On "pause", call `pause_timer` with its `id`, then `start_timer` as before. On "stop", call `start_timer` again with `stopRunning: true`, which stops it as the User's own stop. Then tell the User what started, and what the stop made: the Time Entry in `stopped`, or that it was Discarded, when `stopped` carries no Time Entry.
- `client_archived`, `task_type_not_offered`, `activity_not_offered`, `activity_archived`: tell the User the message, and choose the Work again with them.

## Back to earlier work

When the User goes back to work they tracked before ("back to the logo", "let's carry on with the homepage"), **Start again** from their latest Time Entry of that Activity: it resumes their Paused Timer of that Activity, or starts a new Timer with the same Work.

1. Find the Activity among the Client's `offered_activities`, as in **2. The Work**, among those with a `youLastTracked`. When the User named no Client, call `list_time_entries` with `from` 30 days ago and look for the Activity's name in the Time Entries' Work; each of them names its Client.
2. Call `current_timers`, the User's own unfinished Timers. While one of them is of that Activity, `youLastTracked` is now, so go by that Timer:
   - It is Running: it is running already. Say so, naming it, and ask whether to stop it and start anew. Only on "yes", call `stop_timer` with its `id`, then `start_timer` with its Work, and tell the User what the stop made.
   - It is Paused: call `resume_timer` with its `id`, and tell the User it resumed. On `another_timer_running` or `pause_limit_passed`, do as **Breaks** says.
3. Otherwise, call `list_time_entries` with its `activityId`, and `before` one second after its `youLastTracked`. An Owner sees everyone's Time Entries, so take the User's own: the one whose `timeEntry.endedAt` is the same moment as `youLastTracked`.
4. Call `start_again` with that Time Entry's `id`. Tell the User whether it resumed their Paused Timer (`resumed`) or started a new one.

When `start_again` is refused:

- `same_activity_running`: the User's Timer of that Activity is Running already. Say so, naming it, and ask whether to stop it and start anew. Only on "yes", call `start_again` again with `stopRunning: true`.
- `another_timer_running`: ask whether to pause or stop the Running Timer, as in **3. Start the Timer**. On "pause", call `pause_timer`, then `start_again` again. On "stop", call `start_again` again with `stopRunning: true`.

Whenever `start_again` answers `stopped`, tell the User which Timer it stopped, by its Work, and the Time Entry that stop made, or that it was Discarded.

## Breaks

- The User takes a break: call `current_timers`, and `pause_timer` with the Running Timer's `id`.
- They are back: call `resume_timer` with the `id` of the Paused Timer they mean; when several are Paused and they don't say which, ask. On `pause_limit_passed`, it was Paused longer than the Pause limit and can only be stopped: offer to stop it, which ends it where it was paused, and to start the same Work anew. On `another_timer_running`, ask whether to pause or stop the Running Timer, as in **3. Start the Timer**; on "stop", resume with `stopRunning: true`.

## Done

When the User is done ("done", "finished", "stop the Timer"), call `current_timers` and `stop_timer` with the `id` of the Running Timer, or of the one they name. When they say when they finished ("I finished at five"), send that moment as `endedAt`. Tell the User the Time Entry it made and its duration, or, when it answers no Time Entry, that the Timer measured less than the Grace period and was Discarded.

`discard_timer` throws the measured work away. Call it only when the User asks to discard a Timer.
