# Classification Heuristics

Read this file when you encounter items that are difficult to classify. These
heuristics are derived from common JD patterns and should be applied after
consulting the JDex.

---

## Decision Framework

For every inbox item, answer these questions in order:

### 1. What is this thing?

Categorize the item by type:

| Type | Examples | Signals |
|------|----------|---------|
| **Record** | Receipt, policy, certificate, photo | Static; reference value; no action needed |
| **Task source** | Meeting notes, voicemail, email | Contains commitments, deadlines, or action items |
| **Both** | Buyer meeting notes with attached order | Has reference value AND embedded actions |
| **Stale** | Old reminder, completed item, duplicate | No longer relevant; outdated |

### 2. What domain does this belong to?

Map the item's subject matter to an area. Use keyword matching against the
JDex's area descriptions:

| Content about... | Likely area pattern |
|---|---|
| House, property, car, utilities | Home / property |
| Family, kids, relationships, custody | Family / relationships |
| Money, taxes, insurance, legal, banking | Money / legal / finance |
| Doctor, prescriptions, fitness, therapy | Health / wellness / medical |
| Work projects, employer, career | Work system or career area |
| Trips, flights, hotels, itineraries | Travel |
| Hobbies, games, music, creative projects | Hobbies / interests / creative |
| School, courses, grades, assignments | Academics / school |

### 3. What collection does this join?

Within the area, match to a category. The question is: "What other things
would I look for alongside this item?" A car insurance renewal belongs with
other insurance items, not with other car items — if that's how the user's
system is structured. Always defer to the JDex.

### 4. Does a specific ID already exist?

Scan the category's IDs for an exact or close match. A file about "health
insurance claim for NYC trip" belongs in the health insurance ID, not the NYC
trip ID — unless the user has set up their system differently.

---

## Ambiguity Patterns and Resolutions

### Pattern: Item fits two categories equally well

Example: A receipt for car repair. Is it "car maintenance" or "money/expenses"?

Resolution: Check the JDex for which category already holds similar items.
If both could work and neither has precedent, ask the user. Their answer
establishes the pattern for future items.

### Pattern: Item fits one system's area but was captured in another

Example: A work-related note found in the personal inbox.

Resolution: Route to the correct system. If the correct system has an inbox,
move it there. If you can classify it confidently, file it directly in the
correct system's ID.

### Pattern: Item has no matching ID and the category is sparse

Example: First-ever dentist receipt, and there's a "Health" category but no
"Dental" ID.

Resolution: Propose creating a new ID. The category exists, so the structural
question is already settled — you just need a new slot. Confirm with the user
and assign the next sequential number.

### Pattern: Item has no matching category

Example: User starts a new hobby that doesn't fit any existing category.

Resolution: **Do not create a new category.** This is a structural change that
affects the system's architecture. Move the item to `00.08 Someday` or
`00.04 Needs review` and flag it for the user to decide where it belongs. They
may need to create a new category, repurpose an existing one, or rethink the
area structure.

### Pattern: Item is a task with no associated record

Example: A sticky note that says "Call dentist Monday."

Resolution: This is a pure task. Don't file it as a record — extract the task
into `00.02 Tasks` with appropriate context and deadline, then archive or
delete the original inbox item.

### Pattern: Item is a photo or scan with minimal metadata

Example: `IMG_4521.jpg` — a photo of a shelf in a store.

Resolution: Look at the image. If you can identify the context (e.g., it's a
shelf placement photo for a specific account), classify based on content. If
you can't determine what it is, ask the user. Don't guess.

### Pattern: Item references multiple IDs or categories

Example: Meeting notes that discuss three different accounts.

Resolution: File the complete document in the most relevant location (often
the primary topic or the account that generated the meeting). Extract
individual action items as tasks with references to their respective IDs.
Don't split the document into pieces.

---

## Filename Inference

When inbox items have unhelpful filenames (e.g., `scan001.pdf`, `IMG_4521.jpg`,
`Untitled.md`), rename them during filing:

1. Read or view the content to determine what the item is
2. Construct a descriptive name following the `YYYY-MM-DD Description.ext`
   convention
3. Use the item's inherent date, or today's date if none is apparent

Good renames:
- `scan001.pdf` → `2026-01-15 Property tax statement.pdf`
- `IMG_4521.jpg` → `2026-02-01 Shelf placement photo Acme.jpg`
- `Untitled.md` → `2026-02-08 Buyer meeting notes Red Door.md`

---

## Confidence Calibration

Use these examples to calibrate your confidence levels:

### High Confidence (act without asking)

- A PDF titled "2026 W-2 Form" in a system that has `31.xx Taxes` → file it
- A note starting with "Met with Jordan at Acme" in a work system with an
  Acme account folder → file it in the Acme account
- A photo named "passport scan" in a personal system with `11.xx Identity
  documents` → file it

### Medium Confidence (recommend and confirm)

- A receipt from a restaurant — could be "dining/entertainment" or "business
  expense" depending on context
- A note about "Project X" when there's both a work system and a personal
  projects area
- An insurance document when there are separate "auto insurance" and "general
  insurance" IDs

### Low Confidence (present options and ask)

- A cryptic filename with no readable content
- An item whose subject matter doesn't match any existing area
- Content that spans multiple domains with no clear primary focus
- Anything in a language or notation you don't fully understand

---

## Processing Order

When an inbox has many items, process in this order:

1. **Quick wins first**: Items with high-confidence classification. This clears
   volume and builds momentum.
2. **Related items together**: Group items that reference the same account,
   project, or topic. This reduces context-switching.
3. **Ambiguous items last**: These require user input and are slower to process.
   Having cleared the easy items first reduces the user's decision fatigue.
