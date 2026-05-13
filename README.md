# Dopamine Racing — Shell Mood Board

Static, single-file HTML mood board for narrowing down the drift shell shortlist (52 options across Nissan, Mazda, Toyota). URL-encoded state, zero backend.

## Deploy

1. Drop `index.html` somewhere your GitHub Pages site serves from — e.g. `/moodboard/index.html` in your `dopamineracing` repo, with Pages configured to serve from `main` branch.
2. Hit `https://gavinmcfall.github.io/dopamineracing/moodboard/` (or wherever).
3. Send the URL to Shane.

## Use

**Pick mode**
- Pick your name in the top-right dropdown (Gavin or Shane).
- Tap any card to mark it as a pick. Picks save to your browser's localStorage, so they survive refresh.
- Use filters at the top to narrow by marque, aesthetic tier, or "My picks only".
- "Reset" clears all your picks (with a confirm prompt).

**Share**
- "Save my picks" generates a URL containing your name and pick codes — copy and send it to the other person.
- URL format: `…/index.html#u=gavin&p=PAB-3195.PAB-3194.PAB-3148`

**Compare**
- Open the page (with your own picks already made).
- Hit "Compare", paste the other person's URL.
- See three columns: my picks, both-of-us, their picks. Overlaps highlighted in yellow.

## What's in V1

- All 52 shells embedded as JS data.
- Code, name, marque, aesthetic tier (Origin Labo / BN Sports / Pandem / OEM+), era, manufacturer.
- The 3 from the existing Notion shortlist marked with ★.
- No images yet — adding 52 product shots properly would have dragged this out. V2 task: each shell has an unused `imageUrl` slot in the data, ready to fill in.

## Tech notes

- Single file, no build step, no dependencies. Just IBM Plex Mono via Google Fonts.
- State: localStorage for persistence, URL hash for sharing.
- Editing shells: search for `const SHELLS = [` in the file and edit the array. Each entry takes `code`, `name`, `marque`, `aesthetic`, `era`, `mfr`, optional `note`, optional `shortlisted: true`.
- Adding images later: add `imageUrl: '...'` to each shell, then update the card template inside `renderGrid()` to include an `<img>`.

## Known limitations

- "Login" is a name dropdown — no actual auth. Anyone with the URL can browse and pick. Fine for two friends, not for anything sensitive.
- localStorage is per-browser. If you switch devices, your picks don't follow — re-import via the URL you saved.
- URLs get long with many picks (~10 bytes per code). Browsers handle several thousand chars fine, but for 50+ picks consider sharing the picks list directly instead of the URL.
