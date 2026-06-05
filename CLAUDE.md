# CLAUDE.md — Sweepstake Draw

Persistent context for this project. Claude Code reads this at the start of every session.

## What this is
A single-page web app that runs a **sweepstake draw** — originally built for the FIFA World Cup 2026 (48 teams) but the event name is editable in the UI. It splits the field into three strength pots and allocates teams to players with an animated, screen-recordable reveal. Designed to be shared as a screen recording so a group can watch the draw happen.

The whole app is one file: **`index.html`** (HTML + CSS + vanilla JS, no framework, no build step). Just open it in a browser. Deploy by hosting that one file anywhere static.

## How to run / edit
- **Run:** open `index.html` in any browser. No server or build needed.
- **No dependencies** are bundled. Two things load from the network at runtime:
  - Google Fonts (Bricolage Grotesque + Outfit)
  - Flag images from `https://flagcdn.com/<code>.svg`
- Because flags load over the network, they need an internet connection to appear. There's a graceful fallback: if a flag image fails, it removes itself and only the country name shows. **If offline use is ever needed, embed the flags as base64 data URIs instead of CDN links** (see "Ideas" below).

## The data
48 teams hardcoded in three arrays `POT1`, `POT2`, `POT3` near the top of the script. Each team is `["Country name", "flagcdn-code"]`. England/Scotland use `gb-eng` / `gb-sct`.

Pots are the top / middle / bottom thirds of the field **ranked by bookies' tournament-winner odds** (source: ESPN futures, June 2026). Pot 1 = favourites (Spain, France, England…), Pot 3 = outsiders (…Curacao, Haiti). To update for a different tournament or refreshed odds, replace the three arrays — everything else is generic.

## Randomness (don't weaken this)
Uses `crypto.getRandomValues` with an unbiased rejection sampler (`rnd`) feeding a Fisher–Yates `shuffle`. This is the fairness guarantee and the selling point ("verifiably fair"). The spinning reels are **purely cosmetic** — the result is computed in `allocate()` the instant the draw starts; the animation just lands on the pre-decided outcome. Keep it that way.

## Two draw modes (set in `mode`)
1. **`balanced`** (default) — each player gets an even spread across the three pots. Each pot is shuffled and dealt round-robin (capacity-aware, rotating start per pot).
2. **`open`** — all 48 teams go in one pool, shuffled and dealt round-robin. Pots ignored; pure luck (someone can land all favourites or all minnows).

### Allocation algorithm (`allocate()`)
- `base = floor(48 / N)`, `rem = 48 % N`.
- Everyone gets `base` teams; `rem` **randomly chosen** players get one extra. So totals are equal within 1 and the extra is never biased toward a fixed seat.
- Capacity-aware round-robin (`nextOpen`) fills buckets without ever exceeding a player's cap, and guarantees all 48 teams are dealt exactly once.
- Player names are shuffled and mapped to buckets, so reveal order and name→teams mapping are both random.

### Verified behaviour (tested for N = 2…48)
- All 48 teams dealt, all unique, no dupes, for every N.
- Equal totals within 1.
- **Balanced** keeps each player's pot spread tight (gap 0 when N divides 16 — i.e. N ∈ {2,4,8,16}; small otherwise).
- **Open** has wide pot spread by design (that's the point).

## UI / structure
Three sections toggled by display: `#setup` → `#stage` → `#results`.
- **Setup:** editable event name (live-updates the H1), player-count stepper (2–48, regenerates name fields), mode segmented control, pots preview, live "teams each" calc.
- **Stage:** one player at a time; a flexible grid of cards (`.tcard`), one per assigned team, each a mini slot-reel that spins and locks. Click-to-advance for dramatic pacing during a recording. A running "rail" of already-drawn players builds below.
- **Results:** full table, plus Copy / Download CSV / Draw Again.

## Design conventions
- Fonts: **Bricolage Grotesque** (display) + **Outfit** (body). Don't swap to generic system fonts.
- Palette (CSS vars in `:root`): midnight navy background, **emerald** primary action, **gold** accent. Pot colours: Pot 1 gold `--p0`, Pot 2 emerald `--p1`, Pot 3 blue `--p2`.
- **No brand affiliation.** This was de-branded from an earlier client-branded version — keep it generic/neutral. App wordmark is just "Sweepstake Draw".

## Ideas / possible next steps
- Embed flags as base64 for fully offline/self-contained use.
- Optional "leave leftovers as unallocated spares" instead of giving `rem` players an extra team.
- Save/share a result via URL hash (encode the seed) so a draw is reproducible/auditable.
- Sound effects + a confetti burst on the final reveal.
- A countdown/auto-play option as an alternative to click-to-advance.
- Let pots be edited in-app (paste a list, auto-split into thirds).

## History
- v1: built in Claude.ai as an inline draw machine (fixed 16 players, one team from each of 3 pots, emoji flags).
- v2: rebranded + real flag images.
- v3 (this repo): **de-branded to neutral**, variable player count (2–48), added the **open** draw mode alongside **balanced**, equal-totals handling for non-divisible N, editable event name. Allocation logic unit-tested before wiring into the UI.
