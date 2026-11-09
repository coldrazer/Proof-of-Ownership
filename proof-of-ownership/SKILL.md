---
name: proof-of-ownership
description: Claim-ready home inventory of household valuables built from the photos, videos and receipts already stored on the NAS. Each item is listed by room with dated photo evidence, a last-seen date, the matched receipt price when one exists, and a SHA-256 fingerprint of every evidence file, followed by a list of items that still lack proof. Use when the user wants to document belongings for insurance, prepare for a fire, flood or burglary, or rebuild a list of what they owned after a loss.
---

# Goal

After a fire, flood or burglary, most people cannot prove what they owned. The proof usually already exists on their NAS: years of everyday photos that happen to show the TV, the bikes, the guitar and the laptop, plus scanned or photographed receipts. This Skill turns that scattered evidence into one claim-ready inventory.

The user-visible result is a Markdown report, saved on the NAS only after the user confirms the folder, that contains:

1. A room-by-room table of valuables, one row per physical item, with item type, brand and model (only when visible), serial number (only when legible), photo evidence paths, first-seen and last-seen capture dates, matched receipt (merchant, date, printed price) and a proof strength rating.
2. A "gaps to fix now" list: items that have photos but no receipt, items seen only once, and items with no clear brand or model.
3. An evidence fingerprint appendix with the SHA-256, size and modified time of every evidence file, so the user can later show the files were not altered after the report date.
4. In loss mode for a theft, an optional list of camera events and snapshots inside the incident window.
5. Optionally, a photo album that gathers every evidence photo in one place.

The last-seen date is the key differentiator: an insurer asks not only "did you own it" but "did you still own it when the loss happened". The Skill reports the newest photo of each item for that reason.

Supported scope: photos, videos and documents the user explicitly authorizes inside folders they name. The Skill never estimates market or replacement value, never invents a brand, model or serial number, never identifies people in photos or camera footage, and never sends files outside the NAS.

Two modes:

- **Preparedness mode** (default): build the inventory before anything happens, and show the user which proof is missing while it can still be fixed.
- **Loss mode**: the user has had a loss. Scope the inventory to the affected rooms, report the last-seen date of each item relative to the loss date, and optionally collect camera events for a theft window.

# Required input

Collect these from the user before any delegation. Ask in one short message, offering the defaults in brackets.

1. **Mode**: preparedness or loss. [preparedness]
2. **Photo folders**: the folder names that hold the family photos and videos to search. At least one is required. Use the names exactly as the user gives them.
3. **Receipt folders**: the folder names that hold scanned receipts, order confirmations or warranty documents. Optional; when none is given, the Skill still searches photos for photographed receipts.
4. **Date range**: the capture dates to consider. [all dates]
5. **Categories**: which item categories to look for, chosen from [references/item-categories.md](references/item-categories.md). [all categories]
6. **Loss mode only**: the loss date, the affected rooms, and whether the loss was a theft. For a theft, ask whether the NAS has cameras and, if so, the incident start time, end time and timezone, and which cameras or rooms to check. Camera review is optional and only runs when the user says yes.
7. **Save folder**: asked later, at the save step, not up front.

Do not ask for account names, passwords, policy numbers or any credential. Do not pre-collect parameters owned by a child expert.

# Generation-time capability evidence

- `file-workflow` → `file-agent`
- `ug-filesystem` → `file-agent`
- `ug-photo` → `photo-agent`
- `ug-document` → `document-agent`
- `ug-surveillance` → `security-monitor-agent`
- `ug-knowledge` → `knowledge-agent`

Actions relied on, all confirmed in the public capability catalog:

