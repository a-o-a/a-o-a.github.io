# CLAUDE.md

Guidance for AI assistants working on this repository.

## What This Is

`a-o-a.github.io` is a **single-file static landing page** served via GitHub Pages.

- **One file only**: `index.html` — all HTML, CSS, and JS live inline in this file.
- **No build step**, no framework, no dependencies, no package manager.
- Deployed automatically by GitHub Pages on every push to `main`.

## Stack

| Layer | Technology |
|---|---|
| Structure | HTML5 (single file) |
| Styles | Inline `<style>` with CSS custom properties |
| Scripting | Inline `<script>`, vanilla JS (< 10 lines) |
| Font | Google Fonts — Space Grotesk 700 |
| Hosting | GitHub Pages (direct HTML serve) |

## File Structure

```
/
└── index.html   ← the entire site
```

That's it. Do not add `_config.yml`, `Gemfile`, `package.json`, or any other files unless the user explicitly decides to introduce a build tool.

## Design System (CSS Custom Properties)

All visual values are defined as CSS variables on `:root` at the top of `<style>`. Edit these to change the look — do not scatter magic values into selectors.

**Token groups:**

| Group | Prefix | Examples |
|---|---|---|
| Colors — text | `--text-*` | `--text-primary`, `--text-muted`, `--text-soft` |
| Colors — sun | `--sun-*` | `--sun-core`, `--sun-mid`, `--sun-edge` |
| Colors — sun glow | `--sun-glow-*` | `--sun-glow-inner`, `--sun-glow-outer` |
| Colors — sunset | `--sunset-*` | `--sunset-orange`, `--sunset-pink`, `--sunset-purple` |
| Colors — clouds | `--cloud-*` | `--cloud-fill`, `--cloud-lobe-fill` |
| Colors — waves | `--wave-*` | `--wave-1`, `--wave-2`, `--wave-3` |
| Colors — aurora | `--aurora-*` | `--aurora-a`, `--aurora-b` |
| Colors — mist | `--mist-*` | `--mist-a`, `--mist-b` |
| Durations | `--*-duration*`, `--*-cycle` | `--cloud-duration-1`, `--sun-cycle`, `--wave-duration-1` |

## Scene Architecture

The page renders a layered animated sunset scene. The DOM tree:

```
<main.scene>
  <div.scene__decor aria-hidden>   ← all visual decoration
    .sunset                         ← radial-gradient sky backdrop
    .sky-glow                       ← white bloom behind the sun
    .aurora.aurora-1/.aurora-2      ← drifting ambient color blobs
    .mist.mist-1/.mist-2            ← subtle atmospheric haze
    .cloud.cloud-1/.cloud-2/.cloud-3
    .sun                            ← the glowing disc
    .waves                          ← 3 layered SVG wave paths
  </div>
  <section.scene__content>          ← foreground text (z-index: 1)
    <h1.title#page-title>
  </section>
</main>
```

**Layer order** (back → front): sunset → sky-glow → aurora → mist → clouds → sun → content → waves (waves overlay the bottom via absolute positioning).

## Animations

Every animation is a pure CSS `@keyframes`. Durations are controlled via `:root` variables — change the variable, not the animation.

| Element | Keyframe | Variable |
|---|---|---|
| Sunset glow | `sunset-pulse` | `--sun-cycle` |
| Sun disc | `sun-glow` | `--sun-cycle` |
| Clouds | `cloud-drift-1/2/3` | `--cloud-duration-1/2/3` |
| Waves | `wave-move-1/2/3` | `--wave-duration-1/2/3` |
| Sky bloom | `glow-shift` | hardcoded 18s |
| Aurora blobs | `dream-float` | hardcoded per element |
| Mist | `mist-flow` | hardcoded 22–26s |
| Title text | `shimmer` | `--text-shimmer-duration` |

The JS block (bottom of `<body>`) only randomises cloud `animation-delay` so they don't all start at the same position.

`prefers-reduced-motion` is handled: all animations collapse to `0.01ms / 1 iteration`.

## Conventions

- **CSS custom properties first**: any new color, timing, or size that might be tweaked belongs in `:root`, not inline.
- **`transform: translate3d()` over `translate()`**: used throughout for GPU compositing.
- **`will-change`**: applied only on elements that animate `transform` or `box-shadow`. Don't add it speculatively.
- **`pointer-events: none`** on all decorative elements (`.scene__decor`, `.mist`, `.waves`).
- **Accessibility**: decorative elements carry `aria-hidden="true"`. The `<main>` has `aria-label`; the `<section>` uses `aria-labelledby` pointing to `#page-title`.
- **No IDs for styling**: IDs (`#page-title`) are only for accessibility references.
- **`clamp()` for fluid text**: `font-size: clamp(1.2rem, 5.8vw, 2.6rem)` — use this pattern for responsive sizes.
- **Language**: the page is `lang="ko"` (Korean).

## Making Changes

### Edit text
Change the `<h1>` content inside `<section.scene__content>`.

### Change colors
Edit the relevant `--variable` in `:root`. All uses update automatically.

### Change animation speed
Edit `--cloud-duration-*`, `--wave-duration-*`, or `--sun-cycle` in `:root`.

### Add a new decorative element
1. Add markup inside `<div.scene__decor aria-hidden="true">`.
2. Add CSS with `position: absolute`, `pointer-events: none`.
3. Use an existing `--variable` for color/timing, or add a new one to `:root`.

### Add a wave layer
Copy an existing `.wave-N` `<svg>` and `.wave-N` CSS block; adjust the SVG path `d` attribute and `z-index`.

## Development

No build step. Open `index.html` in a browser — that's the full dev environment.

For a live-reload server:
```bash
npx serve .
# or
python3 -m http.server 8000
```

GitHub Pages serves the file as-is from `main`. No CI/CD configuration needed.

## What NOT to Add

- Do not add `Gemfile`, `_config.yml`, or other Jekyll files — GitHub Pages will try to run Jekyll and may fail.
- Do not add `package.json` or a bundler unless the user explicitly wants a build pipeline.
- Do not split CSS/JS into separate files unless complexity clearly demands it.
- Do not add `_site/`, `.sass-cache/`, or other build artifacts.
