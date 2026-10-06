# The pull sequence

Twelve calls for a three-month read. Do them in this order — the account
lookup gates everything else.

## 0. Identify the account

`list_marketplaces` first. It returns `account_id`, `marketplace`,
`account_type` and `name`.

**More than one TrackIQ MCP can be connected at once, with identical tool
names and different brands behind them.** Call `list_marketplaces` on each
connected server and match on `name` before pulling anything. Never assume
the first server is the right one.

Never print `account_id` back to the user. Refer to the account as
"your US Seller account" or by `name`.

## 1. Pick the window

- **Focus month** — the most recent complete calendar month.
- **Comparison** — the two months before it.

AMC data is monthly. If the focus month is incomplete, say so on slide 11
and use the last complete one instead.

## 2. The calls

| # | Call | Arguments | Gives you |
|---|---|---|---|
| 1 | `list_marketplaces` | — | `account_id` |
| 2 | `get_amc_attribution_paths` | focus + both comparison months, `min_occurrences=5`, `limit=300` | the path table — slides 4, 5, 6 |
| 3–5 | `get_amc_ntb_purchases` | **one call per month**, `limit=500` | NTB by campaign — slides 3, 7 |
| 6 | `get_amc_time_to_conversion` | focus month, `group_by='position'` | the nine buckets — slide 8 |
| 7–9 | `get_account_overview` | one call per month, `view='advertising_summary'` | sponsored spend and sales by channel, total revenue, TACoS |
| 10–12 | `get_dsp_performance` | one call per month, `dimension='campaign'`, `state='all'` | DSP spend, sales, DPV, NTB — slides 3, 7, 9 |

`get_amc_ntb_asins` is optional. Pull it only if the client asks which
products recruit new customers; it does not feed any slide in this deck.

## 3. Five things this data does that will catch you out

1. **`get_amc_ntb_purchases` truncates silently at `limit`.** A three-month
   call with `limit=100` returns 100 rows from the newest month and nothing
   from the other two, with no error and no flag. Pull **one month per
   call**, and if the row count equals the limit, raise the limit and pull
   again.
2. **AMC returns one row per month. Never sum across months.** Each row
   carries its own `month_start`/`month_end`, and each month is a distinct
   customer cohort — adding them double-counts people. Compare months, state
   which month a figure belongs to.
3. **`get_amc_time_to_conversion` returns purchase counts only.** Its
   `total_product_sales`, `ntb_total_product_sales` and units fields all come
   back zero. The distribution is sound; a revenue split by time bucket is
   not available. Say so on slide 11 rather than showing zeros.
4. **AMC and the DSP platform disagree on DSP new-to-brand** — different
   attribution windows, typically a 10–15% gap. Use AMC for every
   cross-channel comparison so denominators match, use the platform figures
   on the DSP-only slide, and label both.
5. **`get_dsp_performance` defaults to `state='ACTIVE'`.** Pass `'all'`
   explicitly or a paused campaign's spend disappears from the month.

## 4. Large results

`get_amc_attribution_paths` and `get_amc_ntb_purchases` can exceed what a
single tool result will carry. When the runtime spills them to a file,
aggregate with a short script rather than reading the whole thing back.
When there is no filesystem, narrow the window to the focus month and raise
`min_occurrences` to 10 — the shape of the finding survives.

## 5. Reading the `path` field

`get_amc_attribution_paths` returns `path` as a JSON-encoded string of the
touch sequence, e.g. `"[[1, SP-Others], [2, DSP]]"`. It is not valid JSON —
the channel tokens are unquoted. Extract the tokens in order, then fold each
to its channel: anything starting `SP` → SP, `SB` → SB, `SD` → SD, anything
containing `DSP` → DSP.

Path length is the number of touches. The channel mix is the *set* of
distinct channels, sorted, joined with `+` — so `SP > DSP > SP` is a
three-touch path with mix `DSP+SP`.
