# Chameleon

A small, private, entirely client-side image format converter. Drop in a photo, pick a new format, get it back — nothing is ever uploaded anywhere.

## What it does

Converts images between PNG, JPG, WebP, BMP, and HEIC (the format iPhones use by default) — all in the browser, using the Canvas API. Detects the real format by reading a file's actual bytes rather than trusting its extension, supports batch conversion with a zip download, and handles resize/quality controls.

## Status

v1.0 — HEIC support added. Verified via logic tracing and a hand-checked BMP encoder round-trip, but not yet tested across a wide range of real devices/browsers beyond initial iPhone testing.

## Running it locally

Browsers block local `file://` pages from running JavaScript for security reasons, so you can't just double-click `chameleon.html` and expect it to work. Serve it instead:

```
python3 -m http.server 8000
```

Then open `http://localhost:8000/chameleon.html` — or, from another device on the same network, `http://<your-machine's-LAN-IP>:8000/chameleon.html`.

## How it works

- **Format detection**: reads the first bytes of the file (the same technique the Unix `file` command uses) rather than trusting the filename extension.
- **Conversion**: PNG, JPG, and WebP use the browser's native Canvas API. BMP output uses a small hand-written encoder, since browsers don't support encoding to BMP natively. GIF can be read but not written — no browser supports GIF encoding via Canvas, and a from-scratch LZW encoder was judged out of scope for v1.
- **HEIC**: lazy-loads `heic2any` (WASM-based) only when a HEIC file is actually added, converts it to a PNG blob, then runs it through the same pipeline as everything else.
- **Batch**: multiple files convert to a shared target format (or per-file overrides), packaged into a zip via JSZip.
- **Privacy**: no backend, no uploads. The only network calls are for fonts and the two small libraries above.

## Docs

- [Product Requirements Document](docs/PRD.md)
- [Bug Log](docs/BUG_LOG.md)

## License

Not yet decided — this repo is private for now, so it hasn't been a pressing question. Worth revisiting before this ever goes public.
