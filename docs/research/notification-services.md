# Which notification services fit Cadence?

Research for [issue #4](https://github.com/nabilkaz/cohort-005-project-fork/issues/4), part of the map in [issue #3](https://github.com/nabilkaz/cohort-005-project-fork/issues/3).

- **Checked on:** 2026-11-23 (pricing pages, docs sites and npm registry metadata all read that day).
- **Sources:** each service's own pricing page, documentation and published npm package metadata. Nothing here comes from a third-party comparison.
- **Not done:** no account was created and no code was run. Every claim is what the docs say, not what was observed.
- **"Docs silent"** means the pages read did not answer the question. It is not a "no".

## The screen

From the map's Notes, a service is rejected if it:

1. has no free plan;
2. needs to call back to Cadence over the public internet (Cadence runs on a laptop);
3. demands a real login system.

It must also store notifications and read state itself, deliver live without a page reload, and let each person switch each kind of notification on or off.

## Summary

| | Knock | Novu | Courier | MagicBell | SuprSend |
| --- | --- | --- | --- | --- | --- |
| Free plan | 10,000 messages/mo | 10,000 workflow runs/mo, **5 workflows** | 10,000 sends/mo | **1,000 deliveries/mo**, 1 project | 10,000 notifications/mo |
| Feed kept for | Unlimited | 30 days | Docs silent | Docs silent | Docs silent |
| Works on a laptop, no inbound calls | Yes | Yes, with dashboard-built workflows | Yes | Yes | Not checked |
| Identifying the person | Public key + user id; signed token optional | App identifier + subscriber id; HMAC hash optional | Token always required, minted by a server call to Courier | Token always required, signed locally | Signed token (details not read) |
| Stores notifications and read state | Yes | Yes (implied, not stated outright) | Yes | Yes | Not checked |
| Live delivery | Yes, websocket | Yes, transport not named | Yes, transport not named | Yes, transport not named | Not checked |
| React | Components and hooks | Components and hooks | Components and one hook | Components; hooks docs silent | Components and headless |
| React 19 | Declared in package | Declared in package and docs | Package allows `>=18`; 19 not named | New package declares nothing | Package allows `>=16.8` |
| Server rendering | Docs silent | Docs silent | Client-side only, stated | Docs silent | Not checked |
| Per-kind switch per person | Yes, on free plan | Yes, on free plan | Yes; free-plan inclusion not stated | Yes; free-plan inclusion not stated | **Paid plans only** |
| Other channels later | Yes | Yes | Yes | Yes | Yes |
| Screen result | Passes | Passes | Passes | Passes | **Rejected** |

## Knock

**Free plan.** $0/month, "For evaluation or small-scale deployments. Start building for free, no credit card required." Up to 10,000 messages a month, where a message is one delivery to one person on one channel. Unlimited team members, environments, stored profiles and workflows. Feed retention unlimited; log retention 30 days. Preferences are included. Knock branding cannot be removed on Free. Going over the limit is not hard-capped: Knock gets in touch about billing. ([pricing](https://knock.app/pricing))

**Laptop only.** Cadence's server calls Knock's API to trigger a workflow; the browser fetches the feed from Knock's client API and holds a websocket to Knock. No page read describes Knock calling the customer's server for an in-app feed. Workflows can include a "fetch" step that does call out, but nothing says it is required. ([feeds overview](https://docs.knock.app/in-app-ui/feeds/overview), [workflows](https://docs.knock.app/concepts/workflows))

**Identifying the person.** The browser gives Knock a public API key and a user id (`user={{ id }}` on `KnockProvider`). By default that is all: "Without enhanced security mode, a request with your public key can act as any user." With enhanced security mode switched on, Knock rejects requests without a signed token. Cadence's server signs that token itself, locally, as an RS256 JWT with a signing key from the Knock dashboard and the user id in `sub`; no call to Knock is needed to mint it. The docs suggest leaving the mode off in a development environment. Nothing asks for a login system: Knock only needs an id, and only `id` is required on a user. People are created by sending their data or inline when a workflow is triggered. ([security](https://docs.knock.app/in-app-ui/security-and-authentication), [users](https://docs.knock.app/concepts/users))

For Cadence: the id would be the one in the session set by the DevUI panel. With the mode off, anyone with the page open could read another person's feed by changing the id. With it on, Cadence's server signs a token for whoever the session says is current.

**Storage and live delivery.** Knock stores feed messages and their state ("seen, read, interacted with, archived"). Clients "receive notifications over a websocket when new messages are sent to an in-app feed channel." ([feeds overview](https://docs.knock.app/in-app-ui/feeds/overview))

**React.** `@knocklabs/react` gives both. Ready-made: `NotificationFeedPopover`, `NotificationFeed`, `NotificationIconButton`. Headless: `useKnockClient`, `useNotifications`, `useNotificationStore`, `usePreferences`; with a custom UI Cadence marks items as seen itself. The package (0.15.1) declares peer support for React `^17 || ^18 || ^19`. Docs are silent on server rendering and on React Router; the package lists `next` and `@tanstack/react-router` as peers, not `react-router`. ([React overview](https://docs.knock.app/in-app-ui/react/overview), [feed](https://docs.knock.app/in-app-ui/react/feed), [custom UI](https://docs.knock.app/in-app-ui/react/custom-notifications-ui), [npm](https://registry.npmjs.org/@knocklabs/react/latest))

**Per-kind switch.** Yes. A person's preference set can opt out per workflow (`"workflows": { "mention": false }`), per category or per channel type; anything unset defaults to on, and an environment-wide default can be set. The `usePreferences` hook reads and writes them from the browser. There is also a Knock-hosted preference page. ([preferences](https://docs.knock.app/preferences/overview), [usePreferences](https://docs.knock.app/in-app-ui/react/sdk/hooks/use-preferences))

**Other channels later.** A workflow is a set of function and channel steps, triggered by its key. The docs do not say outright that adding an email step leaves trigger code untouched, but the trigger names only the workflow. Email needs `email` on the user. ([workflows](https://docs.knock.app/concepts/workflows), [users](https://docs.knock.app/concepts/users))

## Novu

**Free plan.** $0/month, no credit card. 10,000 workflow runs a month, all channels, unlimited subscribers. **At most 5 workflows**, 20 steps each. 2 environments, 3 team members. Inbox feed retention 30 days; activity feed (logs) retention 1 day. Novu branding stays on the Inbox on Free. ([pricing](https://novu.co/pricing), [limits](https://docs.novu.co/platform/developer/limits), [branding](https://docs.novu.co/platform/inbox/prepare-for-production))

**Laptop only.** Workflows built in the dashboard "run via Novu's Shared Bridge Endpoint", which Novu hosts, so nothing reaches Cadence. Workflows written in code with the Novu Framework need Cadence to "host their own Bridge application"; the page read does not say how Novu reaches it. Assume the dashboard route for a laptop. ([workflows](https://docs.novu.co/platform/concepts/workflows))

**Identifying the person.** The Inbox takes an application identifier (public) and a subscriber id, "typically the user's id in your database". An unknown id is created on first use. An optional HMAC setting adds a `subscriberHash`: HMAC-SHA256 of the subscriber id with the Novu secret key, computed on Cadence's server, locally. Without it, "a malicious actor could potentially guess another subscriber's `subscriberId`". Once the setting is on, the Inbox will not load without the hash. No login system is asked for. ([quickstart](https://docs.novu.co/platform/quickstart/react), [production](https://docs.novu.co/platform/inbox/prepare-for-production))

**Storage and live delivery.** The Inbox is "a prebuilt, ready-to-use, and fully customizable UI component for delivering real-time in-app notifications". Novu handles "delivery, unread counts, and preferences" and read state is synced to Novu. The pages do not say "we store them" in so many words, and name no transport, though the Inbox has a `socketUrl` prop. ([Inbox overview](https://docs.novu.co/platform/inbox/overview), [React SDK](https://docs.novu.co/platform/sdks/react))

**React.** `@novu/react` gives both: `Inbox`, `Bell`, `Notifications`, `InboxContent`, `Preferences`, plus headless hooks (the overview mentions them without naming them). It "requires React 18.0.0 or later" and the package (3.21.0) declares `^18 || ^19`. The React quickstart uses React Router for navigation via a `routerPush` prop, in a client-rendered Vite app. Docs are silent on server rendering. ([Inbox overview](https://docs.novu.co/platform/inbox/overview), [quickstart](https://docs.novu.co/platform/quickstart/react), [npm](https://registry.npmjs.org/@novu/react/latest))

**Per-kind switch.** Yes. A subscriber can set channel preferences for each workflow, and globally. A workflow marked critical is hidden from preferences and cannot be switched off. People change them in the Inbox's Preferences view, or Cadence can call the preferences API. Because the switch is per workflow, one kind of notification means one workflow, and the Free plan allows 5. ([preferences](https://docs.novu.co/platform/concepts/preferences))

**Other channels later.** In-app is one channel step beside Email, SMS, Push and Chat in the same workflow. ([workflows](https://docs.novu.co/platform/concepts/workflows))

## Courier

**Free plan.** "Developer": "Start free with 10,000 sends a month." The pricing page does not say whether the inbox or preferences are in or out of that plan, and gives no retention or branding terms. ([pricing](https://www.courier.com/pricing))

**Laptop only.** The flow is browser to Cadence's server, Cadence's server to Courier, token back to the browser; then "the inbox calls Courier directly from the browser". No inbound call from Courier is described. ([authenticate users](https://www.courier.com/docs/in-app/authenticate-users.md), [add an inbox](https://www.courier.com/docs/in-app/add-an-inbox.md))

**Identifying the person.** A token is always required; no keyless or development mode is documented. Cadence's server calls Courier's Issue Token endpoint with the workspace API key and a scope naming the user (`user_id:<id>` plus inbox scopes), then the browser calls `signIn({ userId, jwt })`. So every token costs one outbound call to Courier. Tokens can expire, and the SDK does not refresh them. The id is "whatever your app already calls this user" and need not exist in Courier first. The docs say to "read the user ID from your backend's own authenticated session", which for Cadence is the DevUI session. No login system is asked for beyond that. ([authenticate users](https://www.courier.com/docs/in-app/authenticate-users.md))

**Storage and live delivery.** "Courier stores each user's messages and delivers them in real time." "Read and archive state stays in sync across every device." Transport not named. ([inbox](https://www.courier.com/docs/in-app/inbox.md))

**React.** `@trycourier/courier-react` (v9): `CourierInbox`, `CourierInboxPopupMenu`, `CourierToast`, `CourierPreferences`, and a `useCourier()` hook for building a custom UI. A custom UI must call `inbox.listenForUpdates()` itself or it will not update live. The components "render client-side only". The package (9.3.5) allows React `>=18`; the docs say "React 18+" and never name 19. The older `@trycourier/react-inbox` packages are deprecated. Docs are silent on React Router. ([React SDK](https://www.courier.com/docs/sdk-libraries/courier-react-web.md), [npm](https://registry.npmjs.org/@trycourier/courier-react/latest))

**Per-kind switch.** Yes. Each template maps to a subscription topic with a default of opted in, opted out or required; a person's own choice overrides the default. Set through the `CourierPreferences` component, a hosted page or the API. Whether this is on the free plan is not stated. ([preferences](https://www.courier.com/docs/platform/preferences/preferences-overview))

**Other channels later.** Templates route to several channels; the docs describe one template sending to email and inbox, with opening the email marking the inbox message read. ([inbox overview](https://www.courier.com/docs/platform/inbox/inbox-overview))

## MagicBell

**Free plan.** "Builder", $0/month: 1,000 deliveries a month, 1 project, unlimited team members and users, all integrations. The page gives no retention, branding or preference terms. ([pricing](https://www.magicbell.com/pricing))

**Laptop only.** The browser holds a connection to MagicBell; Cadence's server signs tokens locally. No inbound call is described. ([in-app channel](https://www.magicbell.com/docs/channels/in_app), [user JWT](https://www.magicbell.com/docs/api/authentication/user))

**Identifying the person.** A User JWT is always required: "You'll need a User JWT to use the SDK." Cadence's server signs it locally, HS256, with the project secret key, carrying `user_external_id` or `user_email` (at least one) and the project's `api_key`. The docs do not say whether a login system is expected; they only need an identifier. ([user JWT](https://www.magicbell.com/docs/api/authentication/user), [React](https://www.magicbell.com/docs/libraries/magicbell-react))

**Storage and live delivery.** Notifications "track user interactions such as read, seen, and archived states". "MagicBell maintains a persistent connection to deliver notifications in real-time." Transport not named. ([notification](https://www.magicbell.com/docs/primitive/notification), [in-app channel](https://www.magicbell.com/docs/channels/in_app))

**React.** `@magicbell/react`: `ContextProvider`, `FloatingInbox`, `Inbox`, `InlineNotification`, `ToastContainer`, `UserPreferences`. The current docs describe no headless hooks. An older `@magicbell/react-headless` package still exists on npm (peer React `>=18.3.1`) but is not in the current docs. `@magicbell/react` (1.14.0) declares no peer dependencies, so React 19 support is not stated anywhere. Docs are silent on server rendering and React Router. ([React](https://www.magicbell.com/docs/libraries/magicbell-react), [npm](https://registry.npmjs.org/@magicbell/react/latest))

**Per-kind switch.** Yes in principle: `UserPreferences` "lets users toggle notification channels per category", and the API has a per-user preferences resource with categories and channels. The docs read do not show the exact shape for switching the in-app channel off for one category, nor whether preferences are on the free plan. ([React](https://www.magicbell.com/docs/libraries/magicbell-react), [API reference](https://www.magicbell.com/docs/api/reference))

**Other channels later.** Email, mobile push, web push, Slack, SMS and Teams are channels beside in-app. ([docs index](https://www.magicbell.com/docs))

## SuprSend (added candidate, rejected)

Free plan: $0/month, 10,000 notifications a month, with in-app components, embeddable and headless inbox, and toasts, all carrying SuprSend branding. But the pricing table marks "Channel Preferences", "Categories & Sub-category preferences" and "Ready UI for Preference Center" as not included on Free or Essentials; they start at Business ($250/month). ([pricing](https://www.suprsend.com/pricing))

That fails the per-kind switch on a free plan, so the remaining questions were not pursued. It would only come back into play if Cadence kept the switches in its own database and simply did not trigger switched-off kinds.

## Not screened

Engagespot, Liveblocks Notifications, NotificationAPI and Stream were not examined. Engagespot's pricing URL returned 404 and its React package was last published in May 2025.

## Shortlist

All four starting candidates pass the three rejection rules. In no particular order:

- **Knock.** The roomiest free plan for this job: no cap on workflows, feed kept indefinitely, preferences included. Both ready-made components and a full set of hooks, with React 19 declared. Securing feeds is optional and done by signing a token locally. Trade-off: with the security mode off, any browser can read any person's feed; server rendering is undocumented; branding stays.
- **Novu.** React 19 named in the docs, a quickstart that already uses React Router, and a ready-made preferences view inside the Inbox. Securing feeds is optional and is a one-line HMAC. Trade-off: the free plan allows 5 workflows, and since the switch is per workflow that caps Cadence at 5 switchable kinds; feed kept 30 days; logs kept 1 day; workflows must be built in the dashboard to avoid a bridge endpoint.
- **Courier.** Tokens are mandatory, so one person can never read another's feed, even in development. Ready-made inbox, toast and preferences components. Trade-off: every token needs an outbound call to Courier; rendering is client-side only; React 19 is not named; the pricing page does not confirm the inbox or preferences are on the free plan, nor how long messages are kept.
- **MagicBell.** Tokens are mandatory and signed locally with no extra call. Trade-off: 1,000 deliveries a month is a tenth of the others; the docs are the thinnest of the four (no headless hooks, no React version, no stated free-plan preferences).

### What separates them

- **Number of kinds.** More than 5 switchable kinds rules out Novu's free plan.
- **Our own components or theirs.** Knock and Novu document hooks for a fully custom feed. Courier offers one hook over the same client. MagicBell documents components only.
- **Keeping one person out of another's feed.** Courier and MagicBell force a token. Knock and Novu make it a setting, off by default.
- **Old notifications.** Knock keeps the feed indefinitely, Novu for 30 days. Courier and MagicBell do not say.

## Questions the docs did not answer

- Whether any of the four renders safely under server-rendered React Router. Only Courier addresses rendering at all ("client-side only"). All four will likely need the feed mounted in the browser only; that needs a quick trial.
- Whether Courier's and MagicBell's free plans include preferences and the inbox, and how long they keep messages.
- Which transport Novu, Courier and MagicBell use for live delivery.
- React 19 support for Courier (allowed by the package, not named) and MagicBell (nothing declared).
- Whether a Novu Framework bridge must be publicly reachable. Avoided by building workflows in the dashboard.
- The exact MagicBell call to switch off in-app delivery for one category.
