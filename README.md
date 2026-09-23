# Lookout — QR Scanner

A full-screen QR scanner that opens straight into the camera. Before going
anywhere, it shows exactly what a scanned code contains and lets you choose
to **open the link** or just **view/copy the URL**.

It's a single self-contained HTML file — no build step, no server-side code,
no separate files to host.

## Files

```
lookout-qr-scanner.html   the entire app
```

That's it. The manifest, icons, and service worker needed for PWA install
are all embedded inside this one file as base64 data — nothing else needs
to be uploaded alongside it.

## Deploying it

Upload `lookout-qr-scanner.html` to any static host **as `index.html`**, at
the site root. Two things matter for it to work:

- **HTTPS is required.** Browsers block camera access and service workers
  on plain `http://` — the only exception is `http://localhost` during local
  testing. Netlify, Vercel, GitHub Pages, and Cloudflare Pages all give you
  HTTPS automatically.
- **Serve it at the root as `index.html`**, so the URL is clean
  (`yoursite.com/`, no `.html` in it) and the install prompt appears.

Quickest options:
- **Netlify** — drag the file onto [app.netlify.com/drop](https://app.netlify.com/drop) (rename it to `index.html` first)
- **Vercel** — `vercel` from a folder containing it as `index.html`
- **GitHub Pages** — commit it as `index.html` and enable Pages

## Testing locally

Opening the file directly (`file://`) will **not** work — camera access
requires a real server, even a local one. From a folder containing the file
(named `index.html`), run:

```bash
npx serve .
```

then open `http://localhost:3000` — localhost counts as secure, so the
camera and service worker both work there too.

## How scanning works

The app tries two ways to read a QR code, in order:

1. **The browser's built-in scanner** (`BarcodeDetector` API) — available on
   Chrome, Edge, and other Chromium browsers. No download, hardware-accelerated.
2. **The `jsQR` library, loaded from a CDN** — used automatically on browsers
   without a built-in scanner (Firefox, Safari). It tries three different
   CDN mirrors (cdnjs, jsdelivr, unpkg) in sequence, so a single blocked or
   down CDN won't stop it from working.

If neither is available (e.g. offline with no built-in scanner), the app
says so plainly instead of failing silently.

## Installing as an app

Once deployed over HTTPS, visiting the site on a phone shows an "Add to
Home Screen" / "Install" prompt — automatic on Android/Chrome, or via
Share → Add to Home Screen on iOS/Safari. It then opens full-screen, without
browser chrome, using the reticle icon embedded in the manifest.

## Troubleshooting

**"Camera access needed" / permission denied**
The site isn't on HTTPS (or localhost), or the browser has camera access
blocked for this site — check the address bar for a blocked-camera icon.

**"No camera could be reached"**
The device has no camera, another app/tab already has it open, or your OS
hasn't granted the browser camera access at the system level (macOS:
System Settings → Privacy & Security → Camera; Windows: Settings → Privacy
& security → Camera).

**"Scan unavailable" / "No QR reader could be loaded"**
Neither the built-in scanner nor any of the three `jsQR` CDN mirrors were
reachable — usually a firewall, ad blocker, or fully offline connection
blocking all of them. Camera preview still works; only the automatic
decoding is affected.

## Customizing

- **Colors, type, layout** — all in the `<style>` block near the top of the
  file, defined once as CSS variables (`--ink`, `--phosphor`, `--amber`,
  etc.) so a full re-theme only means changing a handful of values.
- **App name/icon** — the manifest and icons are embedded as base64; edit
  the source project's `build.py` / `gen_icons.py` and rebuild rather than
  hand-editing the encoded strings.
- **QR-only vs other formats** — the native detector is currently scoped to
  `formats: ['qr_code']` in the script; broaden that array to add other
  barcode formats the API supports.
