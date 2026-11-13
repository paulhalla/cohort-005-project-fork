# Hosted notification services for the in-app Notification inbox

Research for issue #2 (child of map #1). Read on 2026-11-14 from the vendors' own pricing pages, docs and SDK repositories. Nothing was signed up for or run; every claim below is what the vendor's page says, not something observed in a live account. Pricing and limits change often, so re-read the linked page before relying on a number.

This file compares; it does not recommend. Anything marked **Unverified** could not be confirmed from a primary source.

## Question

Which hosted notification services could back a persistent in-app Notification inbox (bell plus feed, read/unread state) for a React Router 8 app with a server-side Node backend, where the vendor owns storage and delivery and the app only triggers events and renders the inbox?

Hard constraints: usable free tier, hosted (no self-hosting), works from local dev, React or headless feed client. Live updating is a nice-to-have.

## Comparison table

| | Knock | Novu (cloud) | Courier | MagicBell | SuprSend |
| --- | --- | --- | --- | --- | --- |
| Free tier volume | 10,000 messages/mo | 10,000 workflow runs/mo, max 5 workflows | 10,000 sends/mo | 1,000 deliveries/mo, 1 project | 10,000 notifications/mo |
| In-app feed on free tier | Yes (Feed API and prebuilt components listed) | Yes ("In-app" channel listed) | Unverified on the pricing page; docs say Inbox needs no provider | Unverified on the pricing page ("All integrations") | Yes, "with SuprSend branding" |
| Inbox retention on free tier | Feed retention unlimited | **30 days** | Unverified | Unverified | Unverified (logs 30 days) |
| Vendor branding on free tier | Yes (removal is paid) | Yes (removal is paid) | Unverified | Unverified | Yes |
| Hosted | Yes | Yes (also self-hostable, not required) | Yes | Yes | Yes |
| Node SDK | `@knocklabs/node` | `@novu/api` | `@trycourier/courier` | `magicbell-js` | `@suprsend/node-sdk` |
| Prebuilt React inbox | `@knocklabs/react` | `@novu/react` `<Inbox />` | `@trycourier/courier-react` | `@magicbell/react` | `@suprsend/react` |
| Headless | React hooks plus `@knocklabs/client` | React hooks plus `@novu/js` | `useCourier` hook plus `@trycourier/courier-js` | `magicbell-js` user client (no React hooks in current docs) | React hooks (`useFeedClient`, `useFeedData`) |
| Browser auth for one user | Public key alone by default; RS256 user JWT when "enhanced security mode" is on | Application identifier alone by default; HMAC-SHA256 `subscriberHash` when the toggle is on | JWT minted by the backend, always | HS256 user JWT, always | Public key alone by default; ES256 user JWT when "enhanced security mode" is on |
| States | seen, read, interacted, link clicked, archived | read/unread, seen, archived, snoozed | read/unread, opened, archived | read/unread, seen, archived | seen, read, archived |
| Live updating | Yes, websocket | Yes, websocket | Yes, websocket | Yes, websocket | Yes, socket |
| Environments on free tier | Unlimited; development is opt-in | 2 (Development, Production) | Test and Production | 1 project | Sandbox, Staging, Production by default |
| Wording lives | Vendor-side template | Vendor-side template (or code-first via Novu Framework) | Either: vendor template or inline in the send call | Inline in the API call | Vendor-side template |
| Fails a hard constraint | No | No, but see retention | None found | None found | None found |

## Services that fail a hard constraint

None of the five services with verifiable data fails a hard constraint on the evidence read. Three caveats bear on the decision:

