# Urban Craft Studio — website

Portfolio site for an architecture studio in Pune (principal: Ar. Yusuf Shaikh), built as a client preview.
Theme: "Blueprint to Built". A line becomes a drawing, the drawing becomes a space, the space becomes a place.

## Files

| File | What it is |
|---|---|
| `index.html` | Studio homepage. One self-contained file (HTML + CSS + JS inline). |
| `architect.html` | The principal's personal "monograph" page. Same design system, own content. |
| `deploy/` | Generated copy of the two pages that gets uploaded to Netlify. Git-ignored. Never edit it. |
| `.netlify/state.json` | Links this folder to the Netlify site. Git-ignored. |
| `.claude/launch.json` | Local preview server (`static-preview`, `python -m http.server 8732`). Git-ignored. |

No build step, bundler or package.json. Libraries come from CDNs, pinned to exact versions:
GSAP 3.12.5 + ScrollTrigger (cdnjs), Lenis 1.0.42 (jsDelivr), Three.js r150 (cdnjs, UMD `three.min.js`).
Fonts are loaded from Google Fonts: Fraunces (display), Inter 400/500 (body), JetBrains Mono (technical labels).
Keep the font URL **identical** on both pages.

## Content lives in data objects

- `index.html` → `const DATA = {…}` at the top of the main `<script>`.
- `architect.html` → `const ARCHITECT = {…}` plus `const SHOW_SAMPLE_TESTIMONIALS`.
- Every placeholder entry carries a `// DUMMY` comment; unknown facts are written as `[… — TODO]`.
- Images use the helper `IMG(id, w)` → `https://images.unsplash.com/photo-<id>?w=<w>&q=80&auto=format&fit=crop`
  (800 for thumbnails, 1600 for large views). Always `loading="lazy"` + meaningful `alt`.
- The markup renders from these objects in `render()`. Change content in the data, not in the HTML.

### Content rules from the client brief (do not break)
- **No** awards, client names or testimonials on `index.html`.
- Never use a stock photo of a person's face for Yusuf. The placeholder is the "hands drafting" photo, `1503387762-592deb58ef4e`, marked with a TODO for his real portrait from @iamyusufshaikh.
- The CoA (Council of Architecture) registration **number must never appear** on the site, only "Registered Architect".
- Testimonials on `architect.html` are `sample: true` placeholders and show a visible
  "SAMPLE — replace with a real client review" tag. Only real reviews collected with the client's permission may go live.
  Set `SHOW_SAMPLE_TESTIMONIALS = false` before launch. The section hides itself if no real entries remain.

## Design system (identical `:root` block on both pages; keep them in sync)

- Core: `--paper #EDEAE4`, `--paper-2 #E4E0D8`, `--ink #1A1A1A`, `--ink-2 #242320`, `--terracotta #C2643C`, `--brass #B08D57`.
- Alpha steps: `--paper-aXX` / `--ink-aXX` (e.g. `--paper-a14`, `--ink-a18`). **Don't write raw `rgba(237,234,228,…)` or `rgba(26,26,26,…)`.** Add a token instead.
- Brass plaque shades: `--brass-light/-lighter/-dark/-darker/-text/-muted`.
- Accessibility rule for terracotta:
  - `--terracotta` (#C2643C): large display text, fills, lines, buttons, the cursor bubble only.
  - `--terracotta-text` (#9E4925, 5.09:1 on paper): any **small** terracotta text on light backgrounds.
  - `--terracotta-on-dark` (#D9784E, 5.59:1 on ink): small terracotta text on dark backgrounds.
  - Overrides are collected in the "WCAG AA" block near the bottom of each `<style>`.
- Type scale: `--fs-h1`, `--fs-h2` (section titles), `--fs-h3` (card/sub-headings, weight 500, line-height 1.2),
  `--fs-body`, `--fs-small`, `--fs-mono`. `h1–h3` use `text-wrap: balance`. Quotes are Fraunces 300 italic.
- The site is deliberately a single light "paper" theme with dark ink bands. It has no dark mode.

## Behaviour that spans both pages

- **Cursor**: crosshair with live X/Y. Any dark area must carry `data-cursor-theme="light"` so the cursor turns
  light there (`.cursor-layer.on-dark`). Hover labels use `data-cursor="view|drag|read"` (`architect.html`) or
  `data-cursor="view"` (`index.html`). New dark sections need the attribute.
- **Page transition**: links with `data-transition` play a terracotta wipe with "AR. YUSUF SHAIKH", set
  `sessionStorage['ucs_wipe']` (always inside try/catch) and navigate.
  - Arriving at `architect.html` with the flag plays the reverse wipe instead of its preloader.
  - Arriving at `index.html` **always runs the full floor-plan preloader**, then `jumpToHash()` scrolls to `#works` etc.
  - Both pages have a `pageshow` (bfcache) handler that removes a stuck wipe panel.
- **Preloader** is visible from first paint by plain CSS (`display:block`). Don't gate it behind `html.js`.
- `prefers-reduced-motion`: no 3D, pinning or parallax; fades only. Mobile (≤860–900px) swaps pinned/3D sections
  for stacked layouts (works gallery, journey timeline, gallery drum → swipe deck).

## Known gotchas (each one has already bitten us)

- GSAP reads a CSS `transform: translateY(101%)` as **pixels** (`y`), separate from `yPercent`. When animating the
  wipe panel with `yPercent`, always `gsap.set(panel, { y: 0, yPercent: … })` first.
- `setPointerCapture` on a drag stage retargets the following `click` to the stage. Hit-test with
  `document.elementFromPoint(e.clientX, e.clientY)` to find the clicked item.
- `.stat span` style rules must be scoped `.stat > span`. Otherwise they also hit the number inside `<b>`.
- Set the pinned section's layout (`height:100svh`, flex) **before** creating its ScrollTrigger, which measures it.
- Use `overflow-x: clip` (with `hidden` fallback) on `body`/heroes so iOS never shows sideways scroll.

## Preview & test

- Start the `static-preview` server from `.claude/launch.json` and open `http://localhost:8732/` in the browser pane.
- The pane **caches aggressively**. Add a query string (`?v=2`) or `location.reload()` after edits. If the
  served file looks stale, check with `fetch(url, {cache:'no-store'})`.
- Re-verify at 1440, 768 and 375px: zero console errors and `document.documentElement.scrollWidth - innerWidth <= 0`.
- The preview pane swallows a physical Esc key. Test Esc handlers with a dispatched `keydown`.

## Deploy (Netlify)

Site: `urban-craft-studio` → https://urban-craft-studio.netlify.app (folder already linked via `.netlify/state.json`).

```bash
cp index.html architect.html deploy/
npx netlify-cli deploy --dir deploy --prod --message "<what changed>"
```

Only the two pages go up; never deploy the project root. The Netlify team has "Team login" visitor protection on by
default, so new sites are not public until Visitor access is changed in the Netlify dashboard.

## Git

Remote: https://github.com/sufisclaude-sketch/urban-craft-studio (`main`).
Pushing needs Git Credential Manager's browser sign-in, which can't prompt from a background shell. Run
`git push` in the Terminal panel (an interactive terminal) instead.
