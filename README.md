# Weekly Jam RSVP page

A static page (`index.html`) plus a Google Sheet + Apps Script backend (`apps-script/Code.gs`).

- Anyone with the link can RSVP with a name plus an email, a phone number, or both (no accounts).
- Each week is its own event with its own RSVPs and notes. The next 5 dates are shown.
- Emails and phone numbers live only in your Google Sheet. The public API never returns them.
- A person is recognized by email or phone, so someone who RSVPs by phone and later adds an email stays one person.
- When you add a night's recordings link, everyone who RSVP'd Going or Maybe for that night is emailed the link automatically.
- When you cancel a night in the `Schedule` tab, everyone who RSVP'd Going or Maybe for it is emailed a cancellation notice (with your reason and the next jam).
- You (and only you) get an email for each brand-new RSVP. Set `OWNER_EMAIL` in `CONFIG`; blank means the Google account that owns the script.
- The page footer and every email link to your top-level songs folder (`SONGS_URL` in `CONFIG`).
- An hourly trigger emails everyone on the list about 24 hours before each jam. Reminders are email-only; phone-only people are tracked but not contacted.

## Setup (about 10 minutes)

1. **Create the sheet.** New Google Sheet, then Extensions > Apps Script.
2. **Paste the backend.** Replace the editor contents with `apps-script/Code.gs`. Edit `CONFIG` at the top (name, day, time, location, time zone, `SITE_URL`, `REPLY_TO`). `SITE_URL` is already set to your GitHub Pages address..
3. **Run `setup`.** Pick `setup` in the function dropdown and click Run. Approve the permissions (sheet + send mail). This creates the `Members`, `RSVPs` and `Notes` tabs and the hourly reminder trigger.
4. **Deploy.** Deploy > New deployment > Web app. Execute as: **Me**. Who has access: **Anyone**. Copy the `/exec` URL.
5. **Wire up the page.** Paste that URL into `APPS_SCRIPT_URL` near the bottom of `index.html`.
6. **Host `index.html`.** Any static host works (GitHub Pages, Netlify, Cloudflare Pages). Put the final URL in `SITE_URL` in `CONFIG` so reminder emails link back. If you edit `Code.gs` later, use Deploy > Manage deployments > Edit > New version.
7. **Share the link** with your list.

Open `index.html` before step 5 to preview in demo mode (data stays in your browser).

## Using it

- **Your existing list:** paste emails into the `Members` tab (just the `email` column, plus `name` if you like). They'll get reminders; blank `subscribed` counts as yes. Phone numbers go in the `phone` column (10-digit US numbers are assumed +1; use a leading `+` for other countries).
- **Upgrading an older sheet:** paste the new `Code.gs`, run `setup` once (it adds the new `phone` and `member` columns without touching your data), then Deploy > Manage deployments > Edit > New version.
- **Cancel or add dates (owner only):** open the `Schedule` tab in your Google Sheet. Only people with access to the sheet can edit it, and changes show on the page immediately with no redeploy.
  - Cancel a regular jam: put the date in column A and `cancel` in column B. Optional reason in column E (shown on the page). The date gets a strikethrough tab, RSVPs are closed and no reminder is sent.
  - Add a one-off jam: date in A, `add` in B. Optional time (`19:30` or `7:30 PM`) in C, location in D, and note in E. Blank time and location use your regular ones. One-offs show up once they fall inside the window of dates the page already lists (the next 5 regular jams).
  - Dates can be typed as `2026-12-24` or `12/24/2026`. Delete a row to undo it.
- **Change the slot:** edit `DAY_OF_WEEK` / `START_TIME`, redeploy.
- **Reminder content:** date/time/location, the viewer's own RSVP status, who's going, the latest notes, an RSVP link and a one-click unsubscribe. People who RSVP'd "can't make it" are skipped.
- **Unsubscribe:** the link in each email, or the person unchecks the reminder box when RSVPing.

## Notes and limits

- The web app is public by design, so anyone with the link can post RSVPs/notes. A honeypot field, input length caps and formula-injection guarding are included, but there are no logins. Delete junk rows in the sheet if needed.
- Gmail's script quota is about 100 recipients/day (1,500 on Workspace), which is plenty for a jam list.
- Reminder timing is checked hourly, so emails arrive within an hour of the 24-hour mark.

## Recordings: email the link to everyone who was there

Open the `Recordings` tab in your Google Sheet (it's created by `setup`). Add one row per night:

| date | url | note | notified_url / notified_at / sent_to |
|---|---|---|---|
| 2026-11-20 | `https://1drv.ms/f/c/...` (that night's OneDrive folder link) | optional message | filled in by the script |

- **What happens:** as soon as a row has a valid date and an `https://` link, everyone who RSVP'd **Going or Maybe** for that night gets an email with the link, a line saying they can listen without logging in to OneDrive, your optional note, the top-level songs link and an unsubscribe link. People who said Can't make it, have no email, or unsubscribed are skipped.
- **Updating a record:** change the `url` and everyone is emailed again with the new link. Editing only the note does not re-send. To re-send the same link, clear the `notified_url` cell on that row.
- **Paste the link in one go.** The email goes out the moment the link is saved, so paste it rather than typing it, and double-check it's the right folder.
- **The link must work without a login.** In OneDrive, set the share link to **Anyone with the link can view** (not "specific people"), and test it in a private browser window before adding it to the sheet.
- **Safety net:** if an email batch can't finish (for example, the daily email quota runs out), the hourly job finishes it and never double-sends to people already emailed.
- **Try it first:** in the Apps Script editor, pick `testRecordingEmail` and Run. It emails a sample notice to you.

## Files to upload to GitHub (same folder)

`index.html`, `swampass-header.webp`, `swampass-header-mobile.webp`, `favicon.ico`, `favicon-32.png`, `favicon-192.png`, `apple-touch-icon.png`. You can delete the old `swampass-header.jpg`.

After pasting a new `Code.gs`: run `setup` once (it creates the `Recordings` tab and the instant-send trigger), then Deploy > Manage deployments > Edit > New version > Deploy.

## Cancellation notices and RSVP alerts

- **Cancel a night:** in the `Schedule` tab, enter the date and `cancel` in the action column. Everyone who RSVP'd **Going or Maybe** for that date (with an email, not unsubscribed) is emailed right away: the night is cancelled, your reason, the next jam, the songs link and an unsubscribe link. The script fills in `notified_at` and `sent_to` so nobody is emailed twice, and the hourly job finishes any batch that didn't complete.
- **Fill in the reason first.** The email goes out as soon as the row has a date and `cancel`, and the note is included in it, so type the note before you choose `cancel` (or enter the date last). Double-check the date: a wrong date emails that night's RSVPs.
- **Only dates still ahead** are announced, and removing the row doesn't send anything. Cancelling via `CONFIG.CANCELLED` still works for the page but sends no email, so use the `Schedule` tab.
- **New RSVP alerts:** you get one email per new RSVP with the name, answer, instrument, note, email, phone, the running count for that night and a link to the sheet. Changing an existing RSVP doesn't email you, and no one else is ever copied.
- **Where the songs link is used:** `SONGS_URL` in `CONFIG` is shown in the page footer (served to the page through the config the script publishes) and added to every email to RSVPs (reminders, recording notices and cancellations).
