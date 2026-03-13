# UPLINK TERMINAL

> A client-side link shortener with a retro terminal UI. No backend. No database. Three files.

Runs entirely in the browser — deploy anywhere static files are served: GitHub Pages, Netlify, Vercel, Cloudflare Pages, or just open locally.

---

## Files

| File | Purpose |
|---|---|
| `index.html` | The shortener — fetches `links.json` and handles all redirects |
| `links.json` | Your config — all links, settings, salts, and webhook data |
| `Ls-Pw-Gen.html` | Management dashboard — edit links, manage backups, generate hashes |

**Important:** `index.html` and `Ls-Pw-Gen.html` must be in the same directory as `links.json`. All three files are required.

---

## Quick Start

1. Clone or download this repo
2. Push all three files to GitHub and enable Pages, or drop them on Netlify/Vercel
3. Visit `index.html` with no parameters to reach the operator login
4. Use a URL like `index.html?id=Kynto&ref=Twitter&mode=timed` to trigger a redirect

> **Local testing:** opening `index.html` directly as a `file://` URL will fail to fetch `links.json` due to browser CORS restrictions. Use a local server instead — `npx serve .` or the VS Code Live Server extension both work.

---

## Features

- **Shortcode redirects** — clean `?id=ShortCode` URLs
- **Timed (5s countdown) or instant redirect modes**
- **Auto-bypass** — users can opt into instant redirects for all future visits
- **Locked links** — password-gate any link before it routes
- **One-time visit links** — blocks after first use per browser with optional operator override
- **Link expiry dates** — auto-deactivate after a set date
- **Custom redirect messages** — per-link text shown during the countdown
- **Discord webhook analytics** — hit reports posted to a Discord channel
- **Admin panel** — login, query string builder, QR code generator, claim reset
- **Admin lockout** — 5 wrong passwords locks the terminal in that browser with a fake trace animation
- **Multi-salt system** — add as many salts as you like; the app tries all of them on every hash check so you never need to track which salt belongs to which entry
- **Bypasses redirect tracking** — redirect fires via `window.location.href` in the browser, not a server-side HTTP 301/302, so most referral scrapers never see the hop

---

## URL Parameters

| Parameter | Values | Description |
|---|---|---|
| `id` | shortcode key | Which link to route to |
| `ref` | any string | Optional referral tag |
| `mode` | `timed` / `direct` | Countdown or instant redirect |

```
index.html?id=Kynto&ref=TikTok&mode=direct
```

---

## links.json Structure

```json
{
  "settings": {
    "salts": ["your_salt_here"],
    "masterHash": "sha256-hex-of-admin-password",
    "hookA": "base64-first-half-of-webhook",
    "hookB": "base64-second-half-of-webhook",
    "maxAdminAttempts": 5
  },
  "links": {
    "SHORTCODE": {
      "url": "https://destination.com",
      "locked": false,
      "hash": null,
      "expires": null,
      "message": null,
      "oneTime": false,
      "oneTimePass": null
    }
  }
}
```

### Link fields

| Field | Required | Description |
|---|---|---|
| `url` | Yes | Destination URL |
| `locked` | Yes | `true` to require a password before redirecting |
| `hash` | If locked | SHA-256 hash of the lock password |
| `expires` | No | ISO date string `"2025-12-31"` — link blocks after this day |
| `message` | No | Custom text shown during the countdown |
| `oneTime` | No | `true` to allow only one visit per browser |
| `oneTimePass` | No | Hash of an override password to reset a claim from the blocked screen |

### Old format — still supported

```json
"OldLink": { "url": "https://example.com", "locked": false }
```

All new fields are optional. Missing fields default to `null`/`false` safely.

---

## Management Dashboard (Ls-Pw-Gen.html)

Open `Ls-Pw-Gen.html` in any browser. It auto-loads `links.json` on open.

### Links tab
- View all registered links with status badges
- Add new links — hashing handled automatically
- Inline edit any link (URL, message, expiry, lock status, hashes)
- Reorder with up/down buttons
- Delete with confirmation

### Settings tab
- **Salt table** — add/remove salts. The app tries every salt on every hash check so entries never need to carry a "which salt" field. Add freely, remove carefully — removing a salt breaks any hashes made with it
- **Admin password hash** — paste a new hash here to change the admin password
- **Discord webhook** — edit hookA/hookB or use the obfuscator in Hash Tools
- **Max lockout attempts** — default 5

### Hash Tools tab
- **Hash Generator** — produces SHA-256 via `btoa(pass+salt) → btoa → SHA-256`. Shows which salt was used
- **Webhook Obfuscator** — splits a Discord webhook URL into hookA/hookB. One-click apply to Settings
- **Webhook Revealer** — decodes hookA+hookB back to the full URL for verification

### Export tab
- Builds the complete `links.json` ready to copy and paste back into your file
- Shows a timestamped change log of everything edited this session
- Save Backup button right there before you export