- **Novu, retention.** The free plan keeps Inbox feed items for 30 days ([limits](https://docs.novu.co/platform/developer/limits)). That is not one of the four hard constraints, but it conflicts with a "persistent" inbox: older notifications disappear. It also caps the free plan at 5 workflows.
- **MagicBell, volume.** 1,000 deliveries a month is a tenth of the others' free allowance ([pricing](https://www.magicbell.com/pricing)). Usable for development; whether it is "usable" in production depends on volume. The pricing page says deliveries beyond the plan limit are "charged per use"; how that applies to the $0 plan is unverified.
- **Engagespot, not assessable.** Its pricing page returned 404 on every attempt, so the free-tier constraint could not be checked from a primary source. It is neither passed nor failed. See its section.

"Works from local dev" was assessed from docs only: no page read describes an origin allow-list, an inbound webhook, or a public URL requirement for the plain trigger-plus-feed path. That is absence of evidence from reading, not a confirmed run from localhost. The one documented exception is Novu's optional code-first Framework, which needs Novu Cloud to reach a Bridge Endpoint in your app.

## Knock

- **Free tier.** $0, "up to 10,000" messages sent per month. Unlimited feeds per recipient, Feed API and prebuilt components included, feed retention unlimited, log retention 30 days, unlimited team members and isolated environments. "Remove Knock branding" is "No" on free; the next plan is $250/mo. The FAQ says Knock does not hard-cap messages past the free limit and contacts you about billing instead. [Pricing](https://knock.app/pricing)
- **Hosted.** Yes.
- **Trigger and Node SDK.** `@knocklabs/node`; the client reads `KNOCK_API_KEY`. `client.workflows.trigger('workflow-key', { recipients: ['user-id'], data: {...} })`. Recipients can be identified inline in the trigger call. [SDK repo](https://github.com/knocklabs/knock-node), [trigger docs](https://docs.knock.app/send-notifications/triggering-workflows/api)
- **Feed client.** `@knocklabs/react` ships `NotificationFeedPopover`, `NotificationFeed` and `NotificationIconButton`, and can be used "in a headless way and bring your own UI". The headless path is `useKnockClient`, `useNotifications(knockClient, channelId)`, `useNotificationStore(feedClient)`, then `feedClient.fetch()`, `markAsSeen`, `markAllAsSeen`. [React overview](https://docs.knock.app/in-app-ui/react/overview), [custom UI](https://docs.knock.app/in-app-ui/react/custom-notifications-ui)
- **Browser auth.** By default the public API key plus a user id is enough, and the key "can act as any user". Enhanced security mode requires a user JWT signed by the backend with RS256 using a signing key from the dashboard (`KNOCK_SIGNING_KEY`; `signUserToken('user-1')` in the Node SDK). It is off by default, and the docs suggest leaving it off in development and on in production. [Security docs](https://docs.knock.app/in-app-ui/security-and-authentication)
- **State model.** Engagement statuses are "mutually inclusive": seen (fetched for display in the feed), read, interacted, link clicked, archived, each with a timestamp. Feed content cannot be changed after generation. [Message statuses](https://docs.knock.app/send-notifications/message-statuses), [feeds overview](https://docs.knock.app/in-app-ui/feeds/overview)
- **Live updating.** Yes. The client "receives notifications over a websocket when new messages are sent to an in-app feed channel". [Feeds overview](https://docs.knock.app/in-app-ui/feeds/overview)
- **Local development.** An account starts with production only; enabling environments (one-way) adds development, each environment with its own API keys. [Environments](https://docs.knock.app/concepts/environments)
- **Wording.** Vendor-side: the in-app template is edited in the dashboard on the workflow's in-app step, with Liquid variables filled from the trigger's `data`. [In-app channel](https://docs.knock.app/integrations/in-app/knock)
- **Tests.** No first-party mock or dry-run mode was found. The message-status page describes a "Not sent" status for a channel "in sandbox mode"; whether sandbox mode applies to the in-app feed channel is **Unverified**.

## Novu (cloud)

- **Free tier.** $0, no credit card. 10,000 workflow runs/mo, up to 5 workflows, 2 environments, up to 3 team members, in-app channel included. Novu branding shows on the Inbox UI; removing it needs a paid plan (Pro, from $30/mo). [Pricing](https://novu.co/pricing), [production guide](https://docs.novu.co/platform/inbox/prepare-for-production)
- **Retention.** Inbox feed retention is 30 days on Free, 90 days on Pro and Team. Activity feed (the log) is 1 day on Free. [Limits](https://docs.novu.co/platform/developer/limits). The pricing page's "Activity feed retention: 24 hours" is the log figure, not the inbox one.
- **Hosted.** Yes. Novu is also self-hostable, but the cloud needs none of that.
- **Trigger and Node SDK.** `@novu/api`; `new Novu({ secretKey })`, then `novu.trigger({ workflowId, to: { subscriberId }, payload })`. EU region uses `serverURL: "https://eu.api.novu.co"`. [Server SDK](https://docs.novu.co/platform/sdks/server/typescript)
- **Feed client.** `@novu/react` (React 18+, React 19 supported) with `<Inbox applicationIdentifier subscriberId routerPush />`, composable sub-components (Bell, Notifications, InboxContent, Preferences), and hooks such as `useNotifications` and `useCounts` for a fully custom UI. `@novu/js` is the framework-agnostic client. [Inbox overview](https://docs.novu.co/platform/inbox/overview), [quickstart](https://docs.novu.co/platform/quickstart/react), [useNotifications](https://docs.novu.co/platform/sdks/react/hooks/use-notifications)
- **Browser auth.** The public application identifier plus a subscriber id by default; an unknown subscriber id is created on first use. HMAC is a per-environment toggle: `subscriberHash = HMAC-SHA256(secretKey, subscriberId)` as hex, computed on the server. Once on, the Inbox does not load without the hash. The docs describe turning it on for production; that it is off by default in development is implied, not stated. [Production guide](https://docs.novu.co/platform/inbox/prepare-for-production)
- **State model.** Notification methods: `read`, `unread`, `seen`, `archive`, `unarchive`, `snooze`, `unsnooze`, `delete`. Counts are exposed as unread and unseen. Snooze on Free is limited to 24 hours per notification (from a search summary of the snooze page, not read directly). [useNotifications](https://docs.novu.co/platform/sdks/react/hooks/use-notifications), [tabs](https://docs.novu.co/platform/inbox/configuration/tabs)
- **Live updating.** Yes. A `realtime` option controls a WebSocket subscription; "The notifications list is automatically updated in real-time when new notifications arrive". [useNotifications](https://docs.novu.co/platform/sdks/react/hooks/use-notifications)
- **Local development.** Development and Production environments by default, each with its own application identifier and secret key; workflows are published from Development to Production. Custom environments need Team or Enterprise. [Environments](https://docs.novu.co/platform/developer/environments)
- **Wording.** Vendor-side by default: dashboard template editors "with variables and LiquidJS logic", filled from the trigger payload. Alternatively code-first with Novu Framework, where workflows live in the app's source and Novu Cloud calls a Bridge Endpoint the app exposes; the docs say this goes "through an encrypted client-initiated tunnel". The local setup for that tunnel was not read. [Workflow overview](https://docs.novu.co/platform/workflow/overview), [Framework overview](https://docs.novu.co/framework/overview)
- **Tests.** No first-party mock mode was found. The publish API has a `dryRun` option, which is about promotion, not sending.

## Courier

- **Free tier.** Developer plan: free, "10,000 sends per month", SDKs and CLI included. A multi-channel template counts as one send. "Sends from Test and custom environments count the same as Production." Past 10,000, a credit card moves the workspace to Business at $0.005 per send "including the first 10,000". The pricing page does not name Inbox as a Developer feature, so its inclusion is **Unverified** there; the docs say the in-app channel needs no provider setup. [Pricing](https://www.courier.com/pricing), [Inbox overview](https://www.courier.com/docs/platform/inbox/inbox-overview)
- **Hosted.** Yes.
- **Trigger and Node SDK.** `@trycourier/courier` (v7+); `client.send.message({ message: { to: { user_id }, template, data } })`, or inline `content: { title, body }` instead of `template`. `routing: { method, channels }` accepts `inbox` as a channel. `POST /send` takes an optional `Idempotency-Key`. [Send reference](https://www.courier.com/docs/reference/send/message)
- **Feed client.** `@trycourier/courier-react` for React 18+ (`@trycourier/courier-react-17` for React 17): `<CourierInbox />`, `<CourierInboxPopupMenu />`, `<CourierToast />`, themed with `CourierInboxTheme`, and you can "supply your own views". Headless: the `useCourier()` hook (`auth`, `inbox`, `toast`, `preferences`) or `@trycourier/courier-js` with `getMessages`, `read`, `unread`, `open`, `click`, `archive`, `unarchive`, `readAll`. Components are client-side only. The older `@trycourier/react-inbox` and `react-provider` packages are deprecated. [React SDK](https://www.courier.com/docs/sdk-libraries/courier-react-web), [JS SDK](https://www.courier.com/docs/sdk-libraries/courier-js-web)
- **Browser auth.** Always a JWT: the backend calls `POST /auth/issue-token` (`client.auth.issueToken`) with a scope such as `user_id:user_123 inbox:read:messages inbox:write:events`, and the browser calls `signIn({ userId, jwt })`. Omitting `expires_in` "mints a token that never expires"; SDKs do not refresh tokens. No unauthenticated development mode is documented. [Authentication](https://www.courier.com/docs/platform/inbox/authentication)
- **State model.** Read/unread, opened, archived; `InboxMessage` carries optional `read`, `opened`, `archived` fields. State syncs across devices. [JS SDK](https://www.courier.com/docs/sdk-libraries/courier-js-web)
- **Live updating.** Yes, but explicit: `inbox.listenForUpdates()` "opens the WebSocket connection required for real-time updates"; without it the inbox loads once. The unread total is not pushed over the socket and must be refetched. [React SDK](https://www.courier.com/docs/sdk-libraries/courier-react-web)
- **Local development.** Keys are "scoped to one environment, such as Test or Production"; the docs advise starting with Test. Custom environments are Enterprise. A dedicated environments page was not found (404), so details beyond this are **Unverified**. [API reference](https://www.courier.com/docs/reference/get-started)
- **Wording.** Either: a vendor-side template referenced by id with `data`, or title and body passed inline by the caller. [Send reference](https://www.courier.com/docs/reference/send/message)
- **Tests.** No first-party mock mode was found; test-environment sends still count toward the 10,000.

## MagicBell

- **Free tier.** Builder: $0, "1,000 Deliveries/mo.", "1 Project", "All integrations", unlimited team members and users. Next plan is $249/mo for 50,000 deliveries. The in-app inbox is not named as a plan feature, so its inclusion is **Unverified** on the pricing page. [Pricing](https://www.magicbell.com/pricing)
- **Hosted.** Yes.
- **Trigger and Node SDK.** `magicbell-js`; `import { Client } from 'magicbell-js/project-client'`, then `client.broadcasts.createBroadcast({ title, content, recipients, actionUrl, category, topic })`. Recipients are given by `external_id` or email, 1 to 1,000 per broadcast. Authenticated with a project JWT from the dashboard. [JS client](https://www.magicbell.com/docs/libraries/magicbell-js), [broadcast](https://www.magicbell.com/docs/primitive/broadcast)
- **Feed client.** `@magicbell/react` with `Provider`, `FloatingInbox` and `Inbox`; slots `ButtonComponent`, `InboxComponent`, `ItemComponent` are replaceable, and styling is by CSS variables, class names, or `unstyled`. Headless: `magicbell-js/user-client` (`listNotifications`, `markNotificationRead`, `markNotificationUnread`, `archiveNotification`, `unarchiveNotification`, `markAllNotificationsRead`, `fetchUnreadNotificationsCount`). The current React page documents no headless hooks; whether the older headless React package is still supported is **Unverified**. [React library](https://www.magicbell.com/docs/libraries/magicbell-react), [JS client](https://www.magicbell.com/docs/libraries/magicbell-js)
- **Browser auth.** Always a user JWT, signed on the backend with HS256 using the project's Secret Key, with claims `user_external_id` and/or `user_email` plus `api_key`. No unauthenticated development mode is documented. [User auth](https://www.magicbell.com/docs/api/authentication/user)
- **State model.** `read_at` and `seen_at` timestamps, plus archive/unarchive endpoints. No snooze was found. [Notification](https://www.magicbell.com/docs/primitive/notification)
- **Live updating.** Yes. The in-app channel "maintains a persistent connection to deliver notifications in real-time"; headless use is `new Realtime({ token }).listen(cb)` over WebSocket with automatic reconnect. [In-app channel](https://www.magicbell.com/docs/channels/in_app), [JS client](https://www.magicbell.com/docs/libraries/magicbell-js)
- **Local development.** The free plan has one project, so development and production would share data and the 1,000 deliveries unless a second account is used. No separate environment concept was found in the pages read; **Unverified** beyond that.
- **Wording.** Passed inline by the caller: title and content go in the broadcast call, with optional per-channel `overrides`. No vendor-side template is required.
- **Tests.** No first-party mock mode was found.

## SuprSend

Not named in the ticket; included as another credible service in the category.

- **Free tier.** $0, no credit card, "10k Notifications/mo", "In-app notification feed (with SuprSend branding)", unlimited team members, 30-day log retention. Next plan is $100/mo. New accounts get a 30-day all-features trial and then move to Free. [Pricing](https://www.suprsend.com/pricing)
- **Hosted.** Yes.
- **Trigger and Node SDK.** `@suprsend/node-sdk`; `new Suprsend(workspaceKey, workspaceSecret)`, then `supr_client.workflows.trigger(new WorkflowTriggerRequest({ workflow, recipients, data }, { idempotency_key }))`. [Node trigger](https://docs.suprsend.com/docs/node-trigger-workflow-from-api)
- **Feed client.** `@suprsend/react`: `<SuprSendProvider><Inbox /></SuprSendProvider>` with replaceable `bellComponent`, `badgeComponent`, `notificationComponent` and others, or headless with `SuprSendFeedProvider`, `useFeedClient`, `useFeedData`. The older `@suprsend/react-inbox` is unsupported. [Drop-in feed](https://docs.suprsend.com/docs/react-in-app-feed), [headless feed](https://docs.suprsend.com/docs/react-sdk-headless-feed)
- **Browser auth.** A public key by default. Enhanced security mode is a toggle beside the key, "recommended for Production workspaces", which then requires a user JWT signed on the backend with ES256; the SDK has a `refreshUserToken` callback. That it is off by default elsewhere is implied, not stated. [Client authentication](https://docs.suprsend.com/docs/client-authentication)
- **State model.** Seen, read, archived; archived items are hidden unless a store asks for them. Method names beyond `markAsSeen` and `markAsRead` were not read. [Headless feed](https://docs.suprsend.com/docs/react-sdk-headless-feed)
- **Live updating.** Yes, over a feed socket; the Inbox shows connection status (`ONLINE`, `CONNECTING`, `DEGRADED`, and so on). [Drop-in feed](https://docs.suprsend.com/docs/react-in-app-feed)
- **Local development.** "Every account starts with three default workspaces (Sandbox, Staging, and Production)", each with its own keys. A Test Mode exists in Staging and custom workspaces; what it does was not read. Workspace limits "depend on your billing plan"; the free-plan figure is **Unverified**. [Workspaces](https://docs.suprsend.com/docs/workspaces)
- **Wording.** Vendor-side templates created in the dashboard, filled from the trigger's `data`.
- **Tests.** Test Mode (above) is the only vendor-side mechanism found; its behaviour is **Unverified**.

## Engagespot (not assessable)

The docs describe a hosted in-app inbox with a React component (`@engagespot/react-component`, `<Engagespot apiKey userId userToken />`), a React hooks section, realtime delivery, and a "secure authentication mode" using a backend-signed JWT that is optional but required for production. [React quickstart](https://docs.engagespot.co/docs/sdks-n-libraries/frontend/react/quick-start/)

The pricing page (`engagespot.co/pricing`) and home page returned 404 on every attempt, so the free tier is **Unverified**. Third-party directories report a free plan of 10,000 triggers a month with 7-day feed retention; those are not primary sources and disagree with each other, so they are not relied on here.

## Not investigated

Stream, OneSignal and other adjacent products (activity feeds, push and in-app messaging) were not researched. They are not ruled in or out.

## Avoiding the service in code under test

No vendor documents a mock transport or offline mode for its Node SDK in the pages read. What follows is inference from the SDK shapes, not a vendor claim: each trigger is a single call on a client constructed with an API key (`workflows.trigger`, `novu.trigger`, `send.message`, `broadcasts.createBroadcast`), so the practical seam is the app's own module that wraps that call, replaced in tests. The browser feed client is the other point of contact and only runs in the browser.

Two vendor-specific points bear on this: Courier counts Test-environment sends against the free allowance, and MagicBell's free plan has a single project, so tests that do reach either service spend the same quota as real use.

## Unverified

- Whether Inbox is included in Courier's and MagicBell's free plans (not named on either pricing page).
- Inbox item retention for Courier, MagicBell and SuprSend.
- Branding on the free inbox for Courier and MagicBell.
- That Novu's HMAC and SuprSend's enhanced security are off by default outside production (implied by the docs, not stated).
- Whether Knock's channel sandbox mode applies to the in-app feed.
- Courier's environment behaviour beyond "keys are scoped to one environment" (the environments page was not found).
- MagicBell's overage behaviour on the $0 plan, and whether it has any environment concept besides projects.
- SuprSend's Test Mode behaviour and its free-plan workspace limit.
- Engagespot's free tier and pricing altogether.
- Novu's free-plan snooze limit (seen only in a search summary).
- "Works from local dev" for every vendor: judged from docs, not from a run on localhost.
