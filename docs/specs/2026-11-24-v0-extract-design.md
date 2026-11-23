# V0 "extract": design

**Status:** draft, awaiting the maintainer's validation
**Roadmap stage:** V0, see the [README](../../README.md#roadmap)

## 1. Goal

Turn the visitor side of the WordPress plugin [fixthisplz-wordpress](https://github.com/fixthisplz/fixthisplz-wordpress) into two packages that work on any website:

- **`@fixthisplz/schema`** defines what a report is and how it is sent: the *submission protocol*.
- **`@fixthisplz/widget`** is the reporting UI (button, element picker, form, screenshot, annotation, automatic context). It is a framework-free web component isolated in Shadow DOM.

The WordPress plugin then becomes **the first consumer** of the widget, with no visible regression. That proves the extraction on a real backend before any hub exists.

### Done means

1. A developer can add the widget to any page in a few lines and receive reports, either on an HTTP endpoint that speaks the protocol or in their own `onSubmit` callback.
2. Plugin 1.3.0 uses the published widget, and every 1.2.0 visitor flow still works on the test bench.
3. Both packages are published on npm as `0.1.0` with provenance, built and tested by CI.

## 2. Scope

### In

- The monorepo tooling (§4).
- `@fixthisplz/schema`: JSON Schema, TypeScript types, limits, protocol (§5).
- `@fixthisplz/widget`: port of the plugin's visitor-side code to TypeScript, Shadow DOM, public API (§6).
- `examples/basic`: a plain HTML page that uses the widget with `onSubmit`, also used by the end-to-end tests.
- Plugin 1.3.0 switches to the widget (§8). The changes are made in the plugin's own repository.
- The issue and PR templates still missing from this repository (community health is at 87%).

### Out (later stages)

- The hub, project keys, `data-project` auto-init from a `<script>` tag (V1).
- `FixThisPlz.identify()` and the reporter magic link (V1/V2).
- React and Vue adapters (V3).
- Custom form fields, PNG screenshots, scroll-position capture: not in the plugin today, so not in V0.

## 3. Starting point (facts from the plugin, 1.2.0)

- **About 157 kB of plain ES modules, with no build step**, plus `modern-screenshot` 4.7.0 (55 kB, MIT, vendored).
- Loading happens in layers:
  - a classic `loader.js` (6 kB) installs the console and network buffers and the button;
  - the picker and form are imported on the first click;
  - the annotation editor is imported on the first "Annotate" click;
  - the DOM-reconstruction capture is imported only when needed.
- **WordPress-specific parts are few:**
  - the `window.fixThisPlzConfig` global;
  - the `X-WP-Nonce` header;
  - the WordPress REST error shape;
  - the dashicons button icon;
  - the `wp_page` and `plugin_version` context keys;
  - 75 labels translated in PHP and passed in the config.

  Everything else is plain DOM code.
- **No Shadow DOM yet.** Isolation relies on `all: initial`, ID-scoped rules and `fixthisplz-` prefixes.
- **179 JS tests run with `node --test` and hand-made DOM fakes.** There are no unit tests for the dialog, the picker, the captcha or the annotation editor shell.

## 4. Repository and tooling

```
fixthisplz/
├── packages/
│   ├── schema/          # @fixthisplz/schema
│   └── widget/          # @fixthisplz/widget
├── examples/
│   └── basic/           # index.html + a deliberately hostile stylesheet
├── docs/specs/
├── .changeset/
└── .github/             # CI, release, issue & PR templates
```

| Concern | Choice | Why |
|---|---|---|
| Package manager | **pnpm** workspaces | Strict dependencies; the de facto standard for TypeScript monorepos |
| Runtime | Node **22 LTS** minimum; CI on 22 and 24 | The current LTS lines |
| Language | **TypeScript** with `strict` and `noUncheckedIndexedAccess` | Typed public API; contributors get autocompletion |
| Build | **tsup** (esbuild): ESM, code splitting, `.d.ts` | Keeps today's lazy layers as separate chunks with no configuration work |
| Tests | **Vitest**; **Playwright** for end-to-end tests | Vitest runs the ported tests nearly unchanged; Playwright drives real Chromium and Firefox |
| Lint / format | **Biome** | One fast tool instead of ESLint + Prettier |
| Versioning | **Changesets** | Per-package versions and CHANGELOGs; contributors add a changeset in their PR |
| Size | **size-limit** in CI | Budgets enforced, see §6.6 |
| Publishing | GitHub Actions, **npm trusted publishing (OIDC)**, `--provenance` | No long-lived npm token to leak |

## 5. `@fixthisplz/schema`

### 5.1 Contents

- `report.schema.json` (JSON Schema 2020-12) is **the single source of truth**. TypeScript types are generated from it at build time, so they cannot drift.
- `limits` are exported as constants (5,000 characters of description, 64 KiB of context, 5 MiB of screenshot…). Both the widget and servers read them.
- Error codes are exported as constants (§5.3).
- There is no runtime validator in V0. The plugin validates in PHP, and the hub (V1) will pick a validator when it needs one.

### 5.2 Report context: version 1

The **wire format uses snake_case**, as today, so PHP and other backends read it naturally. Changes from 1.2.0:

| Key | Change |
|---|---|
| `schema_version` | **New.** Always `1`. Lets a backend support several versions later. |
| `widget_version` | **Replaces `plugin_version`.** The widget, not the host, produces the report. |
| `metadata` | **New.** Optional flat map of host-supplied strings (at most 20 keys; 64 characters per key; 500 per value), for things like app version or tenant. |
| `wp_page` | **Removed** as a top-level key. The plugin sends it as `metadata.wp_page`. |

These keys are unchanged, with the same limits and redaction: `url`, `title`, `locale`, `timezone`, `user_agent`, `viewport`, `element`, `console`, `network`, `capture`. Redaction still happens in the browser before sending, and again on the server.

### 5.3 Submission protocol, version 1

The protocol is the one the plugin already implements, written down so any backend can implement it:

- **Request:** `POST <endpoint>`, `multipart/form-data`, with these fields:
  - `description`;
  - `context` (JSON of §5.2);
  - `reporter_email` (optional);
  - `website` (honeypot, must be empty);
  - `captcha_token` (optional);
  - `screenshot` (optional JPEG, at most 5 MiB).
- **Success:** `201` with `{ "id": "…", "number": "…" }`. `number` is what the reporter sees, for example `#42`.
- **Failure:** an HTTP status plus `{ "code": "…", "message": "…" }`.

| Status | Code | Meaning |
|---|---|---|
| 403 | `fixthisplz_not_allowed` | The reporter may not report |
| 403 | `fixthisplz_captcha_failed` | The captcha check failed |
| 404 | `fixthisplz_disabled` | Reporting is switched off |
| 413 | (any) | The payload is too large |
| 422 | (any) | Invalid field. **`message` is shown to the reporter**, so it must be human-readable and translated. |
| 429 | (any) | Rate limited |
| 503 | (any) | The captcha provider is unavailable |

The codes keep their `fixthisplz_` prefix. It is already namespaced, and keeping it means the plugin does not have to change.

## 6. `@fixthisplz/widget`

### 6.1 Public API

```ts
import { FixThisPlz } from '@fixthisplz/widget';

FixThisPlz.init({
  // exactly one of:
  endpoint: 'https://example.com/api/reports',   // protocol v1 (§5.3)
  onSubmit: async (submission) => ({ number: '#42' }),

  headers: { 'X-WP-Nonce': '…' },     // endpoint mode only
  credentials: 'same-origin',         // endpoint mode only, default 'same-origin'
  locale: 'fr',                       // default: <html lang>, then navigator.language
  messages: { /* override any label */ },
  button: { position: 'bottom-right', label: 'Report a problem', icon: '<svg…>' } /* or false */,
  dialog: { title, instructions, privacyText, privacyUrl, askEmail, screenshot: true, captureMethod: 'tab', maskCapture: true, exportFormats: ['md'] },
  collect: { console: true, network: true },
  sensitiveSelectors: ['.secret'],     // in addition to the built-in rules and [data-ftp-mask]
  captcha: { provider: 'turnstile', siteKey: '…' },
  metadata: { appVersion: '2.3.1' },
});

FixThisPlz.open();      // start a report from your own button (when button: false)
FixThisPlz.destroy();   // remove the widget and restore console / fetch / XHR
```

- **`onSubmit` makes V0 useful on its own.** Without any backend, a developer receives the typed submission (the description, context and screenshot `Blob`) and can send it wherever they want.
- In endpoint mode, network capture **skips requests to `endpoint`**. This replaces today's `restUrl` check.
- `init()` validates its options and throws a clear `TypeError` when they are wrong, for example when both `endpoint` and `onSubmit` are given, or neither.

### 6.2 Shadow DOM

- The widget renders inside a `<fixthisplz-widget>` custom element with an **open** shadow root. It keeps the `data-fixthisplz-ui` attribute, so the picker and the screenshot still ignore the widget's own UI. Today's global `front.css` moves inside the shadow root.
- **Theming:**
  - CSS custom properties set on the host element cross the shadow boundary: `--ftp-accent`, `--ftp-button-bg`, `--ftp-button-fg`, `--ftp-button-size`, `--ftp-z-index`, and so on.
  - Parts the host may restyle are exposed with `::part()` (`button`, `dialog`).
- **Defaults come from the brand kit** (`brand/`): the accent is `#6825D5`, and the default button icon is `svg/icons/icon-widget-currentcolor.svg`, inlined so it follows the button's `color`.
- **Icon:** `button.icon` takes an SVG string or an image URL. For anything else there is a **light-DOM `icon` slot**. Slotted content keeps the page's styles, which is how the plugin keeps dashicons without loading their stylesheet into the shadow root.

#### Known pitfalls

Each of these needs a test:

- Inside a shadow root, `document.activeElement` is the host element. The focus trap must use `shadowRoot.activeElement`.
- Events from inside the shadow root are retargeted to the host. The picker's own-UI check must use `event.composedPath()`.
- The capture already walks into open shadow roots, so it must skip the widget's host element.

### 6.3 Loading layers

The current layers become chunks produced by the build:

| Chunk | Contents | Loaded |
|---|---|---|
| entry | `init`, console and network buffers, button | Immediately |
| report | picker, dialog, context, redaction, captcha | First open |
| annotate | annotation editor | First "Annotate" click |
| dom-capture | DOM reconstruction + `modern-screenshot` | Only when tab capture is unavailable or not chosen |

### 6.4 Translations

- The widget ships **English and French** catalogs and falls back to English for any missing key. This fixes today's empty labels when a key is missing.
- Hosts can override any label with `messages`. The plugin keeps passing its PHP-translated labels this way, so its translations on translate.wordpress.org keep working.

### 6.5 Builds

- **The `dist/` on npm is ESM and not minified.** Bundlers minify for their users, and the plugin can vendor readable code: wordpress.org requires human-readable code in plugins.
- **`dist/cdn/` is minified ESM** for loading from a CDN with `<script type="module">`.
- There is no IIFE or global build in V0. Every browser that supports the screenshot APIs also supports modules.

### 6.6 Size budgets

The README promised "< 30 kB gzipped" for the whole widget. With lazy chunks, the number that matters is the initial cost.

- **The entry chunk must stay under 10 kB gzipped.** That is what every page pays.
- The other chunks get budgets set to the size measured right after the port, plus 10%. After that, CI fails on any growth that was not decided on purpose.
- The README will be updated to say "under 10 kB gzipped on page load, the rest loads on demand".

### 6.7 Tests

- **Port all 179 tests to Vitest**, keeping their hand-made fakes. They pass before any refactoring starts, which locks in today's behaviour.
- **New unit tests** cover the untested parts: the dialog (validation, submit, error mapping), the picker, the captcha loader, `init` option validation, and the focus trap inside a shadow root.
- **End-to-end tests** run Playwright on `examples/basic` in Chromium and Firefox:
  - open the widget, pick an element, take a DOM capture, annotate, submit;
  - check the submission received by `onSubmit` against the schema;
  - check that the page's hostile stylesheet does not leak into the widget.
- Tab capture runs in Chromium with the fake-media flags.

## 7. Release

1. CI must be green: lint, type check, unit tests, end-to-end tests, build, size check.
2. A Changesets "version" PR is merged.
3. The release workflow publishes `@fixthisplz/schema@0.1.0` and `@fixthisplz/widget@0.1.0` with provenance and creates the GitHub releases.

**Publishing needs the maintainer's explicit go.**

## 8. Plugin 1.3.0: first consumer

These changes are made in the `fixthisplz-wordpress` repository. Its rules apply: no build step, WPCS, the test bench, and Chrome verification.

### 8.1 Vendoring and loading

- **Vendor the widget.** The readable `dist/` of `@fixthisplz/widget@0.1.0` goes in `assets/vendor/fixthisplz-widget/`, together with its MIT licence and version, just as `modern-screenshot` is vendored today. A small script in `bin/` refreshes it from npm.
- **Add a new `assets/js/adapter.js`** (about 1 kB, a script module). It reads `window.fixThisPlzConfig` and calls `FixThisPlz.init()`:
  - `endpoint` is `restUrl + 'reports'`;
  - the nonce goes in `headers`;
  - the plugin settings map to widget options;
  - `metadata.wp_page` is set;
  - the dashicon goes in the `icon` slot.
- **Update `Assets.php`** to register the widget entry and the adapter as script modules. The visibility rules, which run on the server, are unchanged.
- **Update `Button_Style.php`.** It keeps generating inline CSS, but the CSS now sets custom properties on the `fixthisplz-widget` host instead of styling `#fixthisplz-button`.

### 8.2 Removals

- **Delete** `loader.js`, `reporter.js`, `assets/js/lib/*`, `front.css`, the vendored `modern-screenshot` (now inside the widget), and the JS tests that moved to the widget.
- **Keep** the admin JavaScript and an adapter test.

### 8.3 Server

- **`Context_Validator.php`** accepts context v1 (§5.2).
- **The admin report view** reads `widget_version` and `metadata.wp_page`. It still reads `plugin_version` and `wp_page` for reports stored by 1.2.0: that is stored data, not dead code.
- **A PHP test** checks that the plugin's limits match `limits` in the vendored schema, so the two languages cannot drift apart.

### 8.4 Verification

- PHP tests, adapter test and Plugin Check all pass.
- Every 1.2.0 visitor flow is re-checked in Chrome on the bench:
  - button position and styles;
  - picker by mouse, touch and keyboard;
  - tab and DOM capture with masking;
  - annotation;
  - captcha;
  - honeypot;
  - rate-limit message;
  - logged-in and guest submissions;
  - `md`/`json`/`txt` export;
  - French labels.

## 9. Work order

Each step is reviewed before the next one.

1. Tooling scaffold, CI, issue and PR templates.
2. `@fixthisplz/schema`.
3. Port the widget to TypeScript **unchanged**, with the 179 tests ported and passing.
4. Remove the WordPress assumptions: `init()` API, endpoint or `onSubmit`, translations, `metadata`.
5. Shadow DOM and theming, with the new unit tests.
6. `examples/basic` and the end-to-end tests; measure the sizes and set the budgets.
7. Publish `0.1.0` (after the maintainer's go).
8. Plugin 1.3.0.

## 10. Decisions for the maintainer

1. **Where the plugin lives during V0.** *Recommended:* it stays in `fixthisplz-wordpress` and vendors a tagged widget release. It is under review on wordpress.org, and moving it into `adapters/wordpress` can wait until the hub exists and shared code justifies it.
2. **`onSubmit` callback mode in V0.** *Recommended:* yes. It is what makes the widget usable before the hub exists.
3. **Publish `0.1.0` on npm at the end of V0.** *Recommended:* yes. A `0.x` version signals "early" honestly, and real users give the first feedback.
