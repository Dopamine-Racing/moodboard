# Dopamine Racing — Mood Board V3 Handoff Brief

## TL;DR

The mood board is functional but ~half the product images don't load. Root cause: image URLs were extracted from Shopify HTML by an LLM, which was unreliable (hallucinated filenames, mismatched products). Fix: re-fetch image URLs from Shopify's own JSON API (`/products/{handle}.json`), which is canonical and reliable.

Everything else works — filters, search, save/compare, shortlist, categorisation. Don't rebuild from scratch.

---

## What's built

V3 mood board for picking shells with Shane. Deployed to `https://dopamine-racing.github.io/moodboard/`. Repo: `Dopamine-Racing/moodboard`.

- **186 shells** from SuperG's filtered catalogue (Lexus, Skyline, Toyota, Mazda, Nissan, Silvia, Supra, RX-3) plus 3 Pandora-only shortlist extras
- **23 marques**, **20 manufacturers**, **10 aesthetics**, **6 era buckets** — all categorised heuristically from product titles
- **Filter UI**: search box, marque dropdown, mfr dropdown, aesthetic chips, era chips, view chips (All / My picks / Shortlist★), live result count
- **Modes**: pick shells, save picks (URL-encoded share link), compare two URLs side-by-side with overlap highlighting
- **Shortlist ★**: 3 Pandora shells pinned to top (PAB-3199 GR86 BLS, PAB-3218 GR86 BN-Sport, PAB-3135 Supra Mk4)
- Tuner-shop aesthetic: IBM Plex Mono, #FACC15 yellow accent, dark background
- Pure single-file HTML — no build step, no framework, just HTML/CSS/JS

## What's broken

Image rendering. Roughly half the cards show "IMAGE UNAVAILABLE" or "NO IMAGE" because their URLs are dead or wrong. Specifically:

1. **45 shells** had clearly-fabricated URLs (Firecrawl LLM hallucinated filenames with shared `?v=1697998890` and `?v=1706118137` timestamps). I cleared those — they now show "NO IMAGE" placeholder.
2. **~50 more shells** have URLs that look plausible (passed my checks) but apparently still fail in your browser. These need verification.
3. **3 shortlist shells** had Pandora WordPress URLs I made up — replaced 2 with SuperG equivalents; PAB-3135 still shows "NO IMAGE" because it's not in SuperG's catalogue.

## Recommended fix

Use Shopify's product JSON API instead of HTML scraping. Every Shopify product has a canonical JSON at:

```
https://supergdrift.com/products/{handle}.json
```

Example: `https://supergdrift.com/products/nissan-skyline-bnr32-4dr-sedan-r32-1-10-body-set-pandora-pab-3232.json`

Returns structure like:
```json
{
  "product": {
    "id": ...,
    "title": "...",
    "handle": "nissan-skyline-bnr32-...",
    "images": [
      { "src": "https://supergdrift.com/cdn/shop/files/bnr32-skyline-gtr-02.jpg", "position": 1, ... },
      ...
    ]
  }
}
```

Take `product.images[0].src` for the main image. No LLM, no extraction, no guessing.

## Files (read before doing anything)

All paths below are in this download:

| File | What it is | What to do with it |
|---|---|---|
| `HANDOFF.md` | This file | Read first |
| `index.html` | Current V3 mood board (functional, broken images) | Keep — only the SHELLS array's `imageUrl` field needs updating |
| `shells_v3.json` | The 186 shells with code/name/marque/mfr/aesthetic/era/tags/shortlist/imageUrl | Source of truth for the SHELLS array. Update `imageUrl` field, then regenerate the HTML by replacing the `const SHELLS = [...]` block in index.html |
| `superg_handles.json` | 160 verified `{handle, title, imgSrc}` triples from re-scraping SuperG's catalogue | Best source for matching shells to product handles. Match a shell's code (or name keywords) against the title field to find its handle, then use that handle with the `/products/{handle}.json` API |
| `handles.txt` | Partial code→handle map (only 45 matched via regex) | Convenience reference; superg_handles.json is more useful |

## Exact prompt for Claude Code

```
I'm picking up a project from a previous Claude session. Read HANDOFF.md first.

The mood board (index.html) is functional but image URLs are broken on roughly
half the cards. Goal: get >90% of cards showing real product images.

Approach:

1. Read HANDOFF.md, shells_v3.json, and superg_handles.json
2. For each shell in shells_v3.json:
   a. Find its handle by matching its code against titles in superg_handles.json
      (the code appears in the title text, e.g. "[Pandora] PAB-3232")
   b. If matched, fetch https://supergdrift.com/products/{handle}.json
   c. Take product.images[0].src as the new imageUrl
   d. If no handle match in superg_handles.json, try:
      - Searching SuperG: https://supergdrift.com/search?q={code}
      - Or constructing the handle from the product name (Shopify handles are
        lowercase, hyphen-separated, alphanumeric only)
   e. If still no luck, leave imageUrl empty so the card shows "NO IMAGE"
3. Rate-limit at 1-2 requests per second to be polite to SuperG
4. Update shells_v3.json with the new URLs
5. Regenerate index.html by replacing the `const SHELLS = [` array (line ~830)
   with the updated data. Format matches the existing entries exactly.
6. Open the resulting index.html in a browser and visually confirm. Aim for >90%
   cards with images.

Don't touch the filter UI, the JS, the categorisation, or the shortlist flags.
Only update imageUrl values.
```


## Things NOT to rebuild

- Categorisation (marque/mfr/aesthetic/era) — heuristic but good enough; I checked it
- Filter UI — works in headless Chrome test, all interactions verified
- Save/Compare/Share URLs — V2 functionality preserved
- The 3 shortlist flags (PAB-3199, PAB-3218, PAB-3135)
- IBM Plex Mono + yellow accent design language

## Known weak spots beyond images

- **PAB-3199 and PAB-3135** are categorised as "Other" era — should be Modern (2010s+) and 1990s respectively. Edit `shells_v3.json` if you care.
- **28 other shells** are "Other" era because their titles didn't have a clear chassis cue. Not a bug, just heuristic limits.
- **Some product handles in shells_v3.json may be missing** because I deduplicated by code, not handle. If a handle isn't in `handles.txt`, you'll need to derive it from the product name or search SuperG.

## What I'd do differently

Skip Firecrawl for structured Shopify data. The products.json endpoint was always the right answer — I should have used it from the start instead of scraping HTML through an LLM extraction layer.
