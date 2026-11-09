# Item categories, search terms and vision prompt

Read this file while assembling the instructions for Step 2 and Step 3 of [the Skill](../SKILL.md). Copy only the rows for the categories the user chose. When the user chose all categories, copy every keyword.

## Category keywords

Each keyword is passed one at a time to `ug-photo search-picture` as the `ai-object-name` value. Keywords are plain English object names because they match the object labels used by the photo index. If a keyword returns no results on a device, that is recorded as no results; do not invent replacement keywords at runtime.

| Category | Keywords for ai-object-name | Why it matters for a claim |
|---|---|---|
| Electronics | television, laptop, computer, monitor, tablet, phone, camera, game console, speaker, headphones | Highest claim frequency; models and serials are usually printed on the device |
| Appliances | refrigerator, washing machine, microwave, oven, coffee machine, vacuum cleaner | High replacement cost, often under-reported |
| Furniture | sofa, bed, table, chair, desk, wardrobe | Bulk of contents value in most homes |
| Jewelry and watches | watch, ring, necklace, jewelry | Small, high value, often sub-limited by policies, so photos matter most |
| Musical instruments | guitar, piano, keyboard, drum, violin | Brand and model drive value |
| Sports and outdoor | bicycle, skis, surfboard, golf club, tent | Bicycles are among the most stolen items |
| Tools and garage | drill, saw, lawn mower, tool box | Usually forgotten until after a loss |
| Art and collectibles | painting, sculpture, vase | Needs photo evidence of condition |

## Label and serial OCR terms (Step 2)

Passed one at a time to `ug-photo search-picture` as the `ocr-contains` value, to find photos of rating plates, box labels and stickers:

- Serial
- S/N
- Model No
- IMEI

## Photographed receipt OCR terms (Step 2)

Passed one at a time as the `ocr-contains` value, to find receipts the user photographed instead of scanning:

- Receipt
- Subtotal
- Order Total
- Invoice

## Vision prompt (Step 3)

Use this exact text as the `prompt` value for `ug-photo vision-vlm`. It is written to stop the model from guessing.

> List each valuable household item that is clearly visible in this photo. For each item give: item type; brand and model only if they are printed on the item, its screen or a label in the photo, otherwise write not visible; colour; approximate condition as new, good, worn or damaged; and the type of room it is in. Do not guess brands from shape or style. If no valuable item is clearly visible, answer with the single word none.

## Grouping guidance for Step 3

Photos are grouped into one physical item when they share item type and room and at least one of: the same legible serial number, the same printed brand and model, or the same distinctive colour and placement across photos taken in the same home. When in doubt, keep two records and mark both as medium confidence; a duplicate row is safer for a claim than two items merged into one.
