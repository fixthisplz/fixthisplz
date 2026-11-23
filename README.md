# Fix this, plz

**Let your users tell you what's broken.**
An open-source, self-hosted tool for human bug reporting: a drop-in widget for any website or app, an annotated screenshot, the technical context attached automatically, and a **lightweight built-in tracker** to triage and fix, with optional sync to the tools your team already uses.

> 🧭 **This is a vision document.** Nothing below exists yet beyond the original WordPress plugin.
> It describes where the project is headed so we can discuss it *before* writing code.
> Feedback is very welcome: open an issue or start a discussion.

---

## Why

Error trackers like Sentry are great at catching what **the machine** sees: exceptions, stack traces, failed requests.
But many real problems never throw an error:

- "The *Pay* button does nothing on my phone."
- "The price on this page is wrong."
- "I can't find where to change my address."

Those reports usually arrive as a vague email ("it doesn't work"), with no URL, no screenshot and no browser details, and then come three rounds of back-and-forth before anyone can reproduce the problem.

The tools that solve this today (Marker.io, BugHerd, Userback, Usersnap…) are good, but they are **closed-source SaaS**: your clients' screenshots, which often contain personal data, live on someone else's servers, and the cost grows with every site you manage.

**FixThisPlz is the open-source, self-hosted alternative.** It is built for developers, freelancers and agencies who want their clients and users to report problems easily, without building that tooling in-house.

And because many freelancers and small agencies have **no tracker at all** (bugs live in emails, chats and sticky notes), FixThisPlz ships with a small, opinionated tracker that works out of the box and nudges you towards good habits. If you already use GitHub Issues, Linear or Jira, just turn on the sync.

## What it is (and isn't)

| FixThisPlz **is** | FixThisPlz **is not** |
|---|---|
| A way for **humans** to report what they see | An error tracker or APM (use Sentry for that, the two complement each other) |
| **Capture → triage → fix → tell the reporter** | A project-management suite (no sprints, Gantt charts or time tracking) |
| A small, opinionated bug tracker that teaches good habits | Endlessly configurable: sensible defaults over options |
| Self-hosted, one container | A SaaS (there is no hosted version planned) |
| Stack-agnostic: any website, any framework | Tied to a specific CMS or framework |

## How it works

```
 Your website / app (any stack)
 ┌───────────────────────────────────┐
 │  FixThisPlz widget                │  1. User clicks "Report a problem",
 │  • screenshot + annotation        │     draws on the screenshot, types a note
 │  • URL, viewport, browser, OS     │  2. Context is attached automatically
 │  • recent console errors          │
 │  • failed network requests        │
 └─────────────────┬─────────────────┘
                   │ HTTPS (public project key)
 ┌─────────────────▼─────────────────┐
 │  FixThisPlz Hub (self-hosted)     │  3. Validated, rate-limited, stored
 │  • built-in tracker:              │  4. Triaged in the built-in tracker:
 │    inbox, list, Kanban            │     status, priority, assignee,
 │    statuses, priorities           │     notes, full history
 │    assignees, notes, history      │
 │  • projects, keys, spam filter    │
 │  • retention & purge              │
 └───┬──────────────┬────────────┬───┘
     │              │            │
     ▼              ▼            ▼
  Notify         Sync          Reporter status page
  (optional)     (optional)    "Your report is being fixed" ✓
  Slack /        GitHub /
  Discord /      GitLab /      5. The reporter is kept in the loop
  Teams /        Linear /         until the fix ships
  e-mail /       Jira
  webhook
```

## Quick start *(target experience)*

**1. Run the hub**

```bash
docker run -d -p 3000:3000 -v fixthisplz:/data ghcr.io/fixthisplz/hub
```

Open `http://localhost:3000`, create a project, copy its public key.

**2. Add the widget to your site**

```html
<script src="https://bugs.example.com/widget.js"
        data-project="pk_live_xxx" defer></script>
```

