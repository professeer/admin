# TheProfesseer — Logo Kit

Brand marks for TheProfesseer. Two forms, three color treatments, two file
formats. Pick by **what you're placing** (icon vs. full logo) and **what's
behind it** (light vs. dark background).

## The two forms

| Form | Folder | What it is | Use for |
|------|--------|-----------|---------|
| **Lockup** (logo + text) | `lockup/` | Owl mark + "The Professeer" wordmark | Documents, decks, site header, email signature, letterhead — anywhere with room for the name |
| **Mark** (logo only) | `mark/` | Owl mark on its own | Favicon, app icon, social avatar, watermark, tight spaces where the wordmark would be illegible |

Rule of thumb: if the mark would render smaller than ~24px tall, or the brand
name already appears nearby, use the **mark**. Otherwise use the **lockup**.

## The three color treatments (light vs. dark)

| Treatment | File suffix | Background | Notes |
|-----------|-------------|-----------|-------|
| **Crimson** (primary) | `-crimson` | Light / white / off-white | The default brand look. `#7B0925`. |
| **White** (reversed) | `-white` | Dark / crimson / photo | Use on the crimson band or any dark surface. |
| **Ink** (monochrome) | `-ink` | Light, single-color only | Black `#141414` for faxable/B&W/print-one-color contexts. |
| *(default)* | *(none)* | — | `professeer-logo.svg` / `professeer-mark.svg` = full-color master. |

Never place the crimson logo on a dark background or the white logo on a light
one — contrast fails. When unsure, the safe pairing is crimson-on-light,
white-on-dark.

## File formats

- **SVG** — vector master. Use everywhere you can (web, slides, LaTeX, print).
  Scales to any size with no quality loss. This is the source of truth.
- **PNG** — raster with transparent background, ~1200px tall. Use only where SVG
  isn't accepted (some chat apps, raster-only tools).

## Clear space & minimum size

- **Clear space:** keep padding equal to the height of the owl's head on all
  sides; don't crowd the mark with other elements.
- **Minimum size:** lockup ≥ 120px wide; mark ≥ 16px. Below that, switch form or
  drop detail.
- **Don't:** recolor, stretch, rotate, add shadows, or place on a busy photo
  without a scrim.

## What this kit does NOT yet include (gaps to fill)

These are standard in a complete brand kit but aren't in the repo yet — worth
generating from the SVG masters:

- **Favicon set** — `favicon.ico` plus 16/32/48px PNGs, and a 180px
  `apple-touch-icon.png`.
- **Social avatar** — square (512×512) PNG of the mark with safe padding, for
  GitHub / X / LinkedIn org profiles.
- **OG / share image** — 1200×630 banner using the lockup.
- **Print PDF** — vector PDF of lockup + mark for printers that reject SVG.
- **Mono-on-color knockout** — white mark inside a crimson circle/square tile.
