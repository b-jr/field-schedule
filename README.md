# Field Schedule — Who's on the Field

Shared calendar for tracking which instructors are giving lessons on which day. Single static HTML page hosted on GitHub Pages, with Google Firebase Realtime Database as the shared backend. Any instructor with the link sees the same schedule; edits sync live to every open page, no accounts and no app install.

## How it works

- `index.html` draws the calendar and bottom-sheet editor. Pure HTML/CSS/JS, no build step.
- The Firebase JS SDK is loaded from Google's CDN at runtime.
- All data lives under one branch of the Realtime Database: `field-schedule/entries/<YYYY-MM-DD>` (that day's lineup) and `field-schedule/people` (the quick-add name list with colors).
- Writes are per-day transactions, so simultaneous edits by two people merge instead of overwriting each other.
- If Firebase is unreachable or misconfigured, the calendar still renders and a banner explains what's wrong; edits are refused with a toast instead of silently lost.

## Setup (already done, kept for rebuilds)

1. Firebase console → create project → add a Web app → copy the config values into `FIREBASE_CONFIG` in `index.html`. Only the values — the console's `import ... from "firebase/app"` snippet is for bundler projects and will break a static page.
2. Build → Realtime Database → Create Database.
3. Rules tab → paste the permanent rules from the setup comment inside `index.html` (opens only the `field-schedule` branch, everything else locked, day keys validated). Do not leave test-mode rules in place — they expire in ~30 days and the page will show a "permission denied" banner.
4. Push to GitHub, enable Pages (deploy from branch, `main`, root).

## Editing / maintenance

- Config lives in one block in `index.html` (`FIREBASE_CONFIG`).
- Colors and fonts: CSS variables at the top of the `<style>` block.
- The Firebase SDK version is pinned in the two CDN import URLs (`firebasejs/10.13.0/...`). Bump both together if you ever upgrade.

## Ground rules for content

Anyone with the URL can view and edit (by design, for now — see backlog). Therefore:

- Instructor first names, times, and short notes only.
- **Never** client names, phone numbers, addresses, payment info, or anything about minors in the notes.

---

## Security backlog

Roughly in priority order. Top two are quick and worth doing now-ish; the rest are "when the crew grows or the itch strikes."

- [ ] **Restrict the API key to this site.** Google Cloud console → APIs & Services → Credentials → the project's Browser key → Application restrictions → Websites → allow `b-jr.github.io/*` (plus `localhost` for testing). Stops the key from working on anyone else's site. ~5 min.
- [ ] **Verify the permanent database rules are live.** Realtime Database → Rules should match the block in `index.html` — no `.read: true` at the root, no expiry timestamp left over from test mode. Re-check after any console poking. ~2 min.
- [ ] **Set a Firebase budget/usage alert.** Free (Spark) plan can't run up a bill, but set an alert on Realtime Database usage anyway so abuse or a runaway loop shows up in email instead of as a dead calendar at quota. ~10 min.
- [ ] **Periodic data export.** Realtime Database → ⋮ → Export JSON, drop the file somewhere safe. Cheap insurance against a bad edit or vandalism, since anyone with the link can delete entries. Calendar reminder, monthly. ~2 min each.
- [ ] **Add Firebase Authentication.** The real upgrade: anonymous or email-link sign-in for instructors, then change rules to `".write": "auth != null"` (read can stay public or not). Kills the "anyone with the URL can edit" exposure. ~1–2 hrs including testing on everyone's phones.
- [ ] **App Check (after Auth).** Attests requests come from your actual site, backstopping the referrer restriction, which can be spoofed by scripts. Overkill today; cheap once Auth exists. ~30 min.
- [ ] **Tighten rule validation.** Current rules validate day-key format only. Could also cap entry counts and string lengths (`newData.val().length < 60`) so a hostile script can't stuff megabytes into a note field. ~30 min.
- [ ] **Audit the SDK pin occasionally.** Check Firebase JS SDK release notes a couple of times a year; bump the pinned `10.13.0` URLs if there's a security fix. ~10 min.

### Non-issues (so future-you doesn't re-litigate them)

- The `apiKey` in `index.html` is a project identifier, safe to publish; access control is the rules' job. Restriction above is hardening, not secrecy.
- No `.gitignore`/license needed: no build artifacts, private business page, all rights reserved by default.