or with a bundler:

```js
import { FixThisPlz } from '@fixthisplz/widget';

FixThisPlz.init({ hub: 'https://bugs.example.com', projectKey: 'pk_live_xxx' });

// Optional: tell us who is reporting (never required)
FixThisPlz.identify({ email: user.email, name: user.name });
```

**3. That's it.** Reports land in the hub's inbox, ready to triage.
Optionally, connect Slack to be notified, or GitHub to open an issue for each report you confirm.

On WordPress, install the plugin instead: it embeds the widget and can talk to a hub or work standalone.

## Features

### Widget
- Framework-free **web component**, small (target < 30 kB gzipped), isolated with Shadow DOM so it never clashes with the host site's CSS
- **Annotated screenshot**: arrow, rectangle, highlight, blur
- Automatic context: URL, viewport, device, browser, OS, locale, recent console errors, failed requests
- Optional `identify()` and custom metadata (`plan`, `tenant`, app version…)
- Accessible (keyboard, screen readers), themeable, translatable

### Built-in tracker
- **Inbox** of new reports with an unread badge; a report is marked *seen* once opened
- **List** (filter, sort, search, bulk actions) and **Kanban** (one column per status, drag & drop)
- **Statuses** and **priorities** with sensible defaults (`new → seen → confirmed → in progress → fixed / won't fix / duplicate`), renamable and colourable
- **Assignee**, internal **notes** and a full **history** of every change
- **CSV export**: your data is never locked in
- Works on its own: the tracker is the source of truth unless you choose to sync with another one

### Hub
- Single container, **SQLite by default**, Postgres optional
- Multiple projects (one per client or site), each with a public key and an **allowed-origins list**
- Team members with simple roles (admin / member)
- Rate limiting and spam protection out of the box
- Configurable retention and one-click purge

### Integrations (all optional)
- **Notify**: Slack, Discord, Microsoft Teams, e-mail, signed JSON webhook
- **Sync**: GitHub, GitLab, Linear, Jira. A confirmed report opens an issue, and closing that issue marks the report fixed

### Reporter loop
- Reporters get a **magic link** to follow their report, with no account needed
- When the linked issue is closed, the reporter is told it's fixed

## Good practices, built in

FixThisPlz is opinionated on purpose. If you've never used a bug tracker, its defaults walk you through a workflow that professional teams rely on, without lectures or ceremony:

- **Reproducible by default**: every report already carries the URL, device, browser, console errors and failed requests, so nobody has to ask "which browser?" again
- **Triage first**: new reports wait in an inbox until someone decides *confirmed*, *won't fix* or *duplicate*. Nothing gets lost and nothing gets silently ignored
- **One owner per bug**: an assignee makes it clear who is on it
- **Priority ≠ urgency of the loudest client**: a small, fixed priority scale with short guidance on when to use each level
- **Ask instead of guessing**: a *needs info* status sends the question to the reporter, who answers through their magic link
- **Close the loop**: marking a report fixed asks for a short resolution note, which the reporter receives
- **Learn as you go**: short contextual tips in the UI explain *why* each step matters, and can be turned off once you know

The goal: a freelancer who installs FixThisPlz for a client project should come out of it with better habits than they went in with.

## Privacy & security by default

Screenshots of real apps contain real personal data. FixThisPlz treats that seriously:

- **Self-hosted**: data never leaves your infrastructure
- **Masking**: password fields, card inputs and elements marked `data-ftp-mask` are blurred *before* the screenshot leaves the browser
- **Retention**: reports and screenshots are deleted after a configurable period
- **Minimal collection**: identity is opt-in; IPs are not stored by default
- **Origin allow-list + rate limiting**: a public key in client-side JS is expected, and these limits are what make that safe

## Architecture

A single monorepo:

