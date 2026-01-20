# Link-shortner

A minimal static link shortener / redirect helper with a retro "uplink terminal" UI. This is a client-side, single-file HTML app that runs in the browser and is meant for personal, small-scale use.

Important: this project is NOT offline-first. It requires network access for:
- Redirecting to external target URLs (the entire purpose of the app).
- Reporting hits to a configured Discord webhook (if configured).
- Optional IP lookup (uses ipify) when sending webhook payloads.

Do not use this for production or to protect sensitive resources — see Security below.

## Features
- Shortcode-based links (e.g. `?id=ShortCode1`)
- Timed redirect (5s countdown) or direct (instant) redirect
- Admin panel (client-side password) to copy query-string extensions with custom ref and mode
- Locked links option (client-side password check using client-side hash)
- Optional webhook "analytics" that posts hits to a Discord webhook (reconstructs webhook from two base64 parts)
- Small single-file HTML app — no backend required, but network access is needed for redirects/webhook/ip lookup

## Quick start (run locally)
1. Clone or download this repo.
2. Open `index.html` in your browser or serve the folder with any static server.
3. Use the query string in the address bar to open a shortcode, for example:
   - Timed (default): `index.html?id=BUa8b2&ref=MyRef&mode=timed`
   - Direct: `index.html?id=BUa8b2&ref=MyRef&mode=direct`

## URL parameters
- id — The shortcode key from the `G_CONFIG.links` object.
- ref — Optional referral/string (used for your tracking).
- mode — `timed` (5s countdown) or `direct` (immediate redirect).

Example:
`index.html?id=Kynto&ref=TikTok&mode=direct`

## Admin panel
- Open `index.html` without query parameters to see the login screen.
- Enter the operator password. The app compares a client-side SHA-256-style hash against `G_CONFIG.settings.masterHash`.
- After login you can copy query strings pre-filled with your chosen `ref` and `mode`.

To change the admin password, generate a new master hash (see "Generating hashes" below) and replace `G_CONFIG.settings.masterHash` in `index.html`.

## Adding / editing links
Links live in `G_CONFIG.links` in `index.html`. Example entries:
```js
'MyShort': { url: 'https://example.com/path', locked: false }
'SecretLink': { url: 'https://example.com/secret', locked: true, hash: '<sha256-hash>' }
```
- `locked: true` makes a link require a password in the browser before redirecting.
- `hash` must be the SHA-256-style hash produced by the included generator routine (see Ls-Pw-Gen.html). The salt used must match `G_CONFIG.settings.salt`.

## Generating hashes & obfuscating webhooks
Open `Ls-Pw-Gen.html`:
- HASH_CONVERTER: Use this to produce the `masterHash` or a locked-link hash. The generator performs:
  base64(password + salt) → base64(result) → SHA-256(hex)
  Use the same `salt` value as in `G_CONFIG.settings.salt`.
- WEBHOOK_OBFUSCATOR: Splits and base64-encodes a webhook URL into two parts (partA and partB). These populate `G_CONFIG.settings.hookA` and `hookB` in `index.html`.

If you do not want webhook reporting, remove or clear `hookA`/`hookB` in `index.html` to prevent the app from attempting to POST hits.

## Webhook & IP behavior
- The webhook is reconstructed client-side from the two base64 parts and used to POST a payload when a link is visited.
- The app attempts to fetch the visitor IP from ipify (`https://api.ipify.org?format=json`) to include it in the payload. If that request fails, IP will be "N/A".
- If you care about privacy, do not configure a webhook or remove the ipify lookup.

## Security & privacy (READ CAREFULLY)
- All configuration (salt, masterHash, webhook parts, links and their hashes) is stored in plain JavaScript in `index.html`. Anyone with access to the file or deployed page can read them.
- Password checks for locked links are performed entirely client-side. Because the salt and hashes are public in the source, a determined user can bypass or discover passwords.
- Webhook reporting and IP lookup can leak visitor information. Do not enable these features on public deployments unless you understand the implications.
- This project is educational/personal in nature. For any real security needs, implement server-side validation, password hashing, and server-side webhook handling.

## Configuration pointers
- Salt: `G_CONFIG.settings.salt`
- Master hash: `G_CONFIG.settings.masterHash`
- Webhook parts: `G_CONFIG.settings.hookA` and `hookB` (base64 strings)
- Links: `G_CONFIG.links` object

Remember: if you change the salt, re-generate all hashes that depend on it.

## Deployment
This is a static app and can be hosted on any static hosting provider (GitHub Pages, Netlify, Vercel, etc.). However:
- Redirect targets are external and require network access.
- If webhook reporting is configured, the site must be able to reach Discord's webhook endpoints.

## Example link entries (from this repo)
```js
'BUa8b2': { url: 'https://imgur.com/a/got-you-45Qpn5g', locked: false },
'Hsi822b': { url: 'https://imgur.com/a/7SSpM0k', locked: false },
'Kynto': { url: 'https://discord.gg/K58KvmwftD', locked: false },
'Voltsim': { url: 'https://discord.gg/AC8MhtVfF6', locked: false },
'PwTest': { url: 'https://dummyimage.com/600x400/000/fff.png&text=Pw+Test', locked: true, hash: '74a7b26e07efe4547ffa4bc907df05d141cc6e4e5671c51afea958eda3b13ad9' }
```

## Contributing
This is a small personal project. Suggestions and pull requests are welcome — especially for:
- Improving security by moving sensitive logic to server-side endpoints.
- Making webhook reporting optional/configurable without exposing secrets.
- Documentation and usability improvements.

If you submit a PR that changes hashing or salt behavior, include clear migration steps.

## License
This project is released under the GNU GPL v3. See the included LICENSE file for full terms.

## Contact
- GitHub: [ElectroBoy10](https://github.com/ElectroBoy10)

If you have security or privacy concerns, avoid posting secrets in public issues; use a private channel if possible.

Enjoy — and please avoid exposing this to public users without addressing the security considerations above.
