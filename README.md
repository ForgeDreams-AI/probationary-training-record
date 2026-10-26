# Probationary Firefighter Training Record

A single-page web app for Phoenix Fire Department probationary
firefighters. Each firefighter signs in with their own `@phoenix.gov`
Microsoft account and keeps a digital training record (date / training /
comments, plus name / company / shift / captain) saved to **their own
OneDrive app folder** as `training-record.json`. Nobody else can see it —
not even the person who runs this site. A monthly report view reproduces
the paper sheet, and the record can be emailed to the firefighter's
department training officer from their own mailbox.

Live site: https://forgedreams-ai.github.io/probationary-training-record/

## How sign-in works

- MSAL.js (browser) against `https://login.microsoftonline.com/common`
  (multi-tenant + personal accounts).
- The app itself enforces the `@phoenix.gov` domain check after sign-in.
- Microsoft Graph scopes (delegated): `User.Read`,
  `Files.ReadWrite.AppFolder`, `Mail.Send`.
- There is **no backend and no secret** in this repo. The only identifier
  is the Application (client) ID, which is public by design.

## Azure setup (one time, before sign-in works)

1. In the Azure portal, create an App registration.
2. Supported account types: **Accounts in any organizational directory
   (multitenant) and personal Microsoft accounts**.
3. Redirect URI, platform **Single-page application (SPA)**:
   `https://forgedreams-ai.github.io/probationary-training-record/`
   (trailing slash — this must match exactly what the app sends; the app
   uses the current page URL as its redirect URI).
4. API permissions (delegated): `User.Read`,
   `Files.ReadWrite.AppFolder`, `Mail.Send`.
5. Copy the Application (client) ID and paste it into `index.html` as
   `DEFAULT_CLIENT_ID` near the top of the script, then redeploy. That
   one value then works for every visitor. (A visitor can still override
   it for their own browser via the app's setup section, which saves to
   localStorage and takes precedence.)

Until a real client ID is baked in, sign-in stays off and the app runs in
preview mode with sample entries.