```
fixthisplz/
├── packages/
│   ├── widget/          # the web component (TypeScript, no framework)
│   ├── hub/             # API + dashboard (Node.js / TypeScript)
│   └── schema/          # shared report schema (JSON Schema + TS types)
├── integrations/
│   ├── github/
│   ├── slack/
│   └── webhook/         # reference implementation
├── adapters/
│   ├── wordpress/       # the original plugin
│   ├── react/
│   └── vue/
└── docs/
```

### Extending FixThisPlz

Integrations and adapters are the heart of the community. Each one is **small, self-contained and documented**, so you can add one without understanding the whole codebase.

An integration implements one interface:

```ts
export interface Integration<Config> {
  id: string;                       // "github", "linear", …
  name: string;
  configSchema: JSONSchema;         // renders the settings form in the dashboard
  send(report: Report, config: Config): Promise<{ externalUrl?: string }>;
}
```

The `webhook` integration is the reference implementation to copy from.

## Non-goals

Saying no keeps the project small and maintainable. FixThisPlz will **not**:

- become a project-management suite: no sprints, epics, story points, Gantt charts, time tracking or custom fields
- turn its workflow into a workflow builder: statuses and priorities can be renamed, not re-engineered
- do error monitoring, performance tracing or session replay
- offer a hosted SaaS version
- require reporters to create an account

## Roadmap

| Stage | Scope |
|---|---|
| **V0, extract** | Standalone `widget` package extracted from the WordPress plugin; the plugin (with its tracker) is the first backend |
| **V1, hub** | Self-hosted hub with the built-in tracker ported from the plugin: inbox, list, Kanban, statuses, priorities, assignees, notes, history; e-mail + webhook notifications |
| **V2, close the loop** | Reporter status page, *needs info* questions, resolution notes, in-app good-practice tips |
| **V3, integrations** | Notify (Slack, Discord, Teams) and sync (GitHub, GitLab, Linear); the integration contract is documented; React / Vue adapters |
| **Later** | Mobile (React Native "shake to report"), browser extension for testers |

## How it compares

| | FixThisPlz | Marker.io / BugHerd / Userback | Sentry User Feedback |
|---|---|---|---|
| Open source | ✅ MIT | ❌ | Partially |
| Self-hosted | ✅ | ❌ | ✅ (with full Sentry) |
| Built for non-technical reporters | ✅ | ✅ | ⚠️ secondary feature |
| Annotated screenshot | ✅ | ✅ | ✅ |
| Built-in tracker (no other tool needed) | ✅ | ✅ | ❌ (it's an error tracker) |
| Routes to your issue tracker | ✅ (optional) | ✅ | ✅ |
| Keeps the reporter informed | ✅ | Varies | ❌ |
| Price | Free | Per seat / per site | Part of Sentry plans |

## Contributing

FixThisPlz is at the very beginning, which is the best time to shape it.

- 💬 **Discuss the vision**: open an issue or a discussion; disagreement is welcome
- 🧩 **Build an integration or adapter**: look for the `good first issue` and `integration` labels
- 🌍 **Translate the widget** into your language
- 🐛 **Report a bug**: preferably with FixThisPlz, once it exists 😄

See [CONTRIBUTING.md](CONTRIBUTING.md). Everyone taking part agrees to our [Code of Conduct](CODE_OF_CONDUCT.md).

## Origin

FixThisPlz started as a WordPress plugin, then called *Bug Spotter*, built for a freelance developer's clients: a simple way for them to say "this is broken, here, look" without writing a long email.
It worked well enough that the obvious question came up: *why should this only work on WordPress?*

The name says what the project is about. It isn't software that hunts for bugs: it's a person, pointing at something, asking you to **fix this, plz**. Our job is to make sure that request reaches you with everything you need, and that the person hears back when it's done.

*Spelling: one word in code and URLs (`fixthisplz`), "FixThisPlz" or "Fix this, plz" in prose.*

## License

[MIT](LICENSE)
