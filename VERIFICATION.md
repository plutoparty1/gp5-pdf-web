# Verification and limits

The first release is checked with genuine GP5 bytes and actual generated PDF/ZIP files. Mock renderers are used only for exporter boundary unit tests.

## Repeatable checks

```sh
npm ci
npm run check
npm test
npx playwright install --with-deps chromium webkit firefox
npm run test:browser
npm run build
npm run package
```

The browser suite serves the production build unchanged under `/gp5-qa/`. `GP5_QA_URL` overrides that URL for hosted verification. Both Pages and tagged release workflows require these checks. Test fixtures are small synthetic scores with distinct guitar, bass and drum notation.

## First-release evidence

- Node regressions: 26 passing tests covering metadata decoding, selected original indices, safe ZIP names, cancellation and partial output, lazy-module cancellation/retry, PDF stream errors/close/abort, pagination and header wrapping.
- Browser coverage: 54 passing scenarios across Chromium, Playwright WebKit and Firefox (18 per engine). The suite covers real file chooser/input, CP949 extended `뷁`, UTF-8, default/all/subset/empty/keyboard selection, real ZIP downloads, repeat export and source preservation, cancellation before assets arrive and after one completed PDF, malformed GP5 recovery, font failure/retry, same-origin GET-only asset requests, and 320/390/1440px UI states.
- Independent PDF inspection: generated guitar/bass/drum PDFs contain the correct title, selected instrument and track-specific notation; fonts are embedded, PDF pages are A4, text has no replacement character, notation remains vector content, and selected outputs exclude the other tracks. PDF pages are also rasterized for visual inspection.
- Long-score inspection: a 224-bar score produces 27 pages. Vector bounds remain inside the A4 content margins after SVG dimensions are explicitly matched to PDF coordinates.
- Security review: no score upload or external runtime asset requests; CSP, text-only metadata UI, SVG allowlist, safe archive names, pinned dependency integrity and distributed notices checked. `npm audit` reports zero advisories at release preparation.

Playwright WebKit on Windows/Linux is not a test of the macOS Safari application. Physical Mac/Safari and mobile devices were not available. This release does not promise every old browser, GP5 effect, or damaged/mixed-encoding file renders identically to Guitar Pro.

## Runtime audit

| Hypothesis | Observation and correction |
| --- | --- |
| Node and browser CP949 decoding could differ | Node ICU rejected a valid extended syllable. Native Chromium, WebKit and Firefox decode it correctly; the regression runs in all three browsers. |
| SVG CSS pixels could be scaled twice in PDF output | Initial long-score vectors exceeded A4 margins. Explicit SVG dimensions fixed the scale; all 27 pages were checked independently. |
| Repository-path hosting or deferred assets could fail after local success | The unchanged relative-base build runs beneath a repository-style prefix; asset requests and downloads are exercised. Stalled PDF-module cancellation and subsequent retry are now tested. |
| A PDF stream failure could leave the UI busy | A failing stream reproduced missing error/close/abort settlement. The collector now settles and cleans up on each terminal path, covered by regressions. |

## Resource limits

| Boundary | Limit |
| --- | --- |
| Input | GP5 5.00/5.10, at most 64 MiB |
| Score length | 2,000 bars |
| Rendered SVG | 1,000 parts and 32 MiB |
| PDF per track | 300 pages and 100 MiB |
| Completed PDF data in ZIP | 128 MiB |

These caps reduce excessive browser memory use; they are not a guarantee that every device can process a score at the limit. Score layout and ZIP assembly include synchronous work. Cancellation takes effect between cooperative stages and can be delayed during that work. On partial failure/cancellation, only already completed PDFs are offered, and only when a ZIP Blob was successfully created.

Downloads follow browser settings. The application does not overwrite the input GP5 and cannot universally save automatically to its original folder. An internet connection is needed to fetch the site and its local assets initially.
