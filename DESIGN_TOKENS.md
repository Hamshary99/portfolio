# Design Spec — FIFA Player Card Portfolio

Hand this file to your coding agent ALONGSIDE the working HTML file
(mohammed-bahey-player-card.html). Tell it: "Port this exact design into
[Next.js / your stack]. This spec documents the source of truth — match
it precisely rather than reinterpreting."

## Fonts (Google Fonts, load via next/font or <link>)
- Display / numbers: **Teko** (weights 400,500,600,700) — used for OVR rating, position, card name, section titles
- Body / page copy: **Manrope** (weights 400,500,600,700,800)
- Labels / stats / mono data: **JetBrains Mono** (weights 400,500,700)

## Color tokens
```css
--bg-deep: #060B18;      /* page background */
--bg-panel: #0B1424;     /* section panels */
--bg-panel-2: #0F1B30;   /* card/project tiles */
--cyan: #38F2C0;         /* accent (sparingly, links/hover) */
--pink: #FF3D81;         /* secondary accent */
--gold: #FFC857;         /* primary card accent, ratings */
--ink: #EAF2FF;          /* primary text on dark */
--dim: #7E93B8;          /* secondary/muted text */
--line: rgba(255,255,255,0.08); /* hairline borders */

/* Card-specific (gold FUT card body) */
--card-gold-1: #E7C077;
--card-gold-2: #C89A4E;
--card-gold-3: #A87B33;
--card-gold-4: #7C5A28;
--card-dark-1: #14100A;  /* stats plate */
--card-dark-2: #0A0805;
```

## The card shape (critical — this is the signature element)
```css
clip-path: polygon(6% 0%, 94% 0%, 100% 7%, 100% 86%, 50% 100%, 0% 86%, 0% 7%);
```
This is a compact shield: flat top with slightly beveled corners, single
point at the bottom. Do NOT use a wavy/scalloped top — that was tried and
rejected as messy. Keep it clean and simple.

**Layout inside the shield (top to bottom):**
1. Rating block (top-left, black text `#1A1206` on gold): big OVR number, position, POT pill
2. Nation flag + club crest (below rating, stacked)
3. Player photo/avatar — large, bottom-anchored to the right side, touching the name band
4. Name band — solid dark bar, full width, no gap/border
5. Stats plate — dark background that flows continuously down through the shield's point (do not stop it short with rounded corners/margins — it should look like one continuous dark shape, not a floating box)
6. Chemistry-style callout — lives OUTSIDE/below the card as its own labeled badge, not crammed inside the point

## Key interaction (keep this — it's the signature move)
- Mouse-move tilt on the card via `rotateX`/`rotateY` transform, reset on mouseleave
- Sparkle field: small ✦ glyphs + dots with staggered `twinkle` keyframe animations scattered around the card
- Soft pulsing radial glow behind the whole card (`coreglow` keyframe) and a tighter pulse behind the avatar (`corepulse` keyframe)

## What NOT to change without asking
- The clip-path shield shape
- The gold→dark plate transition
- The tilt + sparkle + glow interactions
These took several iterations to get right — an agent re-deriving them
from scratch will likely regress the proportions. Copy the CSS values
directly from the source file rather than re-implementing from this
description alone.
