# Derivations

Every figure in the deck and where it comes from. Compute these in one pass
and print the table before writing a slide — a number that appears twice
must be the same number.

## Media and revenue (slide 3)

```
sponsored_spend  = get_account_overview(view='advertising_summary').total_spend
dsp_spend        = sum(spend) over get_dsp_performance rows, state='all'
total_media      = sponsored_spend + dsp_spend
total_revenue    = get_account_overview(...).total_revenue
tacos_incl_dsp   = total_media / total_revenue
```

`get_account_overview` covers Sponsored only — its `by_channel` has sp, sb,
sbv and sd and **no DSP**. Adding DSP is what makes the TACoS here differ
from the number in the Amazon console, so the column is always labelled
"TACoS incl. DSP", never "TACoS".

## Path economics (slides 4, 5, 6)

From `get_amc_attribution_paths`, focus month only:

```
reach        = sum(path_occurrences)      # people, not impressions
purchases    = sum(purchases)
sales        = sum(sales_amount)
cost         = sum(total_cost)
ntb_share    = sum(ntb_purchases) / sum(purchases)
path_roas    = sales / cost
conv_per_1k  = 1000 * purchases / reach
```

Group twice over the same rows:

- **by path length** — 1 touch, 2 touches, 3+ touches (slide 4)
- **by channel mix** — the sorted set of distinct channels (slide 5)

The touch-ladder lift on slide 4 is `conv_per_1k(3+) / conv_per_1k(1)`.

For slide 6:

```
dsp_touched_sales = sum(sales) over mixes containing DSP
dsp_only_sales    = sales of the mix that is exactly {DSP}
dsp_in_path_sales = dsp_touched_sales - dsp_only_sales
```

`dsp_in_path_sales` is DSP alongside at least one Sponsored touch. Sponsored
Display counts as a Sponsored touch.

## Cost per new customer (slide 7)

Platform spend over **AMC** new-to-brand purchases, so every channel uses the
same NTB definition:

```
cost_per_ntb(channel) = platform_spend(channel) / amc_ntb(channel)
```

| Channel | Spend from | NTB from |
|---|---|---|
| Sponsored Products | `by_channel.sp.spend` | AMC `campaign_type = SPONSORED_PRODUCTS` |
| Sponsored Brands | `by_channel.sb.spend + by_channel.sbv.spend` | AMC `SPONSORED_BRANDS` |
| Sponsored Display | `by_channel.sd.spend` | AMC `SPONSORED_DISPLAY` |
| Amazon DSP | sum of DSP campaign spend | AMC `DSP` |

Channel ROAS uses the same channel's platform sales, not AMC sales — AMC path
sales are attributed to a path, not to a channel, and splitting them per
channel would be an invention.

## Time to conversion (slide 8)

Nine fixed buckets in order. `pct = purchases / sum(purchases)`.

The three tiles are cumulative: within one hour is buckets 1–4, more than
24 hours is buckets 8–9, more than 7 days is bucket 9.

Bar geometry: `height = (pct / max_pct) * 200`, `y = 270 - height`, label at
`y - 14`.

## The reallocation (slide 10)

The deck's only recommendation. Build it this way and no other:

1. Rank every spend line by cost per new customer, worst first.
2. Take the lines whose cost per NTB is **more than double** the account
   blended figure. Those are the candidates.
3. For each, propose a cut: to zero if the line also returns under ~1.2x, to
   a floor if it returns acceptably and only its NTB efficiency is bad.
4. Move the total into the cheapest-per-NTB line.
5. **The changes must sum to zero** unless the client asked for a budget
   change. The recommendation is a reallocation, not a budget cut.

Then the payoff, which is the number the slide is built around:

```
ntb_now    = sum over cut lines of (dollars_cut / that line's cost_per_ntb)
ntb_after  = total_moved / cost_per_ntb(destination line)
net_gain   = ntb_after - ntb_now
```

Where a cut is partial, apportion that line's NTB by spend share and say so
on the slide — it assumes NTB scales linearly inside the line, which is an
assumption, not a measurement.

State the diminishing-returns caveat every time: the destination line's cost
per NTB will rise as it scales. Quote its own month-over-month movement as
evidence and recommend moving the budget in two steps.

## Rounding

Money whole dollars with thousands separators, `$139,768`. Percentages one
decimal, `71.4%`. ROAS two decimals and a trailing x, `1.54x`. Cost per NTB
two decimals, `$21.70`. Counts with separators, `6,442`.

Compute on unrounded values and round once at print. Percentages that should
sum to 100 are allowed to show 99.9 or 100.1 — do not fudge a figure to make
them tie.
