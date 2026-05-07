# Earned — Web Companion

The static web surface for **Earned**, an iOS life-gamification app that lets people set goals, build habits, complete tasks for points, and spend those points on self-chosen rewards. Family members can be linked together to share progress.

This repository hosts everything the App Store needs to publish the app and everything the iOS client needs to handle invitation links — nothing more, nothing less.

[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)
[![App ID](https://img.shields.io/badge/App%20ID-N96HRUQNQW.com.kamronbekbatirov.Earned-1f6feb?style=flat-square&logo=apple)](https://apps.apple.com/app/earned)

## Why is this a separate web project?

The Earned app itself is native Swift, but iOS requires three publicly reachable resources for the experience to work:

1. **Apple App Site Association (AASA)** — declares which URL paths the app is allowed to handle as Universal Links.
2. **Universal-Link landing pages** — what visitors see if they click an invitation link without having the app installed yet.
3. **Privacy Policy & Terms of Service** — required by Apple to publish the app on the App Store.

All three live here as plain HTML, served from `https://my-bots.uz/earned/...` behind Caddy.

## What's in the repository

| Path | Purpose |
| --- | --- |
| `.well-known/apple-app-site-association` | The AASA manifest. Declares the app ID `N96HRUQNQW.com.kamronbekbatirov.Earned` and the two URL patterns it handles: `/earned/invite/*` and `/earned/join/*`. |
| `invite/index.html` | Landing for friend-invitation links (`/earned/invite/<code>`). Tries to launch the app via the `earned://` custom URL scheme, falls back to the App Store. |
| `join/index.html` | Landing for family-join links (`/earned/join/<code>`). Shows the join code on screen so the user can read it back if the deep-link bounce fails. |
| `privacy/index.html` | Standalone Privacy Policy page. The Markdown source is in `PRIVACY_POLICY.md`. |
| `terms/index.html` | Standalone Terms of Service page. Markdown source in `TERMS_OF_SERVICE.md`. |
| `PRIVACY_POLICY.md`, `TERMS_OF_SERVICE.md` | Source-of-truth Markdown for the legal pages. Edit here, regenerate the HTML, ship. |
| `Caddyfile.new` | Reference Caddy v2 configuration (originally written for a Raspberry Pi 5 host) showing how to serve the AASA file with the right `Content-Type`, set strict security headers, and block sensitive paths. |

## How Universal Links work here

```
User taps  https://my-bots.uz/earned/invite/ABC123
                  │
                  ▼
   ┌─────────────────────────────┐    iOS sees a valid AASA entry
   │  iOS reads cached AASA      │──► /earned/invite/* is on the
   │  (Apple CDN refreshed       │    allow-list → launches the app
   │   every 24 hours)           │    with a NSUserActivity
   └─────────────────────────────┘
                  │
                  ▼ (no app installed)
   ┌─────────────────────────────┐
   │  Browser opens invite/      │  ─►  earned://invite/<code>  (custom scheme)
   │  index.html, runs the JS    │      ↳ if the scheme handler is registered,
   │                             │        the app opens; otherwise the
   └─────────────────────────────┘        "Download Earned" CTA stays visible.
```

The 100 ms `setTimeout` before the deep-link redirect is intentional: it gives the page time to render so users on a fresh install still see the App Store call-to-action when iOS rejects the custom-scheme URL.

## Apple App Site Association — the gotcha

Apple's CDN is strict about three things:

1. The file must be served from `https://<host>/.well-known/apple-app-site-association` (no `.json` extension).
2. The response must have `Content-Type: application/json` exactly.
3. The certificate chain must be valid — Apple does not follow self-signed roots.

The provided Caddy configuration handles all three:

```caddy
handle /.well-known/apple-app-site-association {
    root * /var/www/earned
    header Content-Type application/json
    header Cache-Control "max-age=86400"
    file_server
}
```

If Universal Links are not opening the app, the first thing to check is whether iOS actually fetched the AASA — `curl -i https://<host>/.well-known/apple-app-site-association` should return `200 OK` with the JSON content type.

## Running locally

No build step, no dependencies. Any static file server will do:

```bash
python3 -m http.server 8080
# or
npx serve .
```

For the AASA file to actually exercise Universal Links you need a real domain with HTTPS and the iOS app installed on a physical device — the simulator does not support Universal Links.

## Updating the legal pages

The HTML in `privacy/index.html` and `terms/index.html` is regenerated from the Markdown sources whenever the policy changes. The current pages are dated **4 February 2026**. To update:

1. Edit `PRIVACY_POLICY.md` or `TERMS_OF_SERVICE.md`.
2. Bump the *Last Updated* date inside the document.
3. Re-render to HTML and copy into the corresponding `index.html` (the wrapper styles in the existing files can be reused as-is).
4. Commit, push, deploy.

## Deployment

Anything that can serve static files works: Caddy, Nginx, S3 + CloudFront, Cloudflare Pages, GitHub Pages — as long as:

- The AASA file is delivered with `Content-Type: application/json`.
- HTTPS is enforced (Universal Links require TLS).

A working production configuration is included in `Caddyfile.new`.

## License

Released under the [MIT License](LICENSE).
