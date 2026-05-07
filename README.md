# Earned — Web Companion

> **The supporting website for [Earned](https://apps.apple.com), an iOS app that turns daily self-improvement into a points-based game.**

[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)
[![Static](https://img.shields.io/badge/static-HTML%20%2B%20CSS-orange?style=flat-square)](.)
[![iOS](https://img.shields.io/badge/companion%20to-iOS%20app-000?style=flat-square&logo=apple&logoColor=white)](.)

---

## What is Earned?

Earned is a mobile app for people who want to build better habits but find traditional to-do lists boring. The idea is simple:

- Set goals you actually care about — fitness, learning, family time, anything.
- Complete tasks that move you toward those goals.
- Earn points for finishing tasks.
- Spend those points on rewards you've chosen for yourself: a movie night, an expensive coffee, a video-game session, whatever feels worth it to you.
- Pull family members in so you can share progress and celebrate together.

The app itself is native Swift on iOS. **This repository is not the app.** This is the small public website that has to exist for the app to work properly.

## Why does an iOS app need a website?

Three reasons, and Apple insists on all of them before you're allowed to ship to the App Store:

1. **Friend invitations need to land somewhere.** When a user invites a friend by text or WhatsApp, the message includes a link like `https://my-bots.uz/earned/invite/ABC123`. If the friend already has the app, tapping that link opens it directly to the right screen. If they don't, the link opens a friendly "Download Earned" page that bounces them to the App Store. This repository hosts those landing pages.

2. **Family-join codes need to be shareable.** Same flow as invitations, but for joining a family group: `https://my-bots.uz/earned/join/<code>`.

3. **Apple requires a public Privacy Policy and Terms of Service.** The App Store reviewer will reject any app whose policies are not reachable at a stable HTTPS URL. Both documents are written in Markdown (`PRIVACY_POLICY.md`, `TERMS_OF_SERVICE.md`) and rendered as HTML pages.

## What you'll find in this repository

| Folder | What it does |
| --- | --- |
| `invite/` | The "your friend invited you to Earned" landing page. |
| `join/` | The "you've been invited to join a family" landing page. |
| `privacy/` | The Privacy Policy users see when they tap *Privacy* inside the app. |
| `terms/` | The Terms of Service. |
| `.well-known/apple-app-site-association` | A small JSON file that tells iOS "yes, this website is allowed to deep-link into the Earned app." |
| `Caddyfile.new` | A working production web-server config you can drop onto a Raspberry Pi or any Linux box. |
| `PRIVACY_POLICY.md`, `TERMS_OF_SERVICE.md` | The Markdown sources for the legal pages. Edit these when the policy changes. |

## How a user actually experiences this

```
Friend sends you:  https://my-bots.uz/earned/invite/ABC123
                                │
                                ▼
       ┌──────────────────────────────────────────┐
       │  You have the app installed?             │
       └──────────────────────────────────────────┘
              │ yes                       │ no
              ▼                           ▼
    Earned opens, jumps           A friendly page says
    straight to the invite        "You've been invited to
    acceptance screen.            Earned" with a big
                                  Download button. Tap it,
                                  install the app, sign in,
                                  invitation is waiting.
```

## Running it locally

There's nothing to install — every page is plain HTML and CSS. Just open it in a browser:

```bash
python3 -m http.server 8080
# then visit http://localhost:8080
```

For Universal Links to actually trigger the app you need a real HTTPS domain and a physical iPhone (the simulator does not support Universal Links). If something isn't working, the very first thing to check is whether iOS can fetch the AASA file:

```bash
curl -i https://your-host/.well-known/apple-app-site-association
```

It should return `200 OK` with `Content-Type: application/json`.

## Updating the legal pages

When the Privacy Policy or Terms change:

1. Edit `PRIVACY_POLICY.md` or `TERMS_OF_SERVICE.md`.
2. Bump the *Last Updated* date.
3. Re-render to HTML and replace the body of `privacy/index.html` or `terms/index.html`.
4. Commit, push, deploy.

## Hosting

Anywhere that serves static files works — Caddy, Nginx, S3 + CloudFront, Cloudflare Pages, GitHub Pages — as long as:

- HTTPS is enforced (Universal Links won't work over plain HTTP).
- The AASA file is delivered with `Content-Type: application/json` exactly.

A working production configuration is shipped in `Caddyfile.new`.

## License

[MIT](LICENSE).
