# All Is Well Healing Oasis — Link Page

Linktree-style landing page opened by a QR code on printed materials. Static HTML, no build step, no dependencies, no backend. Hosted on GitHub Pages.

## Files

```
index.html      the link page (logo, welcome line, six buttons)
qr/index.html   redirect stub — printed QR codes point here, it forwards to index.html
logo.png        (optional) square logo, referenced by index.html; page hides the slot if missing
```

## Deploy (GitHub Pages)

1. Push these files to the root of a public repo on the `main` branch (e.g. `b-jr/healing-oasis-links`). Keep the `qr/` folder.
2. Repo → Settings → Pages → Source: *Deploy from a branch* → `main` / `/ (root)` → Save.
3. Live within ~1 minute at `https://<user>.github.io/<repo>/`.

Nothing else to configure. No Actions workflow, no `.nojekyll` needed (no underscore-prefixed paths).

## URLs

| Purpose | URL |
|---|---|
| Link page | `https://<user>.github.io/<repo>/` |
| QR target (print this one) | `https://<user>.github.io/<repo>/qr/` |

**Only ever encode the `/qr/` URL into QR codes.** It contains a `<meta refresh>` + `location.replace` to `../`. If the link page is ever moved or rebuilt, change that one target and every printed code keeps working. Never delete or rename `qr/`.

## Editing content

All editable content is in `index.html`:

- **Links** — the `<ul class="links">` block, marked `EDIT THESE SIX LINKS`. Placeholders to replace: `YOUR_HANDLE` (Instagram), `YOUR_PAGE` (Facebook), `YOUR_REVIEW_LINK` (Google review short link), `+15555555555` (SMS number), the Maps query, and the Book a Session href (point at the direct Squarespace booking URL if one exists).
- **Copy** — `<h1>` and `<p class="tag">`.
- **Colors** — CSS custom properties in `:root` at the top of the `<style>` block.
- **Fonts** — loaded from Google Fonts (Cormorant Garamond, Nunito Sans). Falls back to Georgia / system sans if blocked.

Commit to `main`; Pages redeploys automatically.

## Custom domain (optional, later)

To serve at e.g. `links.alliswellhealingoasis.com`:

1. At the DNS provider for `alliswellhealingoasis.com`, add a `CNAME` record: `links` → `<user>.github.io`.
2. Repo → Settings → Pages → Custom domain → enter the subdomain → Save. Enable *Enforce HTTPS* once the cert issues (a few minutes).
3. This creates a `CNAME` file in the repo root; keep it.

Requires DNS access for the domain, which currently sits with the Squarespace account owner. Not required for the page or QR code to work — the `github.io` URL is fine to print.

## Testing checklist

- [ ] Open `/qr/` on a phone → lands on the link page instantly
- [ ] Every button opens the right destination; SMS button opens Messages
- [ ] Scan the actual printed/proof QR code before sending to print
