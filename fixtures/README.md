# React accessibility fixtures

A single-page React app containing 22 deliberately broken components, each one a
failure mode that is specific to React rather than to HTML in general: focus lost
during reconciliation, conditional renders with no live region, portals without
dialog semantics, client-side route changes that are never announced, and so on.

It exists to give the checker a target with known-good ground truth.

## Running

No build step. React, ReactDOM and Babel are loaded from a CDN.

```sh
cd examples/react-a11y-fixtures
python3 -m http.server 4321
```

Then open <http://localhost:4321/>.

## Layout

Each case lives on its own hash route, `#/<case-id>`, and renders one broken
pattern together with a banner naming the React cause and the findings a checker
is expected to report.

## Ground truth

The page exposes the full case list on `window.__A11Y_FIXTURES__` as an array of
`{ id, title, react, expected }`, so a test can assert against it instead of
relying on a human reading the screen:

```js
const cases = await page.evaluate(() => window.__A11Y_FIXTURES__)
for (const c of cases) {
  await page.goto(`http://localhost:4321/#/${c.id}`)
  const found = await runChecker(page)
  expect(found).toEqual(expect.arrayContaining(c.expected))
}
```

## Cases

| # | Case | React cause |
|---|------|-------------|
| 01 | `div-button` | Click handler on a non-interactive element |
| 02 | `modal-no-trap` | Conditional render with no focus management |
| 03 | `portal-no-dialog-role` | `createPortal` target outside the app root |
| 04 | `route-no-announce` | Client-side view swap, no title or live region |
| 05 | `error-no-live-region` | `&&` conditional render of validation text |
| 06 | `field-not-described` | Error state never wired to `aria-describedby` |
| 07 | `index-key-focus-loss` | Array index as key; reconciliation reuses DOM nodes |
| 08 | `autofocus-steal` | `autoFocus` deep in the tree |
| 09 | `placeholder-as-label` | Controlled input with no label |
| 10 | `fake-select` | Stateful dropdown built from divs |
| 11 | `fake-tabs` | Active index in state with no ARIA mirror |
| 12 | `fake-accordion` | Boolean state drives the render only |
| 13 | `toast-no-status` | Timed conditional render |
| 14 | `spinner-no-status` | Ternary between spinner and data |
| 15 | `icon-button-no-name` | Inline SVG child, no label prop |
| 16 | `load-more-silent` | State growth with no announcement |
| 17 | `dangerous-markup` | `dangerouslySetInnerHTML` bypasses JSX |
| 18 | `nested-interactive` | Composition nests two interactive elements |
| 19 | `heading-order-by-prop` | Heading level chosen as a style prop |
| 20 | `color-only-state` | Selected index styled but never exposed |
| 21 | `focus-lost-on-unmount` | Filtering state removes the focused node |
| 22 | `no-landmarks` | Everything rendered as a `div` |
