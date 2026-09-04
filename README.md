# Field Schedule — "Who's on the field"

Shared scheduling board for lessons and practice time across a facility with 2 fields, 4 batting cages, and the main field's bullpen. Single static HTML page hosted on GitHub Pages, synced live through Firebase Realtime Database. Used by the manager to see availability and confirm requests; conflicts are detected automatically, coach bookings have priority.

No build step, no framework, no server code. One file.

## Files

```
index.html    the entire app (UI, styling, Firebase wiring)
```

## How it works

- Tap a day → bottom sheet shows that day's bookings and an add form.
- Every entry requires a **name, time, and location**; a note is optional. The exact missing field is highlighted if skipped.
- Tapping a saved name in "Tap to add" fills the form and jumps to the Time box (it does not instant-add).
- All changes sync live to every open copy of the page — no refresh.

### Time input
Accepts start times or ranges and normalizes them: `3` → 3:00 PM, `15:00` → 3:00 PM, `3-4:40pm` → 3:00–4:40 PM, `11-1pm` → 11:00 AM–1:00 PM, `3 to 5` → 3:00–5:00 PM. Bare hours are guessed by lesson hours (8–11 → AM, otherwise PM); an explicit am/pm always wins. Unparseable text is kept as typed but is invisible to conflict checking.

### Locations
Fixed dropdown defined in `CONFIG.locations` in `index.html`: Main Field, Main Bullpen, Field 2, Cage 1–4, plus "Other…" for free text. Aliases map old free-text entries ("c2", "bull pen", "f1") onto the real spots. **Main Bullpen is nested inside Main Field** (`within:` key): booking the whole field conflicts with anything in its bullpen. Edit labels/aliases in that block to match what people actually call the spots.

### Conflict detection
Two entries clash when they occupy the same spot (or nested spots) and their time intervals overlap. Ranges use their real span; a lone start time is assumed to occupy `LESSON_MINUTES` (60, set near the top of the conflict section). Conflicts **warn, never block** — the person booking confirms through the dialog. Conflicting entries show a red ⚠ OVERLAP tag; days with any conflict show "⚠ conflict" on the calendar grid.

### Approval workflow
- New entries land as **PENDING** (amber tag, faded chip). Anyone viewing can tap ✓ to confirm.
- The header shows a running "N pending" badge counting all days.
- **Coach priority** checkbox: coach entries are auto-confirmed, and adding one over a regular lesson bumps that lesson back to pending (announced in the confirm dialog first). Coach never auto-bumps coach — that clash only warns.
- Entries created before the workflow existed have no status and count as confirmed.

## Deploy

1. Push `index.html` to the `main` branch root of a public repo.
2. Repo → Settings → Pages → Deploy from branch → `main` / root.
3. Live at `https://<user>.github.io/<repo>/` in about a minute. Commits redeploy automatically.

For a custom domain (Cloudflare DNS): four A records on `@` → `185.199.108.153 / .109. / .110. / .111.`, CNAME `www` → `<user>.github.io`, proxy set to DNS-only (gray cloud), then Pages → Custom domain → Enforce HTTPS.

## Firebase setup

Config lives in `CONFIG.firebase` in `index.html` (values from Firebase console → Project settings → your web app). The web API key is public by design — access control is entirely in the database rules. Restrict the key to your Pages domain in Google Cloud Console → Credentials → website restrictions.

Realtime Database rules (permanent — replaces expiring test mode):

```json
{
  "rules": {
    ".read": false,
    ".write": false,
    "field-schedule": {
      ".read": true,
      ".write": true,
      "entries": {
        "$day": {
          ".validate": "$day.matches(/^[0-9]{4}-[0-9]{2}-[0-9]{2}$/)"
        }
      }
    }
  }
}
```

Data layout: `field-schedule/entries/<YYYY-MM-DD>` (array of `{name, time, note, loc, priority, status}`) and `field-schedule/people` (quick-add list with colors). All writes are per-day transactions, so simultaneous edits merge instead of overwriting each other.

## Pending-request notifications (iOS Shortcuts)

The database is readable over REST, so a Shortcut can poll it:

1. Get Contents of URL → `https://<project>-default-rtdb.firebaseio.com/field-schedule/entries.json`
2. Match Text → `pending` → Count Items
3. If Count > 0 → Show Notification
4. Automations → Time of Day → daily, Run Immediately. One automation per check time.

Share the shortcut via iCloud link to whoever covers approvals.

## Security model (read before widening access)

Anyone holding the page URL can read **and edit** the whole schedule — the rules are open by choice, and "manager-only" is enforced by who receives the link, not by the app. Consequences:

- Share the link only with the manager / trusted staff.
- Never put client phone numbers or personal info in notes.
- If the link leaks or the user circle grows, the upgrade path is Firebase Auth (`".write": "auth != null"`) — not more obscurity.

## Known limits

- Warnings are advisory; nothing is hard-blocked (intentional — the manager overrides consciously).
- Entries have no IDs: two entries identical in name/time/location/note are confirmed/deleted together.
- Entries stored before time+location were required sit outside conflict checking until re-entered.
- Notifications are polling, not push; instant push would require Cloud Functions (paid tier).
- Free Firebase tier (Spark) covers this usage level with enormous headroom.

## Editing the app

Everything configurable sits at the top of the `<script type="module">` block in `index.html`: chip colors, locations and aliases, validation lengths, `LESSON_MINUTES`, Firebase config. Colors/theme are CSS variables in `:root` (currently CSUF-inspired blue `#00274C` / orange `#FF7900`). Commit to `main` to ship.
