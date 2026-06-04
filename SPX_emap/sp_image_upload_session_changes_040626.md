# SP Image Upload Mockup — Session Changes
_4 Jun 2026 | gareth.wangjc@shopee.com_

File edited: `SPX Website emap Enhance Point Info's Readability v6.html`

---

## Background

Dev (Xie Zhengcai) confirmed that the workaround for 1 GB uploads is to use the browser's native folder selection API to read local files directly — no need to zip. Drag-and-drop is not feasible with this approach.

---

## Change 1 — Upload zone: folder instead of zipped folder

**Step 1 upload zone updated:**

| Element | Before | After |
|---|---|---|
| Icon | 📦 | 📁 |
| Title | Images folder (.zip) | Images folder |
| Hint text | "Zip up all the images you want to upload into one folder, then select it here. JPG, PNG and HEIC files only." | "Select the folder containing your images. JPG, PNG and HEIC files only. Do not move or delete files from the folder while the upload is in progress." |
| Button | Choose file | Choose folder |
| File pill | 📦 images_batch_apr25.zip | 📁 images_batch_apr25 |

---

## Change 2 — Removed "Unsupported file formats" section from Step 2

Removed the entire `if(hasBad)` block that showed a table of unsupported files (e.g. `.webp`, `.bmp`, `.gif`). Rationale: unsupported files will surface naturally in Step 4 (upload complete summary), so showing them again in Step 2 is redundant.

Also cleaned up:
- Divider between the unsupported block and the assign section
- Empty-state condition updated: `if(!hasBad&&!hasAssign)` → `if(!hasAssign)`
- Footer note counting skipped files removed

---

## Change 3 — Progress bar on folder selection

When user clicks "Choose folder", the upload zone is replaced by a progress bar showing the folder read in progress.

**Progress section shows:**
- Folder name (📁 images_batch_apr25)
- Status label (`Uploading...` → `Upload complete` in green when done)
- File counter (`X / 25`) right-aligned on the same row as status
- Animated blue progress bar
- Current filename being read (`Reading: filename.jpg`)

**On completion:**
- Counter shows `25 / 25`
- File line shows `✓ All files read`
- Footer populates: `25 images detected across 4 Points`
- "Next →" button becomes active (disabled during upload)

Simulation runs at ~160ms per file (~4 seconds for 25 files).

---

## Change 4 — Step 2: "← Back" → "Cancel"

**Rationale:** By Step 2, the folder read has already happened. "Back" implies undoing the upload, which isn't the case. The only meaningful action is to abandon the whole operation.

- Button label: `← Back` → `Cancel`
- `onclick`: `goTo("upload")` → `closeModal()`
- Inline warning text added below the buttons: *"Cancelling will discard your upload — you'll need to re-select the folder."*

---

## Change 5 — Inline cancel warning on Step 3

Step 3 (Review mapping) keeps `← Back` for valid navigation back to Step 2. Added the same inline warning pattern below its buttons:

*"Cancelling will discard your upload and all image assignments."*

**Rationale:** If a user uploaded a large folder (significant wait time), they should know at every step that closing the modal loses everything — without needing a confirmation popup (popup-in-popup pattern avoided).

---

## Open questions / next session

- Should there be an estimated time remaining during the folder read?
- File type breakdown on completion (e.g. `23 JPG · 2 PNG`)?
- Cancel/pause during active upload?
