# Earned — Web Companion

Static web companion for the **Earned** iOS application. Hosts the public marketing surface that complements the native experience: Apple Universal Links, family invitation handlers, the Privacy Policy, and the Terms of Service required by the App Store.

This repository is intentionally framework-free. Every page is plain HTML and CSS so that the site loads instantly, works without JavaScript, and remains trivial to host behind any reverse proxy or CDN.

## What lives here

| Path | Purpose |
| --- | --- |
| `.well-known/apple-app-site-association` | Apple App Site Association (AASA) file declaring the app ID and the URL patterns that the iOS application is allowed to handle. Served with `Content-Type: application/json`. |
| `invite/` | Landing page for friend invitation links (`/earned/invite/<code>`). Attempts to open the app via the `earned://` custom scheme; falls back to the App Store. |
| `join/` | Landing page for family join links (`/earned/join/<code>`). Same routing strategy as invitations. |
| `privacy/` | App Store Privacy Policy. |
| `terms/` | App Store Terms of Service. |
| `Caddyfile.new` | Reference Caddy v2 configuration that wires the routes together with the correct headers and `Content-Type` for the AASA file. |

## Apple Universal Links

The AASA file is served from `https://<host>/.well-known/apple-app-site-association` with the JSON content type and a 24-hour cache. The companion iOS app declares the following associated domain in its entitlements:

```
applinks:<host>
```

Apple's CDN fetches the AASA, validates the signature of the app bundle, and starts honouring `https://<host>/earned/invite/*` and `https://<host>/earned/join/*` as direct app launches. When the user has not installed the app, the inline JavaScript on each landing page degrades gracefully and bounces them to the App Store.

## Local development

No build step is required. To preview locally, run any static file server from the repository root:

```bash
python3 -m http.server 8080
# or
npx serve .
```

Then open <http://localhost:8080>.

## Deployment

Any static host will do — Caddy, Nginx, S3 + CloudFront, Cloudflare Pages, GitHub Pages, etc. Just make sure of two things:

1. The AASA file is delivered with `Content-Type: application/json` (Apple is strict about this).
2. HTTPS is enforced — Universal Links will not work over plaintext HTTP.

A working Caddy configuration is provided in `Caddyfile.new`.

## License

MIT — see [LICENSE](LICENSE).
