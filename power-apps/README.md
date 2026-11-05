# Power Apps — Hero Page (branch: `power-apps-hero-page`)

This branch adds a hero page for the Probationary Firefighter Training
Record to an **already-built Power Apps app**, as its new main screen.
It does **not** change the live website on `main` — nothing here goes
live until this branch is reviewed and merged.

Live website (unchanged): https://forgedreams-ai.github.io/probationary-training-record/
Repo: ForgeDreams-AI/probationary-training-record

## What is in this branch

| File | What it is |
|---|---|
| `power-apps/hero-page.html` | The hero page itself, standalone, styled to match the website (cream paper, red accent). Title, due line, three big buttons: Open my training record / View this month's report / My saved record in OneDrive, plus the three privacy points from the site. |
| `power-apps/formulas.txt` | Exact Power Apps expressions to paste in: hero screen buttons (`Launch(...)`), making the hero screen first, and the OneDrive file list for the upload screen. |
| `power-apps/visible-copy-patch.md` | The one code change that makes the upload part work: also save a visible copy of the record to `Probationary Training Record/training-record.json` in OneDrive, so Power Apps can list it. Includes the exact JS snippet and the extra Graph permission needed. |

## How the hero page gets into Power Apps

Power Apps canvas apps do not run HTML/JS files directly, so there
is no "drop this file in" step. Do this instead (about 10 minutes):

1. In Power Apps Studio, open the already-built app.
2. **Add a new screen** — name it `HeroScreen`.
3. Rebuild the layout from `hero-page.html` with native controls:
   title label, due label, and three buttons. Copy the wording
   straight from the file so it matches the website.
4. Set the buttons' `OnSelect` from `formulas.txt`:
   - Open my training record → `Launch("https://forgedreams-ai.github.io/probationary-training-record/")`
   - My saved record in OneDrive → `Launch("https://onedrive.live.com/")`
   - Upload my record → `Navigate(UploadScreen)`
5. **Make it the main screen:** move `HeroScreen` to the top of the
   screen list. Power Apps opens the first screen in the list.
6. Save and publish the app. The hero page is now the first thing
   people see.

Do **not** paste `hero-page.html` into an HTML text control — Power
Apps strips scripts from those, and the buttons will not work.
The HTML file here is the design reference and the standalone
web version; the Power Apps screen is the native rebuild of it.

## How "already there to choose from" works

Today the website saves each record to the firefighter's OneDrive
**App Folder** as `training-record.json`. That folder is private and
correct, but Power Apps cannot reliably browse to it — so a user
going to upload would have to hunt for the file.

With this branch's plan:

1. Apply `visible-copy-patch.md` (one extra save in `index.html`):
   every save also writes a copy to a normal, visible folder,
   `Probationary Training Record/training-record.json`, in the same
   OneDrive. The private App Folder copy stays as the real record.
2. In Power Apps, add the **OneDrive** connector and bind the upload
   screen's gallery/dropdown to:
   `OneDrive.ListFiles("Probationary Training Record")`
3. That list already contains `training-record.json` (and any monthly
   PDFs saved there). The user taps the file that is already listed
   instead of browsing for it. Exact expressions are in
   `formulas.txt`.

If a user's list is empty, they have not saved a record on the
website yet — the app should tell them to open the training record,
sign in, add one entry, and come back.

## Before this goes live

Two things still need Anthony's call — the branch is built so either
answer works, but do not merge until these are confirmed:

1. **Which Power Apps app?** Name of the already-built app, and is it
   a canvas app? (If it is a model-driven app, the hero goes in as a
   web resource / dashboard instead, and the formulas change.)
2. **What is the upload for?** What happens to the file after the
   user picks it in Power Apps — where does it go (training officer,
   SharePoint, Dataverse, a flow)? The formulas use the app's existing
   upload action on purpose; no new destination was invented here.
3. **Visible copy — yes or no?** The patch adds a second copy of the
   record to a visible OneDrive folder. That is what makes the file
   "already there" in Power Apps, but it is a new copy of people's
   records, so it needs a yes before it is turned on.

## Branch

```
git checkout power-apps-hero-page
```

Review the three files above, answer the three questions, then merge
to `main` only after the visible-copy patch is approved.
