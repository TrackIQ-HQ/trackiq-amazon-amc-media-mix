# Before you send it

## 1. The footer safe zone — the one that always bites

Slides are 1920×1080 with a 96px top margin and a **104px footer reserved
for the bug and the page number only**. Content must end above y=976.

Overflow is invisible in the source and looks fine in a thumbnail: the first
sign is the TrackIQ bug printing on top of a sentence. On the build this
skill came from, four of twelve slides overflowed and two of them looked
correct until measured.

**Measure, do not eyeball.** Open the deck and run:

```js
// Read the scale off the deck itself. Recomputing it from window.innerWidth
// silently yields 0 in a hidden or zero-width viewport, and every slide then
// measures as passing.
const deck = document.getElementById('deck');
const sc = new DOMMatrixReadOnly(getComputedStyle(deck).transform).a;
if (!(sc > 0)) throw new Error('deck not laid out yet — resize the window and re-run');

const res = [...document.querySelectorAll('.slide')].map((s, i) => {
  const top = s.getBoundingClientRect().top;
  let b = 0;
  [...s.children].forEach(c => {
    if (/bug|pnum|close-lockup|title-lockup|title-kicker/.test(c.className)) return;
    b = Math.max(b, (c.getBoundingClientRect().bottom - top) / sc);
  });
  return { slide: i + 1, over: Math.round(b - 976) };
});
({ bad: res.filter(r => !(r.over < 0)), all: res.map(r => r.over) });
```

`bad` must be empty and every value in `all` must be a negative number. A
`null` or `NaN` is a failed measurement, not a pass — re-run it. Fix anything
that is not negative, and re-run; do not accept a single positive value.

To fix an overflowing slide, in this order:

1. Add `tight` to the section class — `<section class="slide tight">`. That
   alone recovers 30–60px by tightening table rows, card padding and gaps.
2. Shorten table cell text so no row wraps to two lines. A wrapped row costs
   ~33px; three of them cost a slide.
3. Add `nw` to a label cell to stop it wrapping: `<td class="lab nw">`.
4. Cut a table row or a callout sentence.

Never shrink the type below the floor (body 24px, table cells 22px, labels
19px) and never reduce the 104px reserve.

## 2. Render every slide

Screenshot all twelve. Check for clipping, overlap, a missing logo, font
substitution, and a chart label sitting on top of a bar. Do not claim visual
verification without having looked.

## 3. The number checks

- **Every figure appears once, or identically everywhere it appears.** The
  TACoS on the title slide is the TACoS on slide 3.
- **Aggregates survive the detail beneath them.** A total row equals the sum
  of the rows above it.
- **Shares sum to 100%** (99.9 and 100.1 are fine; a fudged value is not).
- **The reallocation nets to zero** unless a budget change was requested.
- **Every month is labelled.** No figure in the deck is ambiguous about
  which month it belongs to.

## 4. The honesty checks

- No sentence claims a channel *caused* a purchase. Paths show
  co-occurrence.
- Slide 11 exists and is filled in.
- Every recommendation row carries a confidence chip, and any figure that
  rests on an assumption says which.
- Nothing is quantified that the data does not support.

## 5. Brand

- Exactly three dark slides: 1, 2 and 12. All others cream.
- Bug bottom-left on every content slide, white on dark and sage on cream.
  Lockup on the title and closing slides only.
- No footnotes anywhere.
- No emoji, no vendor names other than TrackIQ and Amazon's own surfaces.

## 6. Ship

Save as `<client>-amc-path-to-purchase-<month>-<year>.html`. The logos are
base64-embedded, so the file travels on its own — no asset folder, no
network.

For a PDF, print at 1920×1080 landscape with background graphics on. The
template's print stylesheet already drops the screen scaling and breaks one
slide per page.