| Step | CLI and actions | Effect | Approval |
|---|---|---|---|
| 1 | file-workflow get-user-dir, ug-filesystem stat, ug-filesystem ls | read | none |
| 2 | ug-photo object-albums, ug-photo search-picture | read | none |
| 3 | ug-photo meta, ug-photo vision-ocr, ug-photo vision-vlm | read | none |
| 4 | ug-filesystem find | read | none |
| 5 | ug-document extract (schema receipt), ug-document reconcile | read | none |
| 6 | ug-filesystem sha256sum, ug-filesystem stat | read | none |
| 7 | ug-surveillance event-timeline, ug-surveillance event-evidence | read | none |
| 8 | ug-knowledge save-text | state-change | workflow |
| 9 | ug-photo create-album | state-change | workflow |

Steps 1 to 7 only read. Steps 8 and 9 are the only steps that change NAS state, and both run only after the user confirms.

# Workflow

General rules for every step:

- Delegate each step to the single expert listed in the capability evidence. The parent Skill only delegates; it never runs a CLI action itself.
- When assembling each instruction, copy the real values verbatim from the completed result of the upstream step that produced them: verified folder paths, photo paths, item records, receipt paths. Never pass a value that an upstream step did not actually return, and never refer to "the folder from before" without writing the real path.
- Every step that searches or lists has a defined empty result. When a step returns no results, report that plainly. Do not widen the folders, the dates or the categories on your own.
- Keep the user informed with one short progress line per step, for example "Step 2 of 7: found 214 candidate photos across 9 categories."

## Step 1. Verify the folders the user authorized

```text
delegate --target file-agent --instruction 'Resolve the current user home directory with file-workflow get-user-dir. Then check each folder in this list, copied verbatim from the user: the photo folders and the receipt folders the user named in this conversation. For each folder use ug-filesystem stat and ug-filesystem ls with a limit of 50 entries to confirm it exists and is readable. Read only and change nothing. Return, for each folder, the resolved real path, whether it is a photo folder or a receipt folder, and the number of entries seen. Mark any folder that does not exist or cannot be read as not found. Do not guess, search for or substitute any other folder.'
```

Passes forward: the verified real paths, split into photo folders and receipt folders.

Empty result: if no photo folder is verified, tell the user which names were not found, ask for corrected names, and stop. If only receipt folders are missing, tell the user and continue; receipt matching will rely on photographed receipts only.

## Step 2. Find candidate photos of valuables and photographed receipts

Before assembling this instruction, read [references/item-categories.md](references/item-categories.md) and copy the search keywords for the categories the user chose.

```text
delegate --target photo-agent --instruction 'Work only inside these photo folders, copied verbatim from the completed Step 1 result: the verified photo folder paths. Restrict capture time to the date range the user gave, or use all dates if none was given. First list existing object albums with ug-photo object-albums. Then, for each category keyword in this list copied from the category table, run ug-photo search-picture with ai-object-name set to that keyword and dir set to each verified folder: the chosen category keywords. Separately, run ug-photo search-picture with ocr-contains set to each of Serial, S/N, Model No and IMEI to find photos of product labels and rating plates, and with ocr-contains set to each of Receipt, Subtotal, Order Total and Invoice to find photographed receipts. Return at most 40 photos per category keyword, choosing the newest and the oldest first. For every photo return the real file path, capture time, location if present, and the keyword that matched. Return the label photos and the receipt photos as two separate lists. Read only. If a keyword returns nothing, record it as no results and continue with the next keyword; do not change the folders, the dates or the keywords.'
```

Passes forward: candidate item photos grouped by keyword, label photos, receipt photos.

Empty result: if every keyword and both OCR searches return no results, tell the user that no valuables were recognized in the authorized folders for that date range, suggest they check that the folders contain indexed photos, and stop. Do not fall back to other folders.

## Step 3. Identify each physical item and its proof

Before assembling this instruction, copy the vision prompt from [references/item-categories.md](references/item-categories.md).

