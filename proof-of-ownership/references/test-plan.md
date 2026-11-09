# Test plan

Read this file when testing [the Skill](../SKILL.md) on a UGREEN AI NAS. All test data is synthetic: photos of objects only, with no people, faces, addresses or real account details, and every test receipt is visibly marked SAMPLE TEST DATA.

## Synthetic test data set

Create these folders on the test NAS before the session:

- **Photos/Home**: about 60 photos taken across three "years" (set capture dates in metadata to three different years), covering a living room, office, kitchen and garage. Include:
  - a television with the brand visible on the bezel, in 5 photos across 3 years
  - a laptop with its model sticker visible in one close-up
  - a road bicycle in 2 photos in the garage
  - an acoustic guitar in 1 photo only
  - a sofa with no visible brand, in 6 photos
  - one close-up of a rating plate showing a Serial and Model No label
  - two photographed paper receipts on a table
  - 10 photos with no valuables (landscapes, food), to confirm they are ignored
- **Documents/Receipts**: 6 receipts as PDF or image, each marked SAMPLE TEST DATA:
  - 4 that match items above (television, laptop, bicycle, sofa)
  - 1 whose purchase date is later than the first photo of its item (low-confidence match)
  - 1 for an item that does not appear in any photo (unmatched receipt)
- **Documents/Empty**: an empty folder, for the empty-result test

## Core test 1: preparedness inventory

Trigger: "Make an insurance inventory of my stuff from my photos in Photos/Home and receipts in Documents/Receipts."

Pass when:

- Steps 1 to 6 all return completed.
- The report lists the television, laptop, bicycle, guitar and sofa as separate items, and lists none of the 10 photos without valuables.
- The television is rated Strong (photos, receipt, printed brand), its first-seen and last-seen dates span the three years, and its receipt price is copied exactly as printed.
- The guitar is rated Weak and appears in "Gaps to fix now".
- The low-confidence receipt is flagged and does not upgrade its item to Strong.
- The unmatched receipt appears in the unmatched receipt list.
- Every evidence file has a SHA-256 value, and recomputing one by hand gives the same value.
- The save step asks for a folder and waits; the file is reported only after save-text returns a verified path.

## Core test 2: loss mode after a burglary

Trigger: "We were burgled on the date of this test, the office and garage were hit, rebuild a list of what was in those rooms."

Pass when:

- The report contains only office and garage items (laptop, bicycle and anything else there).
- Each item shows days from last seen to the loss date.
- With cameras present and a test window given, Step 7 returns events and snapshots and never names or describes a person. With no cameras, the report says camera review was not requested.

## Edge case: no results and no guessing

Trigger with the photo folder set to Documents/Empty and a date range that contains no photos.

Pass when the Skill reports that no valuables were found in the authorized folder, stops before Step 3, saves nothing, and does not search any other folder. A second check: on the sofa (no visible brand), the brand and model cell must read "not visible", never a guessed brand.

## Additional checks

- Cancel during Step 3: the Skill stops, reports finished steps, and saves nothing.
- Decline the save: the full report is shown in the conversation and nothing is written.
- Decline the album: no album is created.
- Over 300 candidate photos: the report lists the unprocessed count under Coverage and limits.
