# Validating Results

Not every result needs validation. Validate when the result is zero, looks anomalous on its face, contradicts something already established, or the user expresses doubt. When a result looks normal and nothing is suspicious, present it confidently — always with the object URL so the user can see the underlying data for themselves.

There is one exception to "only validate when something looks wrong", and it's the important one:

## Plausible but wrong — the case nothing flags

The `build_*` tools translate a sentence into a structured definition with a language model. When that translation goes slightly wrong — a URL filter instead of a named page, "clicks" instead of "rage clicks", unique sessions instead of unique users, `Checkout` instead of `checkout` — the compute still succeeds and returns a clean, believable number. There is no error, no zero, no anomaly. The result passes every check in this file and is wrong anyway.

The only defense is reading the definition before trusting the number, which is why "Confirm" sits between Build and Compute in the main workflow. Concretely, before presenting a first result from a freshly built object:

- Read the returned definition or its rendered description, and check it against what was asked: page vs. URL, the event type, exact strings and casing of custom values, the aggregation unit, the time range, the grouping dimension, and any attached segment.
- For a funnel, check step order. For a journey, check the resolved pivot label — the whole tree hangs on it.
- Name the definition in your answer, not just the number: "3,412 rage clicks on pages matching `/checkout` over the last 30 days" lets the user catch a misread you couldn't. A bare "3,412 rage clicks on checkout" hides the thing most likely to be wrong.

This isn't a reason to hedge every answer. It's a reason to look once, then answer with confidence.

## Zero results — always validate

Zero is never confidently correct without a cross-check. When a compute returns zero:

1. **Suspect the filter first.** Recompute the same event type without the page/element constraint. If the broader query returns data, the filter was wrong — say what the data actually shows.
2. **Expand the time range.** Try `last_30_days` or `last_90_days`. If data appears in a longer window, the event exists but not in the requested period — report that, it's usually the real answer.
3. **Check overall traffic.** Compute an unfiltered page-view metric to confirm the org has data at all in that window. If that's also zero, the org may have no data for the period.

A funnel that reports zero at step 1 is the same problem one level up: the entry step's filter didn't match anything, so nothing downstream can.

## Anomalous results — validate

A rate over 100%, a count that's physically implausible for the org's traffic, a step count that increases later in a funnel, or a number that contradicts something already established in the conversation — investigate before presenting, without waiting for the user to question it.

## Context already available — use it for free

If you computed a related figure earlier (total traffic, say), mention proportionality without an extra call: "4,200 rage clicks out of 1.2M page views (0.35%)." Don't fetch a denominator just to sanity-check a normal-looking number nobody questioned.

## Trends — check for discontinuities

A sharp drop to zero mid-period or a spike that never recovers deserves investigation before you draw a conclusion. Broaden the metric — remove filters, check overall traffic for the same period — to see whether the discontinuity is specific to this event or affects everything. If overall traffic also drops, it may be a data-collection gap rather than a behavior change. If overall traffic is healthy, the change is probably real: slice by dimension (`top_n` by page, by device) to find what's driving it.

## When the user expresses skepticism

"That doesn't seem right" is usually information, not noise — they know their product. Two moves, in order:

1. **Re-read the definition** and tell them what was actually measured. More often than not the disagreement is about what got built, not about the number, and this resolves it immediately.
2. **Slice by dimension.** `fullstory:update_metric` with the existing `metric_id` and a refinement like "change to top_n grouped by page" shows the distribution — which either confirms the number or reveals where the data really is.

## Presenting validation

Don't narrate every check you ran; a clean validation is invisible. Surface the work only when something was wrong and you corrected it, or when you need the user's input to decide which reading is right.
