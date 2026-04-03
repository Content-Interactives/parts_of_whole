# Parts of a Whole

React activity for reading **fractions from food metaphors**: each round shows a randomized “topped” share (`numerator` / `denominator`) on one of several SVG-like DOM illustrations. Learners adjust steppers (numerator 1–12, denominator 2–12) and submit; correct answers trigger `canvas-confetti`, show brief success copy, then advance the food queue and draw a new fraction.

**Production:** [https://content-interactives.github.io/parts_of_whole/](https://content-interactives.github.io/parts_of_whole/)

---

## Stack

| Layer | Packages / tooling |
|--------|---------------------|
| UI | React 19 (`react`, `react-dom`) |
| Build | Vite 7, `@vitejs/plugin-react` |
| Styling | Tailwind utility classes; local `<style>` block in `PartsOfWhole.jsx` for `.animate-shake` (not `styled-jsx`; attribute `jsx` is inert in standard Vite React) |
| Feedback | `canvas-confetti` |
| Lint | ESLint 9 (flat config), React Hooks / Refresh plugins |
| Deploy | `gh-pages` → `dist` |

`package.json` sets `"type": "module"`.

---

## Build and base path

`vite.config.js` sets `base: '/parts_of_whole/'` for GitHub Pages.

| Script | Command |
|--------|---------|
| Development | `npm run dev` |
| Production bundle | `npm run build` → `dist/` |
| Preview `dist` | `npm run preview` |
| Deploy | `npm run deploy` (`predeploy` → `vite build`, then `gh-pages -d dist`) |

---

## Repository layout

| Path | Role |
|------|------|
| `index.html` | Shell; `#root` |
| `src/main.jsx` | `createRoot`, `StrictMode`, `index.css` |
| `src/App.jsx` | Renders `PartsOfWhole` |
| `src/components/PartsOfWhole.jsx` | Fraction RNG, food index, inputs, check, feedback state machine |
| `src/components/Foods.jsx` | `getFoodsArray()` (Fisher–Yates shuffle), food-specific presentational components |
| `src/components/ui/reused-ui/Container.jsx` | Chrome (`showSoundButton` without `onSound` unless you add it) |

---

## Application logic (`PartsOfWhole.jsx`)

- **Food queue:** On mount, `getFoodsArray()` returns a **shuffled** list of six food configs (Pizza, Brownie, Pancake, Pie, Chocolate Bar, Donut). `currentFoodIndex` selects the active entry; colors and prompt strings (`foodName`, `toppingName`) come from that entry.
- **Fraction generation:** `generateFraction` runs when `currentFoodType` changes (and via success path). Denominator: **Pizza** (and other non-brownie types using the `else` branch)—uniform **2..12**. **Brownie** only—uniform choice from **{2, 4, 6, 8, 10, 12}** so rectangular grid partitioning stays even. Numerator: uniform **1..denominator** (always a proper non-zero slice count).
- **User input:** `numeratorInput` clamped 1–12; `denominatorInput` clamped 2–12. Display colors mirror the active food’s `numeratorColor` / `denominatorColor`.
- **Check:** Strict equality `numeratorInput === numerator && denominatorInput === denominator`. Success: confetti, `feedbackState === 'showText'` (“Great Job!”), after 3 s reset inputs to 1/1, `progressToNextFood()`, and `generateFraction()`. Wrong: 500 ms `animate-shake` on the button.
- **Progress:** After the last food in the current shuffle, a **new** `getFoodsArray()` replaces the queue and index resets to 0.

---

## Food renderers (`Foods.jsx`)

Shared patterns:

- **Radial / wedge foods** (Pizza, Pancake, Pie, Donut): `360 / denominator` slice lines; first divider oriented with `baseCssOffsetDeg = 180`. Filled wedges are **contiguous from slice index 0**—toppings placed by polar geometry (`sin`/`cos` from top-of-clock). `useIsSmallScreen` scales layout on narrow viewports (~≤351 px).
- **Rectangular grid foods** (Brownie, ChocolateBar): `getGridDimensions(denominator)` maps even totals to fixed row/column layouts; numerator segments fill **row-major** from index 0. Brownie/chocolate use `useBrownieScaleFactor()` (breakpoints ~356 / 411 width) for whole-component scale. Sprinkles/nuts use a deterministic `seededRandom` so layouts are stable across re-renders.

Components are plain React + absolutely positioned `div`s (no canvas), with `role="img"` and `aria-label` on roots.

---

## Product integration

- **CK-12 Intent Response** — production / master: pending  
- **CK-12 Flexbooks** — book/lesson link: pending  

Upstream: [github.com/Content-Interactives/parts_of_whole](https://github.com/Content-Interactives/parts_of_whole).

---

## Educational alignment

Topic and Common Core citations are in [`Standards.md`](Standards.md) (e.g. 2.G.A.3, 3.NF.A.1).
