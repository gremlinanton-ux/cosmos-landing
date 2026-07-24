# AVENORA landing — handoff

Single-page marketing site for an AI property-intelligence product.
Target market is Poland; UI ships in Polish, English and Russian.

## Where things are

| | |
|---|---|
| Source | `cosmos-landing/index.html` — **everything is in this one file** (~2470 lines) |
| Repo | https://github.com/gremlinanton-ux/cosmos-landing (branch `master`) |
| Spec | `../AVENORA_Landing_Update_PRD.md` — section structure and copy come from here |
| Design ref | `../3.png` — the "Built on trusted data" layout that section 3 follows |
| Dev server | `.claude/launch.json` → `python -m http.server 8467 --directory cosmos-landing` |

Open with `preview_start {name: "cosmos-landing"}`, then `http://localhost:8467`.

> `python -m http.server` sends caching headers and the browser will happily serve a
> stale copy after an edit. Append a cache-buster when verifying: `http://localhost:8467/?v=12`.

## File map

One file, ordered: `<style>` → markup → four `<script>` blocks.

| Lines | What |
|---|---|
| 10–1043 | All CSS. Section banners (`/* ---------- Navigation ---------- */`) mark the blocks. |
| 1049 | `<nav>` — **direct child of `<body>`**, not inside `.ui` (see gotcha 2) |
| 1120–1252 | `<main>` — the seven content sections |
| 1254 | `<footer>` |
| 1287 | Auth modal markup |
| 1368–1655 | i18n dictionary (`window.T`) + `window.LANG` |
| 1657–1737 | Hero: search field, chips↔status animation, paste/clear button |
| 1739–1890 | Section content generators, scroll reveal, timeline rail geometry |
| 1891–1925 | Language switching |
| 1926–2211 | Auth modal: open/close, validation, confirmation screen |
| 2219–2464 | Three.js scene (the 3D logo background) |

Sections and their anchors, in page order:
`#report` · `#data` · `#how` · `#neighborhood` · `#workspace` · `#pricing` · `#faq`

## Design system — do not drift from this

- **Black and white only.** No gradients, no colour, no illustrations. The single
  exception is error red (`rgba(255,116,116,…)`), because an error is a functional signal.
- Two glass tokens: `.glass` (blur 20px) and `.glass-strong` (blur 44px). Content cards
  use `.card`, which frosts harder at 34px.
- Type is variable Inter with the optical-size axis on. Headings are weight 200 with a
  500 accent line; tracking tightens as size grows.
- Motion vocabulary: **opacity, blur, small translate**. No scaling, no bouncing.
  Transitions 250–400ms ease-out.
- The hero headline's second line uses a brushed-silver gradient via `background-clip: text`;
  its glow is `filter: drop-shadow`, not `text-shadow`, so it works over that gradient.

**The hero and the 3D background are settled.** Several rounds went into the logo geometry,
material and lighting. Don't restyle them without being asked.

## i18n

`window.T = { pl, en, ru }`. Polish is the default and the fallback; the choice persists in
`localStorage['avenora.lang']`.

- Static markup: add `data-i18n="key"` (or `data-i18n-placeholder`, `data-i18n-label`).
- Generated lists (steps, sources, FAQ, …): add the array to all three languages; the
  generators read `T[LANG]`.
- Switching dispatches a `langchange` event on `document`. Anything that caches DOM
  references or derived geometry must re-subscribe there — the section generators, the
  hero status chips and the rail all do.

Adding a string means touching **all three** dictionaries. A missing key renders nothing.

## Gotchas that will bite you

1. **`main` has `pointer-events: none`.** The 3D logo must stay draggable in the gutters
   between blocks. The property inherits, so only leaves take events back
   (`.card`, `.chip`, `.ghost-btn`, `.sec-head`, `a`, `button`, `input`). New interactive
   markup outside those selectors will be dead to the mouse.

2. **The nav sits at the top level on purpose.** It used to live inside `.ui`, whose
   `z-index: 5` created a stacking context and trapped the nav's own `z-index: 100`, so
   sections painted over it. Keep it a direct child of `<body>`.

3. **Timeline rail.** The line is masked so it only exists in the gaps between cards, and
   each card flashes as the pulse passes under it. Mask stops and per-card
   `--flash` delays are measured from live geometry and recomputed on resize and on
   `langchange` (card widths change with the text). The sweep animation **must stay
   `linear`** — with easing, pulse position stops mapping linearly to time and the flashes
   drift out of sync.

4. **`requestAnimationFrame` is throttled hard in a hidden tab** (measured: 2 frames in
   300ms) and the Three.js loop stops entirely. Use the double-rAF trick only to kick off
   CSS transitions. Never gate correctness or focus on it — the auth modal sets focus
   synchronously for exactly this reason.

5. **Watch for class-name collisions.** A pricing paragraph was marked `.lead`, which was
   already the search field's circular paste button; it inherited `display:flex` and a
   34px width and collapsed into a one-word column. Namespace new classes.

6. **`<i>` is inline** — `height` does nothing without `display: block`. This silently
   zeroed the dashboard's neighbourhood meter bars.

7. **Blur validation can eat a click.** A click lands on release; if a blur handler shows
   an error and reflows the panel in between, the button moves out from under the cursor.
   The modal holds validation while a control is pressed (`pressing` flag). Keep that in
   mind before adding more blur-driven UI changes.

8. **`offsetParent` is `undefined` on SVG elements** — useless for testing SVG visibility.
   Use `getBoundingClientRect()`.

## Verifying in the preview pane

The pane is unreliable for visual checks; these workarounds cost real time to rediscover:

- **Screenshots go stale.** The pane only repaints changed regions, so static content can
  screenshot as black. Force a repaint first:
  `document.body.style.opacity='0.999'; setTimeout(()=>document.body.style.opacity='',60)`
- **Screenshot coordinates do not map reliably to page coordinates.** A drag aimed at the
  card row once landed outside the viewport. Prefer `elementFromPoint` and direct event
  dispatch over coordinate-based clicking.
- **Animations and transitions freeze while the pane is hidden** (`visibilityState:
  'hidden'`), so computed styles read the start value instead of interpolating. To check
  that a rule *matches*, inject `transition: none !important`, then measure.

## Not built yet

- **No backend.** The auth form intercepts submit; the confirmation screen stands in for a
  real sign-up call. Google and Apple buttons are not wired to any provider.
- `Analyze` in the hero navigates to `/report/loading`, which does not exist.
- Section 5's interactive map is a placeholder panel ("Mapa interaktywna / W przygotowaniu").
  The PRD notes the map component should be reusable in the future dashboard.
- Footer links, `Forgot?` and section-6 workspace features are copy only.
- Email validation accepts any well-formed address (`REQUIRE_KNOWN_DOMAIN = false`). The
  known-provider list still runs, but only to suggest fixes for typos like `gmial.com`.

## Open question

The PRD spells the brand **AVENORA**; earlier drafts of the site used AVENOR. The site now
says AVENORA everywhere, and the nav shows `AVENORA.PL`. If a real domain differs, that is
one find-and-replace plus the `meta.title` keys in all three dictionaries.
