# Rebuild the Probationary Firefighter Training Record in Power Apps

Standalone rebuild. No connection to any website, no link out to
anyone else's site, no shared account, no client ID, no MSAL.
The person rebuilding it builds their own Power Apps app, signs in
with their own Microsoft account, and their firefighters' records
save to each firefighter's own OneDrive.

## What it is

A digital version of the paper "Probationary Firefighter Training
Record" form, due with the Monthly Grading Report.

- Header: Probationary Firefighter / Company / Shift / Captain
- Rows: Date / Training / Comments, one row per training
- Monthly report view that reproduces the paper sheet
- Send that month's report to the training officer, from the
  firefighter's own mailbox

## Data model — one file per firefighter

Each firefighter's record is one JSON file in their own OneDrive,
in a folder they can see:

```
Probationary Training Record/training-record.json
```

File format (same shape as the paper form — keep it exactly):

```json
{
  "version": 1,
  "profile": {
    "name": "",
    "company": "",
    "shift": "",
    "captain": "",
    "officerEmail": ""
  },
  "entries": [
    { "id": "e1", "date": "2026-08-28", "training": "", "comments": "" }
  ],
  "updatedAt": "2026-11-05T00:00:00.000Z"
}
```

- `entries[].id` — unique per row, e.g. `Text(Now(), "yyyymmddhhmmss") & Rand()`
- `entries[].date` — `YYYY-MM-DD`
- `updatedAt` — set to `Text(Now(), UTC)` on every save

## Screens — build these, in this order

### 1. HeroScreen (main screen — first screen in the app)

This is the front door. No web link, no launch — every button
navigates to another screen inside this same Power Apps app.

- Label, title: `Probationary Firefighter Training Record`
- Label, sub: `Due with Monthly Grading Report`
- Label, privacy note: `Your record saves to your own OneDrive,
  under your own Microsoft account. No one else can see it.`
- Button `My training record` → `Navigate(RecordScreen)`
- Button `This month's report` → `Navigate(ReportScreen)`
- Button `Upload my saved record` → `Navigate(UploadScreen)`

Make HeroScreen the first screen in the screen list. Power Apps
opens the first screen in the list.

### 2. RecordScreen — firefighter information + training entries

**Profile section** — five text inputs, one per field:
`txtName`, `txtCompany`, `txtShift`, `txtCaptain`, `txtOfficerEmail`
(placeholders: `Engine 13`, `C Shift`, `officer@phoenix.gov`).

**Entries section** — a gallery, one row per training:
- Date picker (`DateValue`, format `YYYY-MM-DD`)
- Text input for Training
- Text input for Comments
- Delete button per row

Controls above the gallery:
- Month picker (date picker, or a dropdown of `YYYY-MM` values)
- Button `Show all` — toggles the month filter off
- Button `+ Add entry` — adds a blank row to the local collection

**Save** — a button `Save to my OneDrive`, plus auto-save on
change if you want it. Save writes the whole record as one JSON
file (see "Save / load" below). Show `Saved to your OneDrive`
with the time after a successful save.

### 3. ReportScreen — monthly report (the paper sheet)

Reproduces the paper form on screen:
- Header with the five profile fields
- Table of that month's entries only, sorted by date
- Footer line: `Due with Monthly Grading Report`

Buttons:
- `Save as PDF` — Power Apps `PDF()` function on this screen,
  then save the PDF to the same OneDrive folder (see formulas).
- `Send to training officer` — sends that month's report by email
  from the firefighter's own mailbox to `txtOfficerEmail`
  (see formulas — Office365Outlook connector, HTML body).

### 4. UploadScreen — your saved file is already there

This is the screen that makes the upload work without hunting:

1. Add the **OneDrive** connector as a data source. The user
   signs in with their own Microsoft account — the same account
   their record was saved under.
2. Gallery / dropdown, Items:
   `OneDrive.ListFiles("Probationary Training Record")`
3. That gallery already lists `training-record.json` (and any
   monthly PDFs saved there), because that is the folder the
   RecordScreen saves to. The user taps the file that is already
   listed — they do not browse their device for it.
4. Button `Use this file` — passes the selected file to whatever
   this app already does with uploads (your existing upload action).
   Do not invent a new destination here.

If the gallery is empty, the user has not saved a record yet.
Show: `No saved record found yet. Go to My training record,
add one entry and save it — it will be listed here.`

## Save / load

Local collections (in the app, per user):
- `colProfile` — one row, the five profile fields
- `colEntries` — one row per training entry

**On app start (`App.OnStart`) — load the user's own record:**
Load `training-record.json` from their OneDrive folder into the
collections. If the file does not exist yet, start with a blank
record (empty profile, no entries) and create the file on first save.

**On save — write the whole record back to the same place:**
One file, one folder, same name every time, so the upload screen
always finds it:
- Folder: `Probationary Training Record`
- File: `training-record.json`
- Content: the record JSON above

Exact formulas are in `formulas.txt`. Do not save to a hidden
app folder — Power Apps cannot reliably list hidden folders, and
the whole point is that the file is already there to choose from
on the upload screen.

## Sign-in

There is none to build. Power Apps signs the user in with their
own Microsoft account when they open the app and when the OneDrive
and Outlook connectors run. Do not add MSAL, do not add a client
ID, do not add a redirect URL. Each firefighter sees only their
own OneDrive, because the connectors run as that firefighter.

## What is deliberately NOT in this rebuild

- No link or launch to any website
- No GitHub Pages, no hosted page
- No shared Azure app registration or client ID
- No shared backend, no shared database, no shared folder
- No one else's records — every record lives in the firefighter's
  own OneDrive, under their own account

If you want the same file format as the original paper form, use
`data-model.json` in this folder as the example file.
