# Power Apps — Rebuild the Training Record (branch: `power-apps-hero-page`)

Rebuild the Probationary Firefighter Training Record **in Power
Apps, as a standalone app**. No connection to any website, no link
out to anyone else's site, no shared account, no client ID.

The person rebuilding it builds their own Power Apps app from
scratch, using the spec in this folder. They sign in with their own
Microsoft account, and each firefighter's record saves to that
firefighter's own OneDrive.

This branch does **not** change the live website on `main` — nothing
here goes live until it is reviewed.

## What is in this branch

| File | What it is |
|---|---|
| `power-apps/rebuild-spec.md` | The full rebuild spec, screen by screen: Hero (main screen), Record, Monthly Report, Upload — plus the data model, save/load, and sign-in (none to build). Read this first. |
| `power-apps/formulas.txt` | Exact Power Apps expressions to paste in: collections, save to OneDrive as JSON, month filter, monthly report, PDF, send-to-officer email, and the upload file list. Native Power Apps only — no `Launch()`, no web URL. |
| `power-apps/data-model.json` | Example `training-record.json` — the exact file format the rebuild saves and loads. |

Deleted from the first draft of this branch, on purpose:
`hero-page.html` (a web page that linked out to the website) and
`visible-copy-patch.md` (a patch to the website) — both assumed a
connection to the website. This rebuild has no such connection.

## How to rebuild it

1. In Power Apps Studio, create a **new canvas app** (blank).
2. Add two connectors as data sources: **OneDrive** and
   **Office365Outlook**. Both run as the signed-in user — no
   shared account, no setup per firefighter.
3. Build the four screens in `rebuild-spec.md`, in order:
   `HeroScreen` → `RecordScreen` → `ReportScreen` →
   `UploadScreen`.
4. Make `HeroScreen` the first screen in the screen list — that
   makes it the main screen. Every button on it navigates to
   another screen inside the same app. Nothing launches a website.
5. Paste the formulas from `formulas.txt` into the matching
   controls.
6. Save and publish the app.

## How "already there to choose from" works

The Record screen saves the whole record as one file, every time,
to the same visible folder in the firefighter's own OneDrive:

```
Probationary Training Record/training-record.json
```

The Upload screen lists that exact folder:

```
OneDrive.ListFiles("Probationary Training Record")
```

So when a firefighter goes to upload, `training-record.json`
(and any monthly PDFs they saved there) is already listed. They
tap the file that is already there — they do not browse their
device for it, and they do not re-type anything.

If the list is empty, they have not saved a record yet. The app
tells them to go to My training record, add one entry, save it,
and come back — then it will be listed.

## What is deliberately NOT in this rebuild

- No link or launch to any website
- No GitHub Pages, no hosted page
- No shared Azure app registration or client ID, no MSAL
- No shared backend, database, or folder
- No one else's records — every record lives in the firefighter's
  own OneDrive, under their own account

## Branch

```
git checkout power-apps-hero-page
```

Review `rebuild-spec.md` first, then `formulas.txt`.
