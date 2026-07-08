# Chameleon — Product Requirements Document

**Status:** v1.0 — Scope Locked
**Owner:** Jamie
**Date:** 2026-07-08

---

## 1. Overview

A lightweight tool called **Chameleon** that lets end users convert an image from one file format to another (e.g., HEIC → JPG, PNG → WebP) without needing to install software, understand codecs, or figure out which app can open which format.

## 2. Problem Statement

Users regularly hit a wall where an image file "just doesn't work" — a phone photo (HEIC) won't upload to a web form that only accepts JPG/PNG, a PNG with transparency breaks a print workflow that needs JPG, or a file is simply too large/wrong-format for a target system. Most existing solutions are either sketchy ad-laden websites that upload your photos to an unknown server, or overkill desktop software. There's room for something simple, fast, and trustworthy.

## 3. Goals

- Let a non-technical user convert an image between common formats in under 30 seconds, with zero learning curve.
- Be trustworthy — clear about where files go and whether anything is stored.
- Be genuinely simple: one primary action (convert), minimal UI chrome.

## 4. Non-Goals (v1)

- Not a full image editor (no cropping/filters/effects beyond basic resize, at least in v1).
- Not a DAM (digital asset management) or file organization tool.
- Not targeting professional RAW/photography workflows in v1.

## 5. Target Users

- Primary: Jamie, personal use — the "I hit a wrong-format error and need this fixed now" moment.
- Secondary (open-ended): general end users / church or work contexts, if it turns out to be useful there too. Not designed around any one group's specific needs for v1 — kept generic on purpose so it's easy to reuse.

**Decision:** Personal-first, general-purpose. No group-specific requirements (auth, branding, org-specific workflows) baked into v1.

## 6. User Stories

