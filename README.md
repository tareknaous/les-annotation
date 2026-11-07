# User State Annotation

A small static site for validating Gemini-generated latent evaluation states (LES)
against human judgement. Annotators read 100 conversations from WildChat and, for
the assistant's last response, label:

1. **Instruction followed**: Yes / Partially / No
2. **Reaction type** of the user's next message: Neutral / Critique / Praise
3. **Rating**: 1–5, from the user's point of view

After saving those (they are then locked), annotators see Gemini's `Internal_thought`
for the same turn and rate how plausible it is (1–5).

The Gemini labels themselves are **not** in this repository; they stay with the
analysis code. `data.json` holds only the conversations, the user's real next
message, and the `Internal_thought` shown in step 2.

## Files

| File | What it is |
|---|---|
| `index.html`, `style.css`, `app.js` | The site |
| `data.json` | The 100 items (stratified: 20 per Gemini rating, 12 Neutral + 8 Critique/Praise each) |
| `config.js` | `SYNC_URL` for central storage (empty = browser only) |
| `apps_script.gs` | Optional Google Apps Script backend |

## Publishing on GitHub Pages

```bash
cd annotation
git remote add origin https://github.com/tareknaous/les-annotation.git
git push -u origin main
```

Create the empty repository `les-annotation` on GitHub first. Then go to
**Settings → Pages → Build and deployment**, choose *Deploy from a branch*, `main`, `/ (root)`.
The site appears at `https://tareknaous.github.io/les-annotation/` after a minute.

## How progress is saved

Annotators log in with a username (no password). Every answer is saved immediately.

- **Without `SYNC_URL`**: answers live in that browser's local storage. Logging in again
  with the same username *in the same browser* resumes where they left off. They must
  click **Export** at the end (or any time) and send you `annotations_<username>.json`.
- **With `SYNC_URL`** (recommended): answers are also written to a Google Sheet, so
  annotators can resume from any browser or device and you collect everything in one place.

### Central storage (Google Sheet, ~5 minutes)

1. Create a Google Sheet → **Extensions → Apps Script** → paste `apps_script.gs` → Save.
2. **Deploy → New deployment → Web app**. *Execute as*: Me. *Who has access*: Anyone.
3. Copy the URL ending in `/exec` into `config.js`:
   ```js
   window.SYNC_URL = "https://script.google.com/macros/s/XXXX/exec";
   ```
4. Commit and push. The status in the top bar reads **synced** once it works.

Usernames are not authenticated: anyone with the link and a username can write under it.
That is fine for a small invited group; don't share the link publicly.

## Analysis

From the project root, with either the exported JSON files in a folder or the sheet
downloaded as CSV:

```bash
python scripts/les_annotation_analyze.py outputs/les_annotation/annotations/
python scripts/les_annotation_analyze.py annotations.csv
```

It reports Gemini–human agreement next to human–human agreement (the ceiling), for all
items and separately for Neutral and explicit-reaction turns, plus Krippendorff's alpha
and the Internal_thought plausibility scores.
