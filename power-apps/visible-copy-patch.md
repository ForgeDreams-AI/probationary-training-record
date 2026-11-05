# Make the saved record visible to Power Apps

## The problem this solves

The training record website saves each firefighter's record to their
**OneDrive App Folder** — Microsoft Graph
`/me/drive/special/approot:/training-record.json:/content`.

That folder is private and correct for the website, but it is *not*
a normal folder a person (or a Power Apps file picker) can browse to
easily. If Power Apps is supposed to let someone pick their saved
record when they upload, the file has to also live somewhere visible.

## The fix (in this branch's plan, not yet in the live site)

Every time the website saves the record to the App Folder, also save
an identical copy to a normal, visible folder in the same OneDrive:

```
Probationary Training Record/training-record.json
```

That folder is created automatically the first time a copy is saved.
From then on, in Power Apps, that file is already there to choose from.

## Exact code to add — in `index.html`, inside `saveRecord()`

After the existing App Folder `PUT` succeeds, add a second `PUT`
to the visible folder. It uses the same token, same body, same
`Files.ReadWrite.AppFolder` scope does **not** cover this — the app
needs `Files.ReadWrite` (or `Files.ReadWrite.All`) for a normal
folder. Add that delegated permission in the Azure app registration
alongside the existing ones, or the second save will fail with 403
while the first one still works.

```js
// Visible copy, so Power Apps can list / pick it up on upload.
// Runs after the private App Folder save succeeds.
async function saveVisibleCopy() {
  var res = await graphFetch(
    "/me/drive/root:/Probationary Training Record/" +
      encodeURIComponent(FILE_NAME) + ":/content",
    {
      method: "PUT",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(state.record, null, 2)
    }
  );
  if (!res.ok) throw new Error("Visible copy failed (" + res.status + ")");
}
```

Call it from `saveRecord()`, right after the App Folder write:

```js
// inside saveRecord(), after the first res.ok check:
await saveVisibleCopy();
```

Best-effort version (recommended while testing): wrap the call in
`try { await saveVisibleCopy(); } catch (e) { /* log only — the
private record is already saved */ }` so a visible-copy failure
never blocks the real save.

## What changes for the firefighter

Nothing on the website. They keep signing in and typing entries
exactly as before. The only difference is a second copy appears in
their OneDrive, in a folder they can actually see:

OneDrive → **Probationary Training Record** → `training-record.json`

plus any monthly report PDF they save there from the website's
Print → Save as PDF step.

## What Power Apps reads

With the OneDrive connector added to the Power Apps app:

```
OneDrive.ListFiles("Probationary Training Record")
```

returns `training-record.json` (and any PDFs in that folder).
Bind that list to a gallery / dropdown on the upload screen, so the
saved file is already listed — the user taps it instead of browsing
for it. See `formulas.txt` for the exact expressions.
