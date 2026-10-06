---
name: trackiq-amazon-amc-media-mix
description: Builds an Amazon Marketing Cloud path-to-purchase and media mix deck for an Amazon brand — attribution paths, which channel combinations recruit new customers versus harvest existing ones, cost per new-to-brand customer by channel, what DSP is worth alone versus in a path, and the budget reallocation the numbers support. Use when the user asks for an AMC report, path to purchase, media mix, channel mix, attribution paths, new-to-brand analysis, cost per new customer, a full-funnel or incrementality read, "is DSP worth it", "where should the budget go", or a monthly or quarterly media review.
---

# TrackIQ: Amazon AMC Media Mix

A twelve-slide deck arguing one case: **where the money is going versus
where the new customers are coming from.** Read in ten minutes by a brand
marketing lead or an agency's client, and it ends in a single budget
reallocation with the arithmetic shown.

Output is one self-contained `.html` file, 1920×1080 slides, logos embedded.
It opens in any browser and prints to PDF.

## Requires

- The TrackIQ MCP, for `list_marketplaces`, `get_amc_attribution_paths`,
  `get_amc_ntb_purchases`, `get_amc_time_to_conversion`,
  `get_account_overview` and `get_dsp_performance`. Ask which brand and
  marketplace before pulling anything.
- The account must have **AMC enabled and DSP running**. Without DSP there
  is no media mix to analyse — offer the weekly recap instead.
- Nothing else. No filesystem, no shell, no internet.
- **Without the MCP:** ask the user to export the AMC attribution-path,
  NTB-by-campaign and time-to-conversion tables plus monthly spend by
  channel, and build from those. Slides 3–10 all work from pasted figures;
  say on slide 11 which pulls were manual.

## First run

Fill in a copy of `assets/account.example.md` saved as account.md beside the
skill. Every TrackIQ skill reads the same file, so an account already set up
for another TrackIQ report needs nothing added here.

1. **Brand and marketplace** — which account to pull, and which TrackIQ MCP
   when several are connected
2. **Delivery** — in-chat, file, Slack, n8n or email

If the runtime has no filesystem, print the same block and ask the user to
paste it into their project instructions once.

## Read first

- `assets/pulls.md` — the call sequence, and the five ways this data
  misleads. Read before the first tool call.
- `assets/derivations.md` — every figure's formula, including the
  reallocation arithmetic
- `assets/narrative.md` — what each of the twelve slides is for, and the voice
- `assets/checks.md` — the pre-send checks, including the footer measurement
- `assets/account.example.md` — the first-run answers, filled in once

Copy `assets/deck-template.html` and replace every `{{TOKEN}}`. Do not
rebuild the shell — its layout, print stylesheet and footer reserve are the
part that has been tested.

## Delivery

The deck is produced in the chat first. Delivery is the last step and the
method comes from the Delivery block in account.md — never ask per run.

| Method | What to do | Needs |
|---|---|---|
| `in-chat` | Return the HTML. The default, and the fallback for every other method. | nothing |
| `file` | Write it beside the skill as `amc-media-mix-<YYYY-MM>.html`. | a filesystem |
| `slack` | Post the reallocation and its confidence as text, then upload the HTML. Slack will not render a 1920x1080 deck inline. | a connected Slack tool |
| `n8n` | POST the HTML to the configured webhook, `Content-Type: text/html`. | network access |
| `email` | Hand it to the connected mail tool. | a connected mail tool |

Confirm before the first outward send of a session, fall back to in-chat
loudly when a method is unavailable, and never substitute a different
outward channel. Naming Slack or n8n here is configuration, not deck
content — the deck names no platform but TrackIQ and Amazon.

## Non-negotiables

1. **Never say a channel caused a purchase.** AMC shows which channels
   appeared in a path, not which one did the work. Write "$X of sales came
   from paths containing a DSP impression", never "DSP drove $X". Claiming
   incrementality from co-occurrence is the one error that discredits the
   whole deck, and slide 6 carries a "what this does not say" panel for
   exactly this reason.
2. **One month per `get_amc_ntb_purchases` call, and check the row count.**
   It truncates at `limit` silently — a three-month call at `limit=100`
   returns 100 rows from the newest month and nothing from the other two,
   with no error. If rows == limit, raise it and pull again.
3. **Never sum AMC rows across months.** Each month is a separate customer
   cohort; adding them double-counts people. Compare months, and label every
   figure with the month it belongs to.
4. **Cost per new customer uses AMC new-to-brand for every channel.** AMC
   and the DSP platform disagree by 10–15% on DSP NTB. Mixing them makes
   DSP look cheaper or dearer than it is. Use AMC across the comparison
   table, platform figures on the DSP-only slide, and label both.
5. **Always label it "TACoS incl. DSP".** `get_account_overview` covers
   Sponsored only. Adding DSP is what makes this number differ from the
   console, and an unlabelled TACoS starts an argument with the client.
6. **The reallocation nets to zero** unless the client asked for a budget
   change, and it states the diminishing-returns assumption. The destination
   line's cost per new customer will rise as it scales — quote its own
   month-over-month movement and recommend moving the budget in two steps.
7. **Slide 11 is never deleted.** If the pulls were clean, it says what was
   checked and found sound. Confidence is a number out of ten, split between
   direction and magnitude — they are rarely the same.
8. **Measure the footer safe zone, do not eyeball it.** Content ends above
   y=976 on every slide. Run the script in `assets/checks.md` and require a
   negative value for all twelve. Overflow is invisible in the source and
   prints the logo on top of a sentence.
9. **Never invent a recommendation.** If the analysis does not support a
   reallocation, slide 10 says so and gives the one thing to measure next.
10. **Never print `account_id`.** Refer to the account by name or as "your
    US Seller account".

## When there is more than one TrackIQ MCP

Several can be connected at once, with identical tool names and different
brands behind them. Call `list_marketplaces` on each and match on the
returned `name` before pulling. Getting this wrong builds a correct deck for
the wrong client.

## Relationship to the other TrackIQ reports

The daily check-up and Snacks cover yesterday; the weekly recap covers the
week. This one covers a month or a quarter and is the only report that
questions the shape of the budget rather than its execution. It is the
right artifact for a QBR, a budget conversation, or the question "is DSP
worth it". It is the wrong artifact for "what happened yesterday".

## Version

`trackiq-amazon-amc-media-mix` v1.0.1 (2026-09-21).

If the user asks whether this skill is current, fetch
`https://trackiq.com/skills/registry.json`, compare the `version` field for
`trackiq-amazon-amc-media-mix`, and if it is newer, give them the download link and
the one-line changelog. Do not fetch at any other time.