- As a user, I can drag and drop (or select) an image file and immediately see what format it is.
- As a user, I can pick a target format from a short list of supported outputs.
- As a user, I can click Convert and download the result without creating an account.
- As a user, I can select multiple files at once, convert them all to the same target format, and download them together (v1 requirement — see §7.3).
- As a user, I trust that my image isn't being uploaded/stored somewhere I don't know about (or I'm clearly told if it is).

## 7. Functional Requirements

### 7.1 Supported Formats

| Format | Read (input) | Write (output) | v1 scope | Notes |
|---|---|---|---|---|
| PNG | ✅ | ✅ | ✅ In v1 | Native browser support |
| JPG/JPEG | ✅ | ✅ | ✅ In v1 | Native browser support |
| WebP | ✅ | ✅ | ✅ In v1 | Native browser support (modern browsers) |
| GIF | ✅ (static only) | ✅ | ✅ In v1 | Static only — animated GIF is a separate, harder problem, deferred |
| BMP | ✅ | ✅ | ✅ In v1 | Native browser support |
| ICO | ✅ | ⚠️ | ❌ Deferred | Less standard; may need a small library |
| HEIC/HEIF (iPhone photos) | ⚠️ | ⚠️ | ❌ Deferred to v2 | Not natively supported by browsers — requires a WASM library (e.g. `heic2any`) or server-side processing |
| TIFF | ⚠️ | ⚠️ | ❌ Deferred | Not native — requires library or server-side |
| PDF (image page → image) | ❌ | ❌ | ❌ Out of scope | Different category of problem (PDF, not image) |
| RAW (camera formats) | ❌ | ❌ | ❌ Out of scope | Specialist tooling territory |

**Decision:** v1 ships native-browser-support formats only — PNG, JPG/JPEG, WebP, static GIF, BMP. HEIC and everything else deferred to a later version.

### 7.2 Core Conversion Flow

1. User provides an image (drag-drop, file picker, or paste from clipboard).
2. App detects source format and shows a small preview + file size.
3. User selects target format from available outputs.
4. (Optional) User sets quality/compression level for lossy formats (JPG/WebP).
5. User clicks Convert.
6. App produces the converted file and offers a download.

### 7.3 Batch Processing

**Decision:** Batch is a v1 requirement.

- User can select/drop multiple files at once.
- All files convert to the same user-selected target format.
- Files are packaged into a single `.zip` for download (single-file output can still just download directly, no zip needed for a batch of one).
- Client-side zip packaging via a small library (e.g., `JSZip`) — still no backend, still nothing leaves the device.
- Progress indicator per-file and overall, since batch runs will take longer than a single conversion.

### 7.4 Image Adjustments (stretch, not core)

- Resize / max dimension constraint (useful for "file too large" cases).
- Basic quality/compression slider for JPG/WebP output.

These are cheap to add on top of a canvas-based conversion pipeline, so worth considering for v1 even though they're not core to "format conversion."

## 8. Non-Functional Requirements

### 8.1 Privacy & Data Handling

**Decision:** Client-side only. All conversion happens in the browser via the Canvas API; the image and its data never leave the user's device, and batch zip packaging happens locally too. No server, no upload, no storage, no liability. This also means HEIC/TIFF support (which would need a WASM library or backend) is deferred rather than compromising the "nothing leaves your device" guarantee for v1.

### 8.2 Performance

- Conversion should feel instant for typical photo sizes (under ~10MB) — target under 2 seconds.
- Large files (e.g., 50MB+ scans) should show a progress indicator rather than freezing the UI.

### 8.3 Browser / Platform Support

- Modern evergreen browsers (Chrome, Firefox, Edge, Safari — desktop and mobile).
- No IE11 support needed.

### 8.4 Limits

- Max file size: TBD (suggest 25–50MB as a reasonable v1 ceiling for a client-side tool).
- No account/login required.

## 9. Technical Approach — Options

**Option A: Pure client-side, Canvas API**
Use the browser's native `<canvas>` + `toBlob()` to decode/re-encode images. Covers PNG, JPG, WebP, BMP, static GIF natively. Zero backend, zero cost to run, fastest to build. Does **not** cover HEIC or TIFF.

**Option B: Client-side + WASM libraries**
Add a library like `heic2any` (HEIC→JPEG/PNG in-browser via WASM) to close the biggest real-world gap. Still no backend. Adds bundle size and a bit of complexity.

**Option C: Server-based (Node + sharp)**
A small backend (e.g., a Node/Express or serverless function using `sharp`) handles anything the browser can't, including HEIC, TIFF, and more exotic formats, plus more reliable/faster processing for large files. Costs hosting, needs a data-handling policy, but is the most capable long-term option — and you already have relevant hosting (IONOS, Railway) and patterns from JedForge to reuse.

**Decision:** v1 uses **Option A** — pure client-side, Canvas API, plus `JSZip` for batch download packaging. No WASM libraries, no backend. This matches the single-file, no-dependency (or minimal-dependency) pattern used for the Bible reading guide and sermon browser. Option B (HEIC via WASM) and Option C (backend) are both explicitly deferred — see roadmap in §14.

## 10. UI/UX Requirements

- Single-page, minimal UI: drop zone → format picker → convert button → download.
- Clear, plain-language copy (no jargon like "codec" or "lossless" without a plain-English explanation).
- Visible reassurance about privacy — "Your images never leave your device" — since v1 is client-side only.
- Mobile-friendly — many users will hit this problem on their phone (e.g., trying to upload a HEIC photo somewhere).

## 11. Success Metrics

Personal-use utility first, not a monetized product for v1:
- It works reliably for the formats you personally need most often.
- It's something you'd actually hand a non-technical friend or church member and trust them to use without instructions.
- Reusability is a soft goal, not a hard requirement — built generically enough that repurposing it later (church, portfolio) wouldn't require a rewrite.

## 12. Out of Scope (v1)

- User accounts / saved history
- Animated GIF conversion
- RAW camera formats
- PDF-to-image or image-to-PDF
- Bulk folder upload beyond simple multi-select

## 13. Decisions Log

All open questions from the initial draft are resolved:

1. **App type:** Single-file browser tool (no framework, no build step) — same pattern as the Bible reading guide.
2. **Format scope for v1:** Native-only — PNG, JPG/JPEG, WebP, static GIF, BMP. HEIC/TIFF/others deferred.
3. **Architecture:** Client-side only. No backend, nothing leaves the device.
4. **Batch conversion:** v1 requirement — multi-file select, batch convert, zip download.
5. **Audience:** Personal use first; built generically enough to reuse elsewhere later if useful.
6. **Hosting/branding:** None for now — a local file/artifact, not deployed anywhere.

## 14. Phased Roadmap

- **v1 (this build):** Single-file HTML tool. Client-side only (Canvas API + JSZip). Formats: PNG, JPG/JPEG, WebP, static GIF, BMP. Multi-file batch conversion with zip download. Optional resize/quality controls (§7.4) if low-cost to include.
- **v2 (future, not scoped yet):** HEIC support via WASM (`heic2any`), possibly TIFF.
- **v3 (future, only if warranted):** Backend-assisted conversion for RAW/PDF or other formats browsers can't handle, and/or actual hosting/deployment if the tool proves useful beyond personal use.