```text
delegate --target photo-agent --instruction 'Use only these photo paths, copied verbatim from the completed Step 2 result: the candidate item photos and the label photos. Process at most 300 photos in total; if there are more, keep the newest and the oldest photo for each keyword first and report how many photos were left unprocessed. For each photo, read the capture date and camera with ug-photo meta, read any visible label text with ug-photo vision-ocr, and describe it with ug-photo vision-vlm using this exact prompt: the vision prompt copied from the category reference. Then group photos that show the same physical item into one item record, using item type, brand, model, colour, room and visible serial text. For each item record return: item type, brand and model only when they are printed on the item or a label, serial number only when it is legible in the OCR text, room type, condition, every photo path that shows it, the earliest capture date, the latest capture date, and an identification confidence of high, medium or low. Never guess a brand, model or serial number that is not visible. Read only. If no photo shows an identifiable valuable item, return no results.'
```

Passes forward: the list of item records.

Empty result: if no item record is produced, tell the user that the photos found did not show identifiable valuables, list the counts searched, and stop.

## Step 4. List receipt documents

Run this step only when Step 1 verified at least one receipt folder. Otherwise go straight to Step 5 with the receipt photos from Step 2.

```text
delegate --target file-agent --instruction 'Work only inside these receipt folders, copied verbatim from the completed Step 1 result: the verified receipt folder paths. Use ug-filesystem find with type documents, then again with type image, to list purchase receipts, order confirmations, invoices and warranty documents. Restrict modified time to the date range the user gave, if any. Return the real file path, size and modified time of each file, at most 500 files in total. Read only and change nothing. If no files are found, report no results.'
```

Passes forward: receipt document paths.

Empty result: tell the user no receipt documents were found in those folders and continue to Step 5 with receipt photos only. If there are also no receipt photos, omit Step 5 and mark every item as having no receipt.

## Step 5. Read receipts and match them to items

```text
delegate --target document-agent --instruction 'Use only these files, copied verbatim from the completed upstream results: the receipt document paths from Step 4 and the receipt photo paths from Step 2. Run ug-document extract with schema receipt to read the merchant, purchase date, each line item and its printed price, and keep the coverage and failures lists. Then run ug-document reconcile to match the item records below, copied verbatim from the completed Step 3 result, against the extracted receipt lines, using item type, brand, model and date. A match is low confidence when the purchase date is later than the earliest photo date of the item. For each item return the matched receipt path, merchant, purchase date, the line text exactly as printed, the price exactly as printed with its currency, and the match confidence, or return unmatched. Also list receipts that matched no item. Never estimate, convert, round or total a price. Read only. Item records: the item records from Step 3.'
```

Passes forward: per-item receipt matches, unmatched items, unmatched receipts, extraction failures.

Empty result: if extraction returns no readable receipts, report how many files failed and why, mark every item as having no receipt, and continue.

## Step 6. Fingerprint every evidence file

```text
delegate --target file-agent --instruction 'For each file path in this list, copied verbatim from the completed Step 3 and Step 5 results: every photo path in the item records and every matched receipt path. Run ug-filesystem sha256sum and ug-filesystem stat. Return the path, SHA-256 value, size in bytes and modified time for each file. Read only and change nothing. If a file is missing or cannot be read, list it as failed with the reason and continue with the remaining files.'
```

Passes forward: the fingerprint table.

Empty result: not applicable, since Step 6 only runs on files that earlier steps returned. Failed files are listed in the report appendix as "fingerprint unavailable".

## Step 7. Camera events for a theft window (loss mode, theft, cameras confirmed only)

Run this step only when the user chose loss mode, said the loss was a theft, said the NAS has cameras, and gave an incident window. Otherwise omit it and say so in the report.

```text
delegate --target security-monitor-agent --instruction 'List camera events between the incident start time and end time the user gave, copied verbatim with their timezone, using ug-surveillance event-timeline with topic Human and topic Vehicle, limited to the cameras or rooms the user named. For at most 10 events, retrieve snapshots with ug-surveillance event-evidence using evidence-type snapshot and purpose user_review. Return each event time, camera, topic and snapshot reference. Report only events that exist. Do not identify, name or describe any person, face or vehicle plate. If there are no events in the window, report no results.'
```

Passes forward: event list with snapshot references.

