# 0006. Model classification writes into a closed vocabulary, and out-of-vocabulary answers are dropped

- **Status:** Accepted
- **Date:** 2026-09-05
- **Driven by:** "Add review sentiment and the themes underneath it" (2026-08-05)

## Context

The sentiment card needs to say *what* reviews are negative about, not just how many.
Asked to name the theme of a review in free text, the model returns "sizing" for one,
"size" for the next and "fit issues" for a third. The counts then add up to nothing,
and the one number the owner can act on — "most complaints are about sizing" —
never appears.

## Decision

Themes come from a fixed list of nine (`sentiment.THEMES`), with `"other"` as the
escape hatch so the model has somewhere to put a review that does not fit rather than
bending it into a theme that does not.

Anything the model returns that is not in the vocabulary — a made-up theme, an
invalid sentiment label, a row number out of range — is dropped, not stored. A
dropped review is picked up on the next classification run; a stored bad label would
quietly corrupt the counts forever.

Classification runs at `temperature=0.0`, is written back to the review row so it
happens once, and de-duplicates identical review texts before sending (≈23 distinct
sentences across 140 reviews).

## Alternatives considered

- **Free-text themes, clustered afterwards** — adds an embedding-and-clustering step
  whose output still is not stable between runs.
- **Store whatever the model says** — one bad label per run and the "top complaint"
  number slowly becomes fiction.
- **Fail the whole batch on one bad row** — one malformed row should not cost the
  other fourteen.

## Consequences

- The theme counts are trustworthy by construction — every stored label is one of
  nine known values.
- Adding a theme means editing `THEMES` and re-running classification; the vocabulary
  is a deliberate, reviewable list, not an emergent one.
- The same shape (closed set, drop the rest) is the right default for any future
  model-classification feature — alert types, product categories.
