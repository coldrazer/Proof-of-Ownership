# Report format and proof strength rubric

Read this file at Step 8 of [the Skill](../SKILL.md), when composing the report. Every value in the report must come from a completed step result. Cells with no verified value say "not visible", "no receipt found" or "fingerprint unavailable"; they are never filled with an estimate.

## Proof strength rubric

Rate every item with exactly one level.

| Level | Rule |
|---|---|
| Strong | At least one photo, plus a matched receipt with a printed price, plus either a legible serial number or a printed brand and model |
| Moderate | Either (a) a matched receipt, or (b) two or more photos on different dates that show a printed brand and model |
| Weak | Photos only, with no printed brand or model, or a single photo on a single date |

A receipt match that Step 5 marked low confidence counts as no receipt for this rating, and the gap list says why.

## Report layout

Use this structure and these headings in this order.

### 1. Header

- Title: Home Inventory Evidence
- Run date and mode (preparedness or loss)
- Loss mode only: loss date, affected rooms, theft yes or no
- Folders searched, exactly as verified in Step 1
- Date range searched

### 2. Summary

- Items documented, and the count at each proof strength level
- Items with a matched receipt
- Sum of printed receipt prices, listed separately per currency, with the label "documented purchase prices, not replacement value". Only matched, non-low-confidence receipts are included. If there are none, write "no documented prices".

### 3. Inventory by room

One table per room type, rooms in this order: living room, kitchen, bedroom, office, garage, other.

| # | Item | Brand and model | Serial | Condition | First seen | Last seen | Photos | Receipt | Printed price | Proof |
|---|---|---|---|---|---|---|---|---|---|---|

- Photos: the count, followed by the newest photo path.
- Receipt: merchant and purchase date, or "no receipt found".
- Loss mode: add a column "Days from last seen to loss".

### 4. Gaps to fix now

Ordered by proof strength (Weak first), then by category value order from the category reference. One line per item, with one concrete action:

- No receipt: "Find the order confirmation or warranty card and save it in your receipts folder."
- No brand or model: "Take one close photo of the label or rating plate."
- Seen in only one photo: "Take a current photo so the last-seen date is recent."
- Low-confidence receipt match: "Check that this receipt is for this item."

In loss mode, replace the action text with the evidence the insurer is likely to ask for next, and never suggest creating new evidence after the loss date.

### 5. Camera events (loss mode with theft only)

Event time, camera, topic and snapshot reference for each event from Step 7, or "no camera events found in the incident window", or "camera review not requested".

### 6. Coverage and limits

- Photos left unprocessed over the 300 photo cap
- Receipt files that failed extraction, with the reason
- Categories that returned no results
- Files whose fingerprint was unavailable

### 7. Evidence fingerprints

| File | SHA-256 | Size (bytes) | Modified |
|---|---|---|---|

Under the table, one sentence: "Recomputing these SHA-256 values later and getting the same result shows these files were not changed after this report was produced."

### 8. Notice

"This report lists ownership evidence found on your own storage and purchase prices exactly as printed on your receipts. It is not an appraisal, a replacement-cost estimate, or a guarantee that any claim will be paid."
