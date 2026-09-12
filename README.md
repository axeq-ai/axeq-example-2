# axeq-example-2

Two React targets for developing and regression-testing the accessibility checker.
Neither has a build step — React, ReactDOM and Babel come from a CDN, so there is
nothing to install and no dependency to keep current.

```sh
python3 -m http.server 4321
```

- <http://localhost:4321/fixtures/> — 22 isolated, labelled cases
- <http://localhost:4321/storefront/> — a realistic ecommerce landing page

## Why two

The fixtures answer "does the checker detect rule X", one failure per route, with
an explicit expected result. The storefront answers the harder question: does it
still find them inside a page that looks like something a team would actually
ship, where violations sit next to correct markup and compete for attention.

The well-known broken demos — the W3C Before/After Demo, Deque's Mars Commuter —
are vanilla HTML. Neither exercises the failures that only appear once a framework
is managing the DOM, which is the class this checker exists to catch.

## `fixtures/`

Twenty-two cases, each on its own hash route `#/<case-id>`, each rendering one
broken pattern with a banner naming the React cause and the expected findings.
The full list is exposed on `window.__A11Y_FIXTURES__` as
`{ id, title, react, expected }`.

```js
const cases = await page.evaluate(() => window.__A11Y_FIXTURES__)
for (const c of cases) {
  await page.goto(`http://localhost:4321/fixtures/#/${c.id}`)
  expect(await runChecker(page)).toEqual(expect.arrayContaining(c.expected))
}
```

See `fixtures/README.md` for the case table.

## `storefront/`

"Fernweh", an outdoor clothing store: promo bar, auto-advancing hero, filterable
grid of six products, editorial split, newsletter signup, footer, and a slide-out
cart with working quantity controls.

It is built the way a competent team under deadline builds one — which is why the
problems are the ones that actually reach production rather than textbook
examples:

| Area | What is wrong |
|------|---------------|
| Page | No landmarks, no skip link, `h1` sits inside the rotating hero, `h3` before `h5` |
| Header | Icon-only search / account / cart buttons with no accessible name; nav built from `div`s; cart count changes silently |
| Hero | Advances every 4s with no pause control, slide changes are never announced, dots are unnamed `div`s |
| Filters | Selected chip is indicated by colour alone, no `aria-pressed`, result count changes without announcement |
| Product card | Whole tile is a `div` with `onClick`, with swatches and an add button nested inside it; rating rendered as `★` glyphs with the value never exposed; sale price has no programmatic relationship to the struck-through original |
| Cart drawer | Plain `div` — no `role="dialog"`, no focus trap, no Escape, focus never returns to the trigger; quantity and remove controls are unnamed |
| Toast | "Added to your bag" appears with no live region |
| Newsletter | Placeholder used as a label, error text not tied to the field, neither error nor success announced |
| Footer | Social links wrap bare SVG with no accessible name; muted text on dark ground fails contrast |

Expected findings are grouped by area on `window.__A11Y_EXPECTED__`, and every
affected element carries a `data-a11y` attribute naming its own failures, so a
diff between what the checker reports and what is annotated is mechanical.

## Rule ids

The ids in both files are descriptive (`drawer-no-dialog`, `swatch-no-name`)
rather than axe-core ids. Worth mapping them onto whatever taxonomy the checker
settles on.
