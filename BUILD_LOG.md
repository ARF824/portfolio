# Build Log — polish/audit-fixes

Hardening pass on the Create Networks static portfolio site. All changes are on branch `polish/audit-fixes`. No pushes to remote.

---

## Phase 1 — Progressive-enhancement fallback

**Problem:** `.reveal` hid content via `opacity: 0` unconditionally. Users with JavaScript disabled would see a blank page.

**Fix:**
- Added `<script>document.documentElement.classList.add('js');</script>` as the first child of `<head>` in `index.html`.
- Changed the `.reveal` rule in `style.css` to `html.js .reveal { … }` so the hide-before-reveal only applies when JS is confirmed running.
- `.reveal.visible` and `.d1`–`.d6` delay rules left unchanged.

**Commit:** `phase 1: progressive-enhancement fallback for .reveal`

---

## Phase 2 — Reduced motion + spiral pause

**Problem:** The Fibonacci spiral animation ran continuously regardless of user motion preferences and kept consuming CPU even when scrolled off-screen or the browser tab was hidden.

**Fix:**
- Added `@media (prefers-reduced-motion: reduce)` block at the end of `style.css` that zeroes all animations/transitions and forces `.reveal` elements visible.
- Rewrote the spiral IIFE in `index.html` to:
  - Check `window.matchMedia('(prefers-reduced-motion: reduce)')` — if true, draws one static frame and stops.
  - Use `IntersectionObserver` on `#spiral-canvas` to pause `requestAnimationFrame` when the hero scrolls off-screen.
  - Listen to `document.visibilitychange` to pause when the tab is hidden.

**Commit:** `phase 2: reduced-motion support and spiral visibility gating`

---

## Phase 3 — Focus states & interactive accessibility

**Problem:** No `:focus-visible` outlines existed; decorative SVGs were keyboard-focusable; logo linked to `#` (dead link).

**Fix:**
- Added `:focus-visible` rules to `style.css`:
  - Brand-red (`var(--primary)`) ring for standard links, `.btn`, `.nav-links a`, `.footer-link`.
  - White ring for `.btn-ghost`, `.btn-white`, `.contact-chips a` (dark/red backgrounds where red ring would be invisible).
- Added `aria-hidden="true" focusable="false"` to all 6 addon-icon SVGs in `index.html`.
- Changed logo `href="#"` to `href="index.html"`.

**Commit:** `phase 3: focus-visible rings, SVG aria-hidden, fix logo href`

---

## Phase 4 — Fix broken & dead CSS in utilities.css

**Problem:** `utilities.css` contained numerous rules referencing undefined CSS custom properties (`--primary-color`, `--secondary-color`, `--dark-color`, `--light-color`, `--success-color`, `--error-color`). These vars are never defined anywhere, causing silent failures. The `.btn` block also duplicated rules already defined in `style.css`.

**Removed:**
- `.btn`, `.btn-outline`, `.btn:hover` (duplicate + broken var)
- All `.bg-*` / colored `.btn-*` rules (undefined vars, unused by either HTML file)
- All `.text-*` color utilities (undefined vars)
- `.lead`, `.sm`, `.md`, `.lg`, `.xl`, `.text-center` type-size helpers (unused)
- All `.alert*` rules (undefined vars, unused)

**Kept:**
- `.container`, `.card`, `.flex`, `.grid`, `.grid-3` — used by `original.html` and/or `index.html`
- All spacing utilities (`.my-*`, `.mx-*`, `.m-*`, `.py-*`, `.px-*`, `.p-*`) — no undefined vars

**Verification:** `grep -n "primary-color\|secondary-color\|dark-color\|light-color\|success-color\|error-color" utilities.css` returns nothing.

**Commit:** `phase 4: remove dead/broken CSS from utilities.css`

---

## Phase 5 — SEO + share metadata + favicon + og-image

**Problem:** The page had no meta description, no Open Graph/Twitter card tags, no favicon, and a generic title.

**Changes:**

**`favicon.svg`** — Created: dark rounded rectangle with red "CN" text. SVG favicon works in all modern browsers.

**`favicon-32.png`** — Generated via Python script (`/tmp/gen_images.py`): 32×32 pixel favicon with dark background, red accent bars, and stylized "N" mark. Pure Python (no Pillow dependency) using raw PNG byte construction.

**`og-image.png`** — Generated via same script: 1200×630 Open Graph image with navy-to-dark diagonal gradient, red accent bar, red glow, and subtle Fibonacci arc decorations in the top-right quadrant.

**`index.html` head** updated with:
- `<meta name="description">` — concise 155-char description
- `<meta name="theme-color">` — brand red for mobile browser chrome
- Full Open Graph block (`og:type`, `og:title`, `og:description`, `og:image`, `og:url`)
- Twitter card block (`twitter:card`, `twitter:title`, `twitter:description`, `twitter:image`)
- Favicon links (SVG primary, PNG fallback)
- Updated `<title>` to "Create Networks — Web Design & Digital Services"
- JS class snippet remains first child of `<head>`

**`original.html`** updated with:
- `<meta name="robots" content="noindex">` — keeps the legacy page out of search indices
- Favicon links (matching index.html)
- Removed `<meta http-equiv="X-UA-Compatible" content="ie=edge">` (obsolete IE directive)

**Commit:** `phase 5: SEO meta, OG/Twitter tags, favicon, og-image`

---

## Phase 6 — Tappable contact chips + contrast nudge

**Problem 1:** Phone and email chips in the CTA section were plain `<span>` tags — not tappable on mobile, not activatable by assistive technology.

**Fix:** Replaced the phone and email `<span>` chips with `<a href="tel:…">` and `<a href="mailto:…">` respectively. Added `.contact-chips a` rule to `style.css` to give `<a>` tags the same pill styling as `<span>` chips (inheriting color, removing underline).

**Problem 2:** `.section-sub` used `color: var(--gray)` (`#6b7280`) on a white background, which fails WCAG AA (contrast ratio ~4.0:1 against white; minimum is 4.5:1 for normal text).

**Fix:** Changed `.section-sub` to `color: #4b5563` directly — a slightly darker gray that passes WCAG AA (~5.9:1 on white). The global `--gray` variable was left unchanged since it is used in other contexts (dark backgrounds, small/bold text) where the original value is acceptable.

**Commit:** `phase 6: tappable contact chips, section-sub contrast nudge`

---

## Phase 7 — README, LICENSE, BUILD_LOG

- **`README.md`** — Updated from placeholder `# portfolio` to full project README with stack overview, local dev instructions, deployment note, and author.
- **`LICENSE`** — Added MIT License (2026, Fernando Alvarez).
- **`BUILD_LOG.md`** — This file. Documents all phases, decisions, and rationale.

**Commit:** `phase 7: README, LICENSE, BUILD_LOG`

---

## Summary

| Phase | File(s) Changed | Category |
|-------|----------------|----------|
| 1 | `index.html`, `style.css` | Progressive enhancement |
| 2 | `style.css`, `index.html` | Accessibility / performance |
| 3 | `style.css`, `index.html` | Accessibility |
| 4 | `utilities.css` | CSS hygiene |
| 5 | `index.html`, `original.html`, `favicon.svg`, `favicon-32.png`, `og-image.png` | SEO / metadata |
| 6 | `index.html`, `style.css` | UX / accessibility / contrast |
| 7 | `README.md`, `LICENSE`, `BUILD_LOG.md` | Documentation |
