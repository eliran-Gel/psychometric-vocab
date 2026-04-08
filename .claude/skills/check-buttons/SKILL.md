---
name: check-buttons
description: This skill should be used when the user asks to "check all buttons", "test buttons", "verify flashcard buttons work", "בדוק כפתורים", or reports that classification buttons (know well / partially know / don't know) are not working in Hebrew or English. Covers the full diagnostic checklist for the psychometric-vocab site.
version: 1.0.0
---

# Button Diagnostic Skill — psychometric-vocab

Covers both Hebrew and English flashcard classification buttons.

## Buttons to verify

### Hebrew flashcards
| Button | Calls | Expected result |
|---|---|---|
| ❌ לא יודע | `rateCard(0)` | word stays in list, added to mistakes |
| 🔶 יודע חלקית | `rateCard(1)` | word appears in "יודע חלקית" tab (progress.correct >= 1) |
| 🌟 יודע טוב | `markHebrewKnownWell()` | word removed from flashcard list, appears in "יודע טוב" tab |
| ↩ החזר לתרגול | `unmarkKnownWell(word)` | word removed from known_well, re-added to flashcard list |
| 🌟 יודע טוב (from partial tab) | `promoteWordToKnownWell(word)` | word moves from partial tab to well tab |
| ↩ אפס (from partial tab) | `resetWordProgress(word)` | word's progress deleted, removed from partial tab |

### English flashcards
| Button | Calls | Expected result |
|---|---|---|
| ❌ Don't know | `rateEngCard(0)` | word stays in list, added to eng_mistakes |
| 🔶 Partially know | `rateEngCard(1)` | word appears in "Partially Known" tab (eng_progress.correct >= 1) |
| 🌟 Know it well | `markEngKnownWell()` | word removed from flashcard list, appears in "Know Well" tab |
| ↩ Move to Partial | `unmarkEngKnownWell(word)` | word removed from eng_known_well, appears in partial tab |
| 🌟 Know Well (from partial tab) | `promoteEngWordToKnownWell(word)` | word moves from partial tab to well tab |
| ↩ Reset (from partial tab) | `resetEngWordProgress(word)` | word's eng_progress deleted, removed from partial tab |

## Known past bugs (already fixed)

### Bug 1 — `applyEngFilter` not excluding known-well words
- **Symptom**: "Know it well" marked the word but it stayed in the flashcard list
- **Root cause**: `applyEngFilter()` built `filteredEngWords` without filtering `getEngKnownWell()`
- **Fix**: Add `filteredEngWords = filteredEngWords.filter(w => !getEngKnownWell().includes(w.word));` after building the list
- **Compare with Hebrew**: `applyFilter()` does `filteredWords = filteredWords.filter(w => !knownWell.includes(w.word));` ✓

### Bug 2 — `ENG_VOCAB` undefined variable
- **Symptom**: "Partially Known" and "Know Well" tabs showed nothing (silent ReferenceError)
- **Root cause**: `renderEngKnownWell()` and `renderEngKnownPartial()` called `ENG_VOCAB.filter(...)` but the constant is named `ENGLISH_VOCAB`
- **Fix**: Replace all `ENG_VOCAB` → `ENGLISH_VOCAB`

### Bug 3 — `markEngKnownWell` used undefined variable
- **Symptom**: "Know it well" button silently failed (early return)
- **Root cause**: Used `engFilteredWords[engFcIndex]` — variable doesn't exist; it's `filteredEngWords`
- **Fix**: Change to `filteredEngWords[engFcIndex]`

### Bug 4 — `updateEngSR` never incremented `correct`
- **Symptom**: "Partially know" clicks never showed words in partial tab
- **Root cause**: `if (quality >= 3) progress[word].correct++` — buttons only send 0/1/2
- **Fix**: Change to `if (quality >= 1) progress[word].correct++`

## Diagnostic checklist

When buttons seem broken, check in this order:

1. **Variable names** — search for references to arrays like `filteredEngWords`, `filteredWords`, `ENGLISH_VOCAB`, `VOCAB`. Any typo causes a silent `undefined` early return.
2. **`applyFilter` / `applyEngFilter`** — confirm they filter out known-well words from the list.
3. **`updateSR` / `updateEngSR`** — confirm the quality threshold matches what buttons actually send (0/1/2 for Hebrew; 0/1/2 for English).
4. **Render functions** — confirm `renderEngKnownWell` and `renderEngKnownPartial` use `ENGLISH_VOCAB`, not `ENG_VOCAB` or any other alias.
5. **`getData` / `setData`** — if `isGuestMode` is true, `setData` is a no-op; data won't persist.

## Key localStorage keys

| Key (with `pv_` prefix via `sKey()`) | Content |
|---|---|
| `known_well` | `string[]` Hebrew words marked "know well" |
| `progress` | `{[word]: {correct, wrong, lastSeen, interval, nextReview}}` Hebrew SR |
| `eng_known_well` | `string[]` English words marked "know well" |
| `eng_progress` | `{[word]: {correct, total, date}}` English partial tracking |
| `eng_srs` | `{[word]: {n, ef, interval, due}}` English SM-2 data |
