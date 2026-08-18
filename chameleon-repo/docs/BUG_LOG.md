# Magic Bytes — Bug Log

**Status:** All three items below have since been fixed in `chameleon.html`; entries kept as-is for the record, with a Fixed note added to each. See [Retrospective note](#retrospective-note) for the lesson carried forward.
**Related doc:** image-converter-prd.md
**Date logged:** 2026-07-08
**Found by:** Jamie, manual testing of v1

---

## Summary

| ID | Title | Severity | Type | PRD gap? | Status |
|---|---|---|---|---|---|
| BUG-001 | No feedback when Convert All skips unsupported files | Medium | Missing behavior | Yes — batch/mixed-content behavior undefined | Fixed |
| BUG-002 | Clear All / individual remove doesn't remove card from screen | High | Code defect | No — implementation bug | Fixed |
| BUG-003 | Convert All uses each file's target format from add-time, not the live bulk dropdown | High | Logic/UX mismatch | Yes — timing/binding of bulk control undefined | Fixed |

---

## BUG-001 — Silent skip of unsupported files during batch convert

**Repro steps:**
1. Add files that are recognized but unsupported (e.g. HEIC).
2. Click "Convert All."
3. Nothing happens for those files — no message, no status update, no indication anything was attempted.

**Root cause:** `convertAll()` filters to only `sniff.decodable` items before looping, so unsupported items are excluded with no summary and no per-card update beyond the static "not supported yet" note that was already there when the file was added.

**Impact:** At the moment the user actually acts (hits Convert All), there's zero confirmation of what happened or why — the static note shown earlier is easy to forget by the time you're clicking a global button.

**What the PRD didn't specify:** §7.2 (Core Conversion Flow) and §7.3 (Batch Processing) both describe the happy path — add, pick format, convert, download — but never define what should happen when a batch contains a mix of convertible and non-convertible files. This is a genuine gap in the spec, not a contradiction of it.

**Proposed fix direction (not implemented):** Show a run summary after Convert All completes (e.g. "2 converted, 3 skipped — unsupported format"), and update each skipped card's own status area at the moment of the attempt, not just its static add-time note.

**Fixed:** `convertAll()` now sets each skipped card's status to "Skipped — unsupported format" and shows a run summary via `showConvertSummary()` covering converted/failed/skipped/pending counts.

---

## BUG-002 — Clear All / individual remove doesn't remove the card from screen

**Repro steps:**
1. Add files (reproduced with the HEIC files specifically, but not format-specific).
2. Click "Clear all," or the × on an individual card.
3. The card(s) remain visible on screen.

**Root cause:** `removeItem(id)` removes the item from `state.items` correctly, but tries to remove the DOM node via `item.el`. Cards are actually built with the reference stored as `item.cardEl` — `item.el` is never set on any item. So the internal state is correct, but the DOM node is never touched.

**Impact:** State and UI go out of sync. Internal list updates correctly (new conversions work fine regardless), but removed cards stay stuck on screen indefinitely, looking like "remove" silently failed.

**What the PRD didn't specify:** Nothing — this is a straightforward implementation defect (a property-name mismatch introduced during the build), not a requirements gap. Logged for the record, not as a spec issue.

**Proposed fix direction (not implemented):** Fix `removeItem` to reference `item.cardEl`, or standardize on a single property name for the card element throughout the file.

**Fixed:** `removeItem` now references `item.cardEl` (`if (item.cardEl) item.cardEl.remove();`), and `cardEl` is the single property name used throughout.

---

## BUG-003 — Convert All uses each file's target format captured at add-time, not the current bulk dropdown value

**Repro steps:**
1. Add JPG files while the "New files convert to" dropdown is on its default value.
2. Change the "New files convert to" dropdown to PNG.
3. Click "Convert All."
4. Downloaded zip contains files in the *original* format, not PNG — until each file's own per-card dropdown is manually set to PNG, at which point it works as expected.

**Root cause:** `item.targetFormat` is set once, in `addFile()`, from whatever `els.bulkFormat.value` happens to be at that instant. Changing the bulk dropdown afterward has no effect on files already on the page — only editing a card's own `<select>` updates that item's `targetFormat`. `convertAll()` simply reads each item's (possibly stale) `targetFormat` at conversion time.

**Impact:** The "New files convert to" control silently behaves like a one-time default captured at drop-time, rather than a live batch setting — this doesn't match the natural mental model of "set the format up top, hit Convert All," which is how it reads in the UI.

**What the PRD didn't specify:** §7.3 states files should "convert to the same user-selected target format," but never defines the *timing or binding* of that selection — whether it's captured once per file, read live at conversion time, or how a later bulk change should reconcile with a per-card override made in between. This is a real interaction-contract gap, not just an oversight in code.

**Proposed fix direction (not implemented):** Convert All should read the bulk dropdown's current value live and apply it to every card that hasn't been manually overridden; per-card selections should be treated as explicit one-off overrides that persist until changed again.

**Fixed:** the bulk dropdown's `change` listener now pushes its value onto every item where `!item.formatOverridden`, and a per-card override sets that flag so it's excluded from future bulk pushes — matching the proposed fix direction exactly.

---

## Retrospective note

All three of these are the kind of thing that reads as "obvious in hindsight" but wasn't actually nailed down in the requirements: what happens when a batch is mixed (some convertible, some not), what "remove" needs to actually touch end-to-end, and what it means for a global control to apply to items added before vs. after it changes. Worth carrying forward into future PRDs on this project — bulk/batch operations in particular benefit from an explicit "what happens when state changes mid-flow" section, since that's exactly where these three bugs lived.
