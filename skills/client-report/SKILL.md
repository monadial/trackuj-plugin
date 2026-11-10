---
name: client-report
description: Prepares last month's Client report of one Client in trackuj for an Owner, summarises it for the Client by Task Type and Activity, links the Overview where its PDF downloads, and drafts the cover email. Use when an Owner wants to report, bill or send a Client last month's work ("prepare Acme's report", "what do I send Acme for October?").
---

# Prepare a Client report

You work in trackuj as the User, through its tools. A **Client report** is a Report of one Client's work in a Period, by everyone who did it, naming none of them, to hand to that Client. Only an Owner reads one.

## How the tools answer

- Each tool but `list_organisations` takes `organisation`, the Organisation's address. Name it in every call once you know it.
- A refusal comes back as a tool error whose JSON names its `reason` and a `message` in the User's Language, as set in trackuj. Answer in the language of the User's request, even when the message and trackuj's names are in another. Tell the User the message, unless a step below says more.
- Moments come in the Organisation's Time zone, with their offset, and durations in seconds. Write a moment you send in the same form, with the same offset, and tell the User durations in hours and minutes.

## 1. The Organisation

Call `list_organisations`. Work in the Organisation the User named, by its name or address, or in their only one; when they have several and named none, ask which. Their `role` there must be `owner`: an Assistant reads no Client report, so tell an Assistant that an Owner of the Organisation prepares it, and stop.

## 2. The Client and the month

1. Call `list_clients`, and take the Client the User named, matched by its name, letter case aside. When none or several fit, ask.
2. The Period is the previous calendar month, unless the User named another: `from` its first day's midnight, `before` the next month's first midnight, both in the Organisation's Time zone, the `timeZone` that `read_settings` answers.

## 3. Read the Report

Call `report` with `audience` `client`, the Client's `clientId`, `from` and `before`. Add `language` (`sk` or `en`) when the User wants the Report in another Language than their own.

- `client_report_not_found`: only an Owner reads a Client report. Tell the User so.
- Its `seconds` is 0: the Client had no work that month. Tell the User, and draft no email.

## 4. Answer the Owner

Your own words to the Owner are in the language of their request, even when the Report's Language is another: only the summary and the email, which are for the Client, are in the Report's Language.

Write, from the Report alone, its numbers as it gives them:

1. **The summary for the Client**: the month, the total, then each Task Type with its total, and under it each Activity with its total. Name no User: a Client report names no one.
2. **The PDF**: the Report's `overview` link, where the Owner opens the Overview under the same filters and downloads the PDF to attach.
3. **The cover email**: a short email to the Client, in the Report's Language, naming the month and the total, with the summary by Task Type, saying the PDF is attached, and signed with a placeholder for the Owner's name. It quotes no price: the Report carries none.

Done when the Owner has the summary, the link and the draft email.