Empty result: report "no camera events found in the incident window" in the report. Never substitute events from another time window.

## Step 8. Compose the report, then save it after confirmation

Compose the report yourself from the verified results of Steps 2 to 7, following [references/report-format.md](references/report-format.md) exactly. Rate each item with the proof strength rubric in that file. In loss mode, add the days between each item last-seen date and the loss date.

Show the user a short summary first: number of items, how many are Strong, Moderate and Weak, how many have receipts, and the top five gaps. Then ask which folder to save the full report in. This is a waiting_user point: do nothing further until the user names a folder or declines.

When the user names a folder:

```text
delegate --target knowledge-agent --instruction 'Save the inventory report text that follows this instruction as a new Markdown file in the folder the user confirmed in this conversation, copied verbatim, using ug-knowledge save-text with filename-hint Home Inventory Evidence followed by the date of this run. Do not overwrite or modify any existing file. Return the verified output file path and the content size. Report text: the full report composed from the verified results.'
```

If the user declines to save, show the full report in the conversation instead and end with completed status for the inventory.

## Step 9. Optional evidence album

After the report is saved, offer once: "Do you want an album that gathers all evidence photos in one place?" Run only on a clear yes.

```text
delegate --target photo-agent --instruction 'Create a photo album named Home Inventory Evidence followed by the date of this run, using ug-photo create-album with add-item set to each photo path in the item records, copied verbatim from the completed Step 3 result. Add only those photos. Do not move, rename or delete any photo. Request approval through the normal workflow before creating the album. Return the real album UUID and the number of photos added.'
```

# Completion and failure

Report a business result only after the delegated expert returns completed with real data. Handle every delegation status as follows.

- **completed**: use the returned data as the only input for downstream steps. Pass forward only what the expert actually returned. If the expert reports partial coverage (for example, photos left unprocessed over the 300 cap, or receipts that failed extraction), carry those counts into the report under "Coverage and limits".
- **failed**: stop every step that depends on that result. Steps that do not depend on it may continue: a failed Step 4 or Step 5 still allows a report without receipts; a failed Step 6 still allows a report marked "fingerprints unavailable"; a failed Step 7 still allows a report marked "camera review failed". A failed Step 1, Step 2 or Step 3 stops the Skill, because there is no verified item list. Tell the user which step failed, the reason the expert gave, and what was already verified.
- **cancelled**: stop at once. Tell the user which steps finished and which did not. Do not save anything. Offer to resume from the first unfinished step.
- **waiting_user**: relay the expert question to the user word for word, wait for the answer, and pass it back. Do not answer on the user's behalf.
- **waiting_approval**: tell the user what is waiting for approval (the report save in Step 8, or the album in Step 9) and wait. Do not report the file or album as created until the expert returns completed with a verified output path or album UUID.

Empty and partial results:

- No verified photo folder, no candidate photos, or no identifiable item: report the no results outcome clearly and stop. Do not search other locations.
- Items with no receipt are not an error. They are listed under "gaps to fix now" with a practical suggestion, such as exporting the order confirmation from the store account or photographing the serial label.
- Never fill a missing brand, model, serial number, date or price with a guess, an average or a typical value. Leave the cell as "not visible" or "no receipt found".

Final message to the user on success: the number of items documented, the proof strength split, the saved report path returned by Step 8, the album UUID if Step 9 ran, and the three most valuable gaps to fix. Add one line stating that the report documents ownership evidence and printed purchase prices only; it is not an appraisal or a guarantee that a claim will be paid.

Reference files, read only when the step that links them says so:

- [references/item-categories.md](references/item-categories.md): category keywords, label and receipt OCR terms, and the vision prompt.
- [references/report-format.md](references/report-format.md): report layout and the proof strength rubric.
- [references/test-plan.md](references/test-plan.md): core tests, edge cases and the synthetic test data set.
- [references/disclosures.md](references/disclosures.md): third-party components, AI tools used, and data handling.