### Backups tab
- Save named backups to browser localStorage
- **Pin** backups — pinned backups never count toward the auto-delete limit and are never auto-deleted
- **Rename** and **delete** any backup
- **Auto-purge** — set a limit (10, 20, 50, or none). When exceeded, oldest unpinned backups are deleted automatically. Pinned backups are untouched
- **Load** any backup into the working session

### Workflow

```
Open Ls-Pw-Gen.html
  → Make changes (Links / Settings tabs)
  → Export tab → click Regenerate → copy the JSON
  → Paste into links.json in your repo
  → Commit and push
```

The tool can read `links.json` but cannot write back to it — that requires a server. The copy-paste step is the manual part of a static setup.

---

## Salt System

The multi-salt system means you can use different salts for different links without tracking which is which. The app tries every salt in `settings.salts` on every password check until one matches.

**Rules:**
- Add new salts at any time — safe, never breaks anything
- Do not remove a salt if any existing hash was generated with it — the hash will stop working
- The salt used when generating a hash via the dashboard is shown in the output so you know which one to keep

---

## Admin Panel (in index.html)

Open `index.html` with no URL parameters. Enter the operator password.

- Build and copy query strings with ref tags and mode selection
- Quick-fill the builder from any link card
- Generate QR codes for any shortcode
- Reset one-time claim flags
- Reset the auto-bypass preference

### Changing the admin password

1. `Ls-Pw-Gen.html` → Hash Tools → Hash Generator
2. Enter your salt and new password → copy the hash
3. `Ls-Pw-Gen.html` → Settings → Admin Password Hash → paste → Save
4. Export → copy JSON → paste into `links.json` → commit

---

## Admin Lockout

5 wrong passwords fills 5 red dots one by one, shaking the card each time. On the 5th failure the screen switches to a dramatic **ACCESS_TERMINATED** view with a staggered fake trace:

```
> BREACH_DETECTED: excessive auth failures
> LOGGING_INCIDENT: session fingerprinted
> UPLINK_PROTOCOL: terminal suspended
> OPERATOR_ALERT: notified ✓
```

The "operator notified" line is a complete lie. There is no server. The lockout is stored in `localStorage` as `uplink_admin_locked` and clears when site data is cleared.

---

## Discord Webhook

Hit reports include: shortcode, ref tag, mode, visitor IP (via ipify), destination URL, and one-time status.

**Setup:**
1. Create a webhook in your Discord server (Server Settings → Integrations → Webhooks)
2. `Ls-Pw-Gen.html` → Hash Tools → Webhook Obfuscator → paste URL → Split and Encode
3. Click "Apply to Settings" → go to Export → copy → paste into `links.json`

**Disable:** set `hookA` and `hookB` to empty strings `""`.

---

## One-Time Visit Links

Set `"oneTime": true`. After first visit the browser stores a timestamp in `localStorage` under `uplink_visited_SHORTCODE` and subsequent visits show the **LINK_CLAIMED** screen with a fake trace showing when they first visited.

**Optional override:** set `oneTimePass` to a hash. Entering the correct password on the claimed screen resets the flag and re-runs the router. Useful for "I sent this link to one person who needs to re-open it".

**Reset from admin panel:** `index.html` admin panel → Reset One-Time Claim section → enter shortcode → Clear.

---

## localStorage Keys

| Key | Purpose |
|---|---|
| `uplink_visited_SHORTCODE` | One-time visit flag, one per shortcode |
| `uplink_admin_attempts` | Wrong password counter (resets on success) |
| `uplink_admin_locked` | Lockout flag after max failures |
| `autoSkip` | User's instant-redirect preference |
| `uplink_backup_TIMESTAMP` | Saved backup (one per backup) |
| `uplink_backup_limit` | User's chosen auto-purge limit |

---

## Deployment

Static files — no build step, no server required.

| Host | How |
|---|---|
| **GitHub Pages** | Push repo, enable Pages in Settings → Pages |
| **Netlify** | Drag and drop the folder |
| **Vercel** | Connect the repo, deploy |
| **Cloudflare Pages** | Connect the repo |
| **Local** | `npx serve .` then open `http://localhost:3000` |

All three files must be served from the same origin and directory.

---

## Security Notes

- All config (salts, hashes, webhook parts) is readable in plain JSON by anyone who views `links.json`
- Password checks are entirely client-side — a determined person with the hash and salt table can brute-force
- The webhook split is obfuscation, not encryption
- IP logging uses a third-party service (ipify)
- The admin lockout and one-time visit flags are stored in `localStorage` and clear with site data
- **This project is a personal deterrent, not a security system.** It works brilliantly for keeping casual nosy people out. It will not stop someone who understands browser dev tools.

---

## Backwards Compatibility

All original-format links work without changes:

```json
"OldLink": { "url": "https://example.com", "locked": false }
```

New fields are optional and default safely.

---

## License

GNU GPL v3 — see `LICENSE`.

## Contact

GitHub: [ElectroBoy10](https://github.com/ElectroBoy10)

Don't post secrets (webhook URLs, passwords) in public issues.
(hmmmm just excuse my discord webhook in plain site i have no backend and its my only option for now)
(used claud ai to edit my original files and add new features)
