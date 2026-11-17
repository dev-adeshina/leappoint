**File:** `docs/UI-UX/design-specification.md`  
**Status:** Design proposal derived from PRD v1.0 and the current SDD draft  
**Product name:** Not supplied — use the working title *Travel Services Marketplace*  
**Audience:** Product owner, UI/UX designer, and frontend developer  
**Design target:** Responsive web application  
**Design principle:** No invention. Simplicity is the standard.

---

## 1. Purpose

This document defines the user experience and visual interface for the Travel Services Marketplace described in the Product Requirements Document (PRD) and Software Design Document (SDD).

It translates the approved product scope into:

- User journeys and screen responsibilities.
- Information architecture and navigation.
- Page-level wireframes.
- High-fidelity visual direction and reusable UI rules.
- Responsive behaviour.
- Loading, empty, error, and success states.
- Interactive prototype flows and acceptance criteria.

It is a design specification, not a new product requirements document. It does not authorize features that are absent from the PRD.

### 1.1 Source documents

| Source | Version/status used | Role in this specification |
|---|---|---|
| [`docs/PRD/PRD.md`](https://github.com/dev-adeshina/leappoint/blob/main/docs/PRD/PRD.md) | PRD v1.0, Draft | Product scope, actors, journeys, functional requirements, trust boundaries and MVP exclusions |
| [`docs/SDD/travel-sdd.md`](https://github.com/dev-adeshina/leappoint/blob/main/docs/SDD/travel-sdd.md) | Draft derived from PRD v1.0 | Application boundaries, business modules, technical constraints, integrations and open decisions |

**Source precedence:** The PRD defines what the product must do. The SDD defines the proposed technical structure. This document defines how those requirements are presented and interacted with. If a visual idea conflicts with either source, the source documents take precedence.

### 1.2 Design notation

- **Required** — explicitly supported by the PRD or SDD.
- **Design proposal** — a visual or interaction recommendation that does not change business behaviour.
- **Open decision** — the source documents do not yet define the behaviour. The design must remain neutral until resolved.
- **Deferred** — outside MVP or blocked by an unresolved dependency.

---

## 2. Product experience principles

### 2.1 Core principles

1. **Make the next action obvious.** Every page should have one clear primary action and a small number of relevant secondary actions.
2. **Show real status, not optimistic status.** A booking is not confirmed until the external booking provider confirms it. A payment is not successful until verified. A visa outcome must never be implied.
3. **Keep customer and provider workspaces distinct.** Their goals and permissions differ; do not combine their navigation into one overloaded dashboard.
4. **Keep operational information scannable.** Use clear page titles, concise summaries, tables/lists, status labels and contextual actions.
5. **Use progressive disclosure.** Put essential information first; reveal secondary details when needed.
6. **Use plain language.** Prefer “Waiting for your information” to internal workflow terminology.
7. **Do not manufacture trust.** Do not invent provider verification badges, customer counts, ratings, savings, success rates, testimonials or guarantees.
8. **Preserve privacy boundaries.** Provider-only notes must never appear in customer views. Support-only access must not be represented as ordinary user access.
9. **Do not over-design.** Avoid decorative charts, excessive gradients, unnecessary animation, oversized card grids and ornamental dashboard widgets.
10. **Design for the actual MVP.** No native-app patterns, built-in video calls, automated visa eligibility decisions or unsupported booking inventory.

### 2.2 Product areas

The product has three experience areas:

- **Public marketplace:** discover providers and services; enter flight or hotel search; learn what the platform offers.
- **Customer account:** manage service inquiries/orders, consultations, messages, progress and travel bookings.
- **Provider workspace:** manage provider profile, listings, inquiries, quotes, appointments, cases, client bookings and team access.
- **Administration/support workspace:** manage providers, users, listings, orders, support, disputes, categories, workflow templates and audit review.

The platform administrator/support experience is a separate permission boundary, not a general dashboard available to providers.

---

## 3. Roles and access-aware experience

| Role | Main user goal | Main interface |
|---|---|---|
| Visitor | Understand the platform, browse public providers/services, begin travel search | Public website |
| Customer | Find help, inquire/purchase, book consultations, follow cases, message providers, book travel | Customer account |
| Provider owner | Operate provider account and manage services, customers, appointments, cases and bookings | Provider workspace |
| Provider team member | Complete assigned provider work | Provider workspace, limited by assigned permissions |
| Platform administrator/support agent | Review and support platform activity within authorized scope | Admin/support workspace |

### 3.1 Access rules visible in the UI

- A provider application must not be presented as an approved public provider profile until approval is complete.
- Provider team members see only actions permitted by their assigned role/permissions.
- Customer case pages show customer-visible milestones, notes and attachments only.
- Internal provider notes must be visually and technically separated from customer-visible updates.
- Admin access to conversations is for authorized support purposes; do not add a casual “view all chats” shortcut.
- Do not imply a provider is officially endorsed. Verification wording must reflect the actual approved check.
- Do not expose another customer's records through provider client lists, search, URLs or shared browser state.

---

## 4. User journeys

Journey maps below show the experience required by the PRD. Where the PRD leaves a decision open, the journey identifies the dependency rather than inventing a default.

### 4.1 Customer — find and purchase a service

**Goal:** Find a suitable provider and obtain a service.

| Stage | User action | Interface response | Key screen/state |
|---|---|---|---|
| Discover | Select a service category or search | Show relevant service listings and available filters | Marketplace / search results |
| Compare | Open a listing or provider profile | Present service scope, provider details, terms, pricing method and consultation options | Service detail / provider profile |
| Decide | Choose inquiry, quote request or direct purchase where enabled | Show the next required step; do not force a purchase where inquiry is the chosen path | Inquiry / purchase entry |
| Authenticate | Sign in or register if required | Preserve the intended destination and return after authentication | Sign in / register |
| Submit | Send inquiry or accept the listed purchase/quote path | Confirm submission and show its current status | Inquiry or order detail |
| Pay | Pay if payment is required | Show amount, currency, terms and payment state | Payment step |
| Follow | Open the service order | Show status history, progress milestones, actions required and case-linked conversation | Customer order/case |
| Complete | Provider marks work complete | Show completion state and eligible review action | Completed order |

**Primary design requirement:** The customer must understand what is being purchased, what is included, what is excluded or required, the pricing method, estimated duration and applicable cancellation/refund terms before committing.

**Do not design:** guaranteed visa approval, a government application submission workflow, an automatic eligibility verdict or a guaranteed processing date.

### 4.2 Customer — book a consultation

| Stage | User action | Interface response | Key screen/state |
|---|---|---|---|
| Choose | Select a provider and consultation type | Show title, topic, format, duration, free/paid status and preparation details | Consultation details |
| Review | Check terms and available slots | Show provider availability and time zone; render appointment time in the customer's local time while retaining the relevant zone context | Availability |
| Select | Choose a time slot | Temporarily reflect the selected slot; prevent duplicate booking at the server boundary | Booking form |
| Pay | Pay if paid | Do not show paid appointment as confirmed before successful payment unless approved provider configuration allows it | Payment state |
| Confirm | Complete booking | Show requested/confirmed status according to provider configuration | Appointment confirmation |
| Manage | View, cancel or request rescheduling according to policy | Show current appointment status and applicable terms | Appointment detail |

**Open decisions:** automatic confirmation versus provider approval; meeting-link provider; free consultation limits; cancellation/reschedule/no-show policy. Do not invent these rules in the UI.

### 4.3 Customer — track a service case

| Stage | User action | Interface response | Key screen/state |
|---|---|---|---|
| Open | Open an active service order | Show a concise order summary and current progress | Case overview |
| Understand | Review milestones | Separate completed, current, upcoming, waiting and blocked steps | Progress tracker |
| Act | Respond to a request for information | Make the required action prominent and explain what is needed | Action required |
| Communicate | Open case-linked chat | Show conversation context, timestamps and unread state | Case conversation |
| Follow | Return later | Show latest update time and any changed milestone | Case overview |
| Finish | Review completed case | Show completion and eligible review action | Completed case |

Milestone states supported by the PRD: Not started, In progress, Waiting for you, Waiting for an external party, Completed, Skipped and Blocked. The customer view should use understandable labels and distinguish provider activity from external authority decisions.

### 4.4 Customer — direct flight booking

| Stage | User action | Interface response | Key screen/state |
|---|---|---|---|
| Search | Enter origin, destination, dates and traveler count | Validate required search fields | Flight search |
| Compare | Review returned offers | Show offer details, total price and restrictions returned by Duffel | Flight results |
| Select | Choose an offer | Show selected itinerary and current price/terms | Offer detail |
| Enter details | Provide required traveler details | Validate required fields and clearly identify errors | Traveler details |
| Submit | Continue to booking/payment according to integration | Show pending state while external provider responds | Booking processing |
| Confirm | External provider confirms | Display confirmed status and booking reference | Booking detail |
| Manage | Review applicable change/cancellation/support information | Show information available through the booking arrangement | Booking detail |

**Critical state rule:** Pending, failed, cancelled and action-required are distinct states. Never style a pending booking as confirmed.

### 4.5 Customer — hotel booking

The PRD includes hotel search and booking, but the SDD states that no hotel provider has been selected.

**Design treatment:** Reserve a hotel entry point and a neutral search-page structure only. Do not create realistic search results, hotel cards, prices, availability, booking steps or confirmation states until the hotel provider, API capabilities, payment flow and confirmation process are selected.

### 4.6 Provider owner — onboarding and approval

| Stage | User action | Interface response | Key screen/state |
|---|---|---|---|
| Apply | Start provider application | Explain information required and review process without promising approval | Provider application |
| Submit | Complete provider/business details | Validate required fields and show submission summary | Application review |
| Wait | Application under review | Show a clear pending status and next-step explanation | Application status |
| Decision | Application approved/rejected or further action required | Show actual decision and available action | Application status |
| Activate | Enter provider workspace after approval | Show setup tasks relevant to profile and services | Provider workspace |

Do not expose unpublished provider profiles in public search. Do not invent document requirements or verification checks beyond the approved application policy.

### 4.7 Provider — manage inquiries, quotes and service orders

| Stage | User action | Interface response | Key screen/state |
|---|---|---|---|
| Review | Open inquiry queue | Show customer, service, received date and current status | Inquiries |
| Respond | Open inquiry and reply | Keep the inquiry context visible | Inquiry detail |
| Quote | Create and send a quote where applicable | Show amount, currency, terms and quote status | Quote composer |
| Convert | Customer accepts quote | Reflect accepted quote and resulting order state | Order detail |
| Manage | Update order status and customer actions | Show status history and customer-visible update controls | Order detail |
| Complete | Mark service complete | Record completion and expose eligible next step | Completed order |

### 4.8 Provider — manage a case

| Stage | User action | Interface response |
|---|---|---|
| Create/manage | Open a service order/case | Show customer, service and order context |
| Set workflow | Select or create a workflow template | Show template steps before applying |
| Adapt | Adjust milestones for this customer | Make clear changes apply to this case only |
| Update | Change milestone state and add update | Separate customer-visible update from internal note |
| Request action | Mark customer action required | Display exact request and relevant due/estimated date if supplied |
| Communicate | Use case-linked conversation | Preserve case context and unread indicators |
| Audit | Review milestone history | Show who changed the milestone and when |

### 4.9 Provider — book travel for a client

| Stage | User action | Interface response |
|---|---|---|
| Select client | Choose an existing client record or create one | Keep client identity visible throughout |
| Search | Search flights (and hotels only when integration exists) | Use the same booking tools as the customer flow |
| Compare | Review offers and restrictions | Show supplier-provided details |
| Submit | Enter traveler details and submit booking | Show pending/processing state |
| Record | Store booking under provider and client | Show external reference and actual status |
| Share | Share booking details with customer | Use the permitted customer communication path |

### 4.10 Administrator/support — platform operations

Admin tasks are operational rather than consumer-facing. The interface should prioritize queues, filters, record search and traceable actions.

Core journeys:

- Review provider application → inspect submitted information → approve/reject/suspend according to policy.
- Manage listing → inspect listing → update or suspend where authorized.
- Find customer/order/appointment/booking → inspect relevant status → handle authorized support task.
- Handle report/dispute → review related context → record outcome/action.
- Manage categories and workflow templates.
- Review audit events.

Administrative actions that affect accounts, orders, payments or access should require clear confirmation and display the consequence before submission.

---

## 5. Information architecture

### 5.1 Public website

```text
Public
├── Home
├── Services
│   ├── Service category
│   ├── Search results
│   └── Service detail
├── Providers
│   ├── Provider search/results
│   └── Provider profile
├── Flights
│   ├── Flight search
│   ├── Flight results
│   ├── Offer details
│   └── Traveler details / booking
├── Hotels
│   └── Reserved entry point (integration pending)
├── Sign in
├── Create account
└── Provider application
```

The exact public route for guest inquiries is an open PRD decision. Until resolved, do not promise that a visitor can submit an inquiry without an account.

### 5.2 Customer account

```text
Customer account
├── Overview
├── My services
│   ├── Active orders
│   ├── Order detail / case tracker
│   └── Completed / cancelled orders
├── Appointments
│   ├── Upcoming
│   └── Appointment detail
├── Messages
│   └── Context-linked conversation
├── Travel bookings
│   ├── Flights
│   └── Hotels (only after integration is selected)
├── Notifications
└── Profile & preferences
```

The account overview should summarize existing customer work, not become a marketing dashboard. Only show sections that have data or an actionable empty state.

### 5.3 Provider workspace

```text
Provider workspace
├── Overview
├── Inquiries
│   └── Inquiry detail / quote
├── Service orders
│   └── Order detail / case tracker
├── Appointments
│   ├── Appointment list
│   └── Consultation types & availability
├── Clients
│   └── Client detail
├── Travel bookings
│   ├── Flight search / results / booking
│   └── Hotels (integration pending)
├── Messages
├── Services & profile
│   ├── Public profile
│   └── Service listings
├── Team & access
└── Account settings
```

Do not add provider revenue analytics, marketing campaigns, CRM automation, inventory management or staff performance scoring: these are not defined in the source documents.

### 5.4 Admin/support workspace

```text
Administration
├── Overview
├── Provider applications
├── Providers
├── Users
├── Service listings
├── Service orders
├── Appointments
├── Travel bookings
├── Payments & refunds
├── Support cases / reports / disputes
├── Service categories
├── Workflow templates
└── Audit log
```

Payment and payout screens must reflect only the commercial and settlement model eventually approved. Do not design unsupported payout controls as if they are already operational.

---

## 6. Navigation design

### 6.1 Public desktop header

**Layout:** Brand wordmark placeholder on the left; primary links in the middle; account actions on the right.

```text
┌──────────────────────────────────────────────────────────────────────┐
│ [MARKETPLACE]   Services   Providers   Flights   Hotels   [Sign in] │
│                                                        [Get started] │
└──────────────────────────────────────────────────────────────────────┘
```

- Use the project’s actual product name and logo when supplied. Until then, use a text placeholder.
- Keep navigation limited to the main discovery destinations.
- “Get started” may lead to registration; do not imply a specific paid conversion.
- On small screens, collapse links into a menu with keyboard-accessible open/close behaviour.

### 6.2 Customer navigation

Desktop: compact left sidebar with a top utility bar. Mobile: bottom navigation for the most-used destinations plus a “More” menu for secondary destinations.

Recommended primary items: Overview, My services, Appointments, Messages, Travel bookings. Profile, notifications and preferences remain accessible from the top utility/account menu.

### 6.3 Provider navigation

Desktop sidebar: Overview, Inquiries, Service orders, Appointments, Clients, Travel bookings, Messages, Services & profile, Team & access.

Use clear active navigation state. Group related items only when the grouping reduces scanning effort. Avoid deeply nested navigation.

Mobile provider navigation should not attempt to show the entire desktop sidebar at once. Use a compact bottom bar for high-frequency tasks and a full navigation drawer for the rest.

### 6.4 Admin navigation

Use a persistent desktop sidebar and a compact mobile drawer. Admin navigation should be visibly distinct from the public/customer shell so users can recognize the operational context.

### 6.5 Navigation behaviour

- Keep the current page title visible.
- Show breadcrumbs on deeper provider/admin records where they aid orientation.
- Preserve search/filter state when opening a record and returning to a list.
- Display unread message indicators only when unread messages exist.
- Display notification indicators only when unread notifications exist.
- Do not use color alone to indicate the active route or a status.

---

## 7. Screen inventory and high-fidelity page specifications

This section defines the screens that should be designed before frontend implementation. “High fidelity” means the designer should specify hierarchy, dimensions, component treatment, content density, responsive behaviour and interaction—not merely choose a color palette.

### 7.1 Public home page

**Purpose:** Explain the marketplace and give visitors direct paths to service discovery and flight search.

**Desktop structure:**

1. Header/navigation.
2. Hero section with concise headline, supporting copy and primary action.
3. Prominent search/entry panel with service discovery and flight-search entry.
4. Service-category discovery section using only categories supported by the service catalogue.
5. Provider/service discovery preview.
6. Clear “how it works” explanation based on the customer journey.
7. Trust and safety explanation: providers assist; relevant authorities make visa decisions.
8. Footer with only confirmed links and policies.

**Visual direction:** Take inspiration from the Fast Flight reference’s strong hero, clean navigation and prominent search widget. Adapt it to this multi-service marketplace. Do not copy its aircraft artwork, exact layout, wording or unsupported social-proof claims.

**Do not include:** invented passenger counts, review scores, partner logos, approval rates, “best price” claims, guaranteed outcomes or fabricated provider cards.

**Primary action:** Explore services or search flights, depending on the selected hero entry.

### 7.2 Service search/results

**Desktop layout:**

```text
┌─────────────────────────────────────────────────────────────────────┐
│ Header                                                              │
├─────────────────────────────────────────────────────────────────────┤
│ Services  /  Search results                                         │
│ [Search service or destination........................] [Search]    │
├───────────────────┬─────────────────────────────────────────────────┤
│ FILTERS           │ Results                                         │
│ Category          │ [Sort control if supported]                     │
│ Destination       │ ┌─────────────────────────────────────────────┐ │
│ Provider type     │ │ Provider / service name                     │ │
│ Language          │ │ Short description                           │ │
│ Price method      │ │ Destination · Language · Estimate           │ │
│ Availability*     │ │ Price / Request a quote                     │ │
│                   │ │ [View details]                              │ │
│                   │ └─────────────────────────────────────────────┘ │
│                   │ ...                                             │
└───────────────────┴─────────────────────────────────────────────────┘
```

`*` Availability filter only when the underlying information is available.

**Listing card content:** provider name, service name, short description, supported destination, language, price/pricing method, estimated duration when provided, and a clear detail action.

**Filters:** category, destination, provider type, language, price and availability where information exists. Do not show disabled or fabricated filters.

**Mobile:** filters open in a sheet/drawer; results become a single column. Keep active filter count and a clear reset action.

### 7.3 Provider public profile

Show:

- Provider/business name and provider type.
- Description.
- Supported service categories and destinations.
- Languages and operating hours.
- Published service listings.
- Consultation types where available.
- Customer reviews only when supported and eligible.
- Accurate verification wording, if an approved verification process exists.
- Clear inquiry/booking action.

Do not use a verification badge by default. If the verification process is not defined, omit the badge rather than invent its meaning.

### 7.4 Service detail

**Recommended hierarchy:**

1. Service name and provider identity.
2. Short summary.
3. Price or pricing method (“Fixed price”, “Request a quote”, or both as configured).
4. What is included / deliverables.
5. Supported destination/location.
6. Prerequisites and information the customer may need.
7. Estimated duration with an explicit “estimate” label.
8. Cancellation/refund terms.
9. Primary action: inquire, request quote or purchase as configured.
10. Related consultation option where one exists.

Keep the primary action visible on desktop. On mobile, a restrained sticky action bar may be used only if it does not obscure content or accessibility controls.

### 7.5 Authentication

Sign-in, registration, email verification and password reset should use a consistent narrow form layout.

- Email and password fields.
- Clear validation.
- Password visibility toggle.
- Links between sign-in, registration and password reset.
- Email verification confirmation and resend action where supported.
- Do not add social login or phone-number authentication; not specified by the PRD.

After authentication, return the user to the intended protected action where possible.

### 7.6 Customer overview

**Purpose:** Help customers resume work.

Recommended content order:

1. Page title and brief welcome.
2. Action-required item(s), if any.
3. Active service orders with current status.
4. Upcoming appointments.
5. Recent messages/unread state.
6. Recent travel bookings and current booking status.
7. Empty-state entry points for services, consultations or flights.

Do not invent KPI cards or financial charts. Use actual user-specific records only.

### 7.7 Customer service order / case detail

**Desktop two-column layout:**

```text
┌────────────────────────────────────────────────────────────────────┐
│ My services / Order reference                                      │
│ Service title                         [Current status]              │
│ Provider name · Order number · Created date                         │
├──────────────────────────────────────────────┬─────────────────────┤
│ CASE PROGRESS                                │ ORDER SUMMARY       │
│ ● Completed — milestone                      │ Provider            │
│ ● Completed — milestone                      │ Service             │
│ ◉ In progress — current step                 │ Amount / currency   │
│ ○ Upcoming — next step                       │ Order status        │
│ ○ Upcoming — next step                       │ Terms               │
│                                              │                     │
│ ACTION REQUIRED (when applicable)             │ CONTACT             │
│ What the customer needs to do                │ [Open conversation]│
│ [Go to conversation]                         │                     │
│                                              │                     │
│ CUSTOMER-VISIBLE UPDATES                     │                     │
│ Latest update and date                       │                     │
└──────────────────────────────────────────────┴─────────────────────┘
```

On mobile, stack the order summary below the progress and action-required content.

Show completed/current/upcoming states distinctly. Display latest update date. Label estimates. Do not expose internal notes. If a milestone is waiting on an external authority, state that it is external and avoid implying the provider controls the decision.

### 7.8 Appointments

Appointment list should support clear separation of upcoming and past appointments. Each row/card shows consultation type, provider, date/time with timezone context, format, duration and status.

Appointment detail shows the applicable cancellation/reschedule terms and meeting details when available. Do not create a built-in video meeting screen; online meetings use an external link in the MVP.

### 7.9 Messages

Use a simple two-pane desktop layout:

- Conversation list: context label, other participant, latest message preview, timestamp and unread marker.
- Conversation panel: context header, message history, timestamps and text composer.

Each conversation must show whether it relates to an inquiry, service order, appointment or booking. Do not create a general-purpose social inbox disconnected from a business context.

Mobile: conversation list opens first; selecting a conversation opens a full-screen thread with a clear back action.

Attachments remain hidden/disabled until secure file types, limits, access, retention and deletion rules are approved. Do not show an upload button that cannot be safely supported.

### 7.10 Provider overview

**Purpose:** Surface provider work that needs attention.

Recommended content:

1. Page title and provider account status.
2. New/awaiting inquiries.
3. Orders requiring provider action.
4. Upcoming appointments.
5. Recent messages/unread state.
6. Recent booking activity.
7. Shortcuts to manage services and profile.

Use compact count summaries only when the underlying records can supply them. Avoid decorative analytics and invented revenue widgets.

### 7.11 Provider inquiry and quote detail

Show inquiry context, customer identity, related service, received date, messages/updates and current status.

Quote composition should include only fields supported by the approved quote model: amount, currency, terms and expiry only if expiry is implemented. Show a review summary before sending. Keep “Send quote” visually distinct from “Save draft” only if draft persistence is actually supported.

### 7.12 Provider service order / case workspace

Use a clear work area with:

- Customer and order summary.
- Current order status and status history.
- Milestone list with state, responsible party and customer-facing visibility.
- Actions to start, complete, skip, block or update milestones, according to the PRD.
- Customer action request.
- Customer-visible update composer.
- Separate internal note composer/area.
- Case-linked conversation access.

Internal notes must have a persistent “Internal — not visible to customer” label and a distinct visual treatment. Never rely on color alone to separate them.

### 7.13 Provider appointments and consultation setup

Appointment list: date, customer, consultation type, duration, format, status and actions permitted by policy.

Consultation type form: title/topic, description, free/paid, duration, price/currency when paid, meeting format, timezone, availability, booking window/advance notice, cancellation/rescheduling terms, preparation information and confirmation mode.

Do not invent a calendar provider, meeting provider or availability rule. The design should display the selected provider timezone and customer-local time.

### 7.14 Provider clients and assisted bookings

Client list should be searchable and paginated. Client detail should show the records associated with that provider only. The assisted flight booking flow should make the selected client visible at every stage to prevent accidental booking under the wrong customer.

### 7.15 Provider services and profile management

Profile editor should be organized into clear sections: identity/business details, description, contact/operating locations, services/destinations, languages, operating hours and relevant professional/business details. Listing editor should include name/category, description/deliverables, destinations, prerequisites, pricing method, estimated duration, required customer information and cancellation/refund terms.

The form must show public preview or a clear summary of what will be published. Do not invent mandatory credentials or verification documents.

### 7.16 Team and access

Show team members, membership status and assigned roles/permissions according to the final permission model. The PRD requires owner invitations and role assignment but does not define the role catalogue. Until approved, design a neutral role assignment structure without hard-coded permission names or assumed defaults.

### 7.17 Admin/support overview and queues

Use a restrained operations dashboard inspired by the information hierarchy of the Live Monitor reference: important queues first, recent activity second, analytics only when supported by real data.

Suggested overview modules:

- Provider applications awaiting review.
- Reports/support cases requiring action.
- Recent order/appointment/booking issues.
- Recent audit events, subject to admin permissions.

Do not invent “live visitors”, checkout funnels, sales totals, geographic demand, revenue targets or conversion charts. Those metrics are not specified as admin dashboard requirements.

Admin list pages should use search, filters, status, created/updated date and pagination where applicable. Row actions must reflect permission and record state.

---

## 8. Wireframe library

These are structural wireframes, not pixel-perfect artwork. High-fidelity design should preserve the hierarchy and content responsibilities.

### 8.1 Flight search

```text
┌─────────────────────────────────────────────────────────────────────┐
│ Header                                                              │
├─────────────────────────────────────────────────────────────────────┤
│ Flights                                                             │
│ Find a flight                                                       │
│ [Round trip] [One way] [Multi-city]                                 │
│ [From........] [To..........] [Swap]                                │
│ [Departure date] [Return date] [Travelers] [Search flights]         │
└─────────────────────────────────────────────────────────────────────┘
```

Trip types are explicitly present in the Fast Flight inspiration and are consistent with the flight-search concept, but confirm the booking integration supports each option before exposing it as functional. If not confirmed, expose only supported options.

### 8.2 Flight results

```text
┌─────────────────────────────────────────────────────────────────────┐
│ Search summary: Origin → Destination · Dates · Travelers [Edit]     │
├──────────────────────┬──────────────────────────────────────────────┤
│ FILTERS              │ Available offers                              │
│ Stops                │ ┌──────────────────────────────────────────┐ │
│ Time                 │ │ Airline / flight segments                │ │
│ Price                │ │ Departure — Duration — Arrival           │ │
│ Other supported      │ │ Baggage / restrictions if supplied        │ │
│ filters              │ │ Total price                    [Select]   │ │
│                      │ └──────────────────────────────────────────┘ │
│                      │ ...                                          │
└──────────────────────┴──────────────────────────────────────────────┘
```

Only show filters and fare details returned by the integration. Do not fabricate baggage allowance, refundability or seat availability.

### 8.3 Provider work queue

```text
┌─────────────────────────────────────────────────────────────────────┐
│ Provider workspace / Inquiries                         [New filter] │
│ [Search customer or inquiry.........................]               │
│ [All] [New] [Awaiting response] [Quote sent]                         │
├─────────────────────────────────────────────────────────────────────┤
│ Customer     Service          Received       Status       Open      │
│ Customer A   Service name     Date           New          [View]    │
│ Customer B   Service name     Date           Awaiting...  [View]    │
└─────────────────────────────────────────────────────────────────────┘
```

Status tabs must use actual implemented statuses. Do not hard-code a simplified lifecycle if the PRD allows service-specific states.

### 8.4 Admin review queue

```text
┌─────────────────────────────────────────────────────────────────────┐
│ Administration / Provider applications                              │
│ [Search provider........................] [Status filter]           │
├─────────────────────────────────────────────────────────────────────┤
│ Provider       Type          Submitted       Status       [Review]  │
│ ...                                                               ...│
└─────────────────────────────────────────────────────────────────────┘
```

Review detail should separate submitted information, review decision, and administrative action history.

---

## 9. Visual design direction

### 9.1 Reference interpretation

The following references are inspiration, not product requirements or templates to copy.

1. **[Fast Flight — Premium Flight Booking Web Design](https://dribbble.com/shots/25501705-Fast-Flight-Premium-Flight-Booking-Web-Design)**  
   Borrow: clean top navigation, strong headline hierarchy, prominent booking/search panel, generous spacing and focused CTA.  
   Do not borrow: its exact brand, aircraft illustration, promotional claims, passenger counts, ratings or any feature not supported by the PRD.

2. **[Flight Booking — Web App](https://dribbble.com/shots/22091492-Flight-Booking-Web-App)**  
   The Dribbble project describes a flight-search and fare-comparison experience. Borrow the idea of a clear search-to-results progression, offer comparison and a coherent booking flow.  
   Do not assume every visual element or fare capability is supported by Duffel; display only data supplied by the integration.

3. **[Modern E-Commerce Admin Dashboard — Live Monitor](https://dribbble.com/shots/27526914-Modern-E-Commerce-Admin-Dashboard-UIUX-Design-Live-Monitor)**  
   Borrow: calm operational hierarchy, clear modules, restrained data visualization and quick recognition of items needing attention.  
   Do not borrow commerce-specific metrics such as live visitors, checkout funnels, sales by region or revenue performance unless separately required.

### 9.2 Overall visual character

**Direction:** Modern, calm, premium-but-practical travel marketplace.

- Light interface as the default.
- Neutral page backgrounds.
- White or near-white content surfaces.
- One restrained primary accent.
- Dark, high-contrast typography.
- Soft borders and subtle shadows; avoid heavy glassmorphism.
- Rounded corners used consistently, not excessively.
- Real travel/provider imagery only where it explains or supports content.
- No decorative gradients behind every card.
- No large illustration that pushes the search form below the fold.

The public website may use more expressive imagery than the account workspaces. Customer, provider and admin screens should prioritize clarity and task completion.

### 9.3 Proposed color tokens

These are **design proposals**, not brand colors from the PRD or SDD. They should be validated against the product owner's brand direction before implementation.

| Token | Proposed value | Use |
|---|---|---|
| `color.brand.700` | `#1D4ED8` | Primary actions, active navigation |
| `color.brand.600` | `#2563EB` | Hover/interactive emphasis |
| `color.brand.50` | `#EFF6FF` | Subtle selected/active backgrounds |
| `color.text.primary` | `#172033` | Main text |
| `color.text.secondary` | `#667085` | Supporting text |
| `color.text.muted` | `#98A2B3` | Timestamps and secondary metadata |
| `color.surface.page` | `#F7F8FA` | App background |
| `color.surface.card` | `#FFFFFF` | Cards, panels, forms |
| `color.border.default` | `#E5E7EB` | Dividers and outlines |
| `color.success` | `#16803D` | Success state |
| `color.warning` | `#B7791F` | Waiting/attention state |
| `color.danger` | `#C0392B` | Errors/destructive actions |
| `color.info` | `#2563A6` | Informational state |

Status colors must always be paired with text labels and/or icons. Check contrast before release.

### 9.4 Typography

**Proposal:** Use one readable sans-serif family across the application. A system sans-serif stack is acceptable for the first implementation; do not add multiple font families without a brand reason.

| Style | Desktop size | Weight | Use |
|---|---:|---:|---|
| Display | 48–56 px | 600–700 | Public hero only |
| H1 | 32 px | 600 | Page title |
| H2 | 24 px | 600 | Main section title |
| H3 | 18–20 px | 600 | Card/panel title |
| Body | 16 px | 400 | Default content |
| Body small | 14 px | 400 | Supporting content |
| Label | 13–14 px | 500 | Form labels and table metadata |
| Caption | 12 px | 400 | Timestamps and helper text |

Mobile hero and page-title sizes should reduce fluidly. Avoid tiny text for table content and status labels.

### 9.5 Spacing and layout tokens

Use a consistent 4 px base spacing scale.

`4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80`

- Public content max width: approximately 1200–1280 px.
- Dashboard content max width: approximately 1440 px.
- Desktop sidebar: approximately 240–256 px.
- Top utility bar: approximately 64–72 px.
- Standard card padding: 20–24 px.
- Form field height: approximately 44–48 px.
- Table row height: approximately 56–64 px.
- Use whitespace to group content; do not wrap every field in a card.

These are starting values for Figma and CSS, not fixed business requirements.

### 9.6 Shape, border and elevation

- Inputs and buttons: 8–10 px radius.
- Cards/panels: 12–16 px radius.
- Pills: fully rounded only for compact status/category labels.
- Default border: 1 px neutral border.
- Shadows: subtle and reserved for overlays, dropdowns and elevated surfaces.
- Avoid multiple nested cards.

### 9.7 Iconography

Use one consistent line-icon family. Icons must be accompanied by accessible labels when meaning is not universally obvious. Do not use emoji as interface icons.

### 9.8 Imagery

- Public landing page: travel photography may support the hero and destination context.
- Provider profiles: use provider-supplied profile/logo imagery only when available.
- Service cards: prefer simple category imagery or no image; do not invent provider assets.
- Dashboard: keep imagery minimal; operational records should remain the focus.
- Do not use airline logos unless returned/authorized through the booking provider or otherwise licensed.

---

## 10. Component design system

### 10.1 Buttons

| Variant | Purpose | Behaviour |
|---|---|---|
| Primary | Main action on a page/form | One dominant primary action per visual section |
| Secondary | Supporting action | Neutral outline or surface treatment |
| Tertiary/ghost | Low-emphasis action | Used for contextual actions |
| Destructive | Delete, suspend, cancel where authorized | Requires clear consequence and confirmation where appropriate |
| Link | Navigation or inline action | Underlined or otherwise clearly distinguishable |

Button states: default, hover, focus-visible, pressed, disabled, loading.

Never use a disabled-looking button to hide a required explanation. Provide helper text or an inline reason where useful.

### 10.2 Form fields

Standard field anatomy:

- Visible label.
- Input/control.
- Optional helper text.
- Inline validation/error message.
- Required indicator where applicable.

Use correct input types and autocomplete attributes. Do not rely on placeholder text as the only label.

### 10.3 Status badges

Use a small set of visual treatments, but preserve the actual status wording from the business record. Do not merge semantically different states.

Examples:
- Success: Completed, Confirmed, Approved.
- Attention: Waiting for you, Requested, Action required.
- Neutral: Not started, Pending, In progress.
- Danger: Failed, Rejected, Cancelled, Disputed (where applicable).
- External: Waiting for an external party.

Color is supplementary; status text remains mandatory.

### 10.4 Cards

Use cards for distinct entities or grouped information: service listing, provider profile summary, appointment summary, booking offer, order summary. Do not card-wrap every section of a page.

### 10.5 Tables and lists

Use tables for dense provider/admin management data on desktop. On mobile, transform rows into stacked summary cards or a responsive list with key information and actions.

Required behaviours:
- Clear column labels.
- Row hover/focus state.
- Sort only where sort behaviour exists.
- Pagination for long datasets.
- Empty state within the table/list region.
- Preserve filter state when opening and returning from a record.

### 10.6 Search and filters

- Search field with visible label or accessible name.
- Filters should reflect actual searchable data.
- Show active filter chips and a clear-all action.
- On mobile, use a filter sheet/drawer.
- Do not expose unsupported filter choices.

### 10.7 Progress tracker

A vertical timeline is preferred for service case milestones.

Each item can display:
- Milestone name.
- State.
- Responsible party.
- Customer-visible description/update.
- Date or estimate if supplied.
- Action required, if any.

Completed, current, upcoming, blocked and external-waiting states must be visually distinct and textually identified.

### 10.8 Messaging components

- Conversation list item.
- Message bubble.
- Timestamp.
- Unread marker.
- Context label.
- Composer.
- Send/loading/failed state.

Keep message bubbles readable and avoid chat-app decoration that distracts from the business context.

### 10.9 Dialogs, drawers and toasts

- Dialogs: destructive confirmation, short focused decisions.
- Drawers: mobile filters, navigation, contextual detail where appropriate.
- Toasts: transient confirmation for low-risk actions only.
- Important business changes (quote sent, appointment requested, booking status changed) require a durable page-level confirmation or status record, not only a toast.

---

## 11. Responsive layout specification

The application is responsive web. Native mobile applications are explicitly out of MVP.

### 11.1 Breakpoint proposal

| Range | Layout behaviour |
|---|---|
| `>= 1280 px` | Full desktop layout; sidebar and multi-column content |
| `1024–1279 px` | Compact desktop; reduce gutters and card columns |
| `768–1023 px` | Tablet; collapsible sidebar, reduced columns |
| `< 768 px` | Mobile; single-column content, compact navigation, stacked forms |

Breakpoints are design tokens to validate in implementation, not PRD requirements.

### 11.2 Public pages

- Desktop: centered content, horizontal navigation, multi-column provider/service results.
- Tablet: reduce grid columns and maintain readable search controls.
- Mobile: single-column sections; hero search form stacks vertically; filters move to a sheet; buttons become full-width where appropriate.

### 11.3 Customer workspace

- Desktop: sidebar + content region; case details may use two columns.
- Tablet: collapsible sidebar; summary panels stack as needed.
- Mobile: single-column; case tracker first, action-required next, order summary after; conversation uses full-screen thread.

### 11.4 Provider workspace

- Desktop: persistent sidebar and data tables.
- Tablet: collapsible navigation; tables may scroll horizontally only when the data cannot be transformed.
- Mobile: prioritize queue review, case updates, messages and appointment details; use stacked rows and full-width actions.

### 11.5 Admin workspace

- Desktop: sidebar + dense data tables.
- Tablet/mobile: compact drawer and responsive list cards; avoid tiny multi-column tables.
- High-impact actions must remain easy to inspect and confirm on mobile.

### 11.6 Responsive form rules

- Stack multi-column forms on narrow screens.
- Keep labels above controls.
- Do not use tiny two-column inputs for long values.
- Keep primary submit action reachable without obscuring the final fields.
- Preserve entered data after validation errors.

---

## 12. Loading, empty, error, success and special states

Every asynchronous or data-dependent page must have an intentional state. Do not leave blank panels or rely on a spinner as the only feedback.

### 12.1 Global loading

- Initial page load: restrained skeletons matching the eventual content shape.
- Button submission: inline loading label and disabled repeat submission.
- Search: retain search criteria and show progress in the results region.
- Real-time refresh: update the affected row/section without blocking the whole page.
- Avoid indefinite loading. Provide retry or support guidance when an operation times out.

### 12.2 Empty states

| Screen | Empty-state message intent | Action |
|---|---|---|
| Service search | No matching services found | Adjust search or clear filters |
| Customer services | No service orders yet | Browse services |
| Appointments | No upcoming appointments | Find a consultation |
| Messages | No conversations yet | Start from an inquiry, order or appointment |
| Travel bookings | No bookings yet | Search flights |
| Provider inquiries | No inquiries yet | Review/publish services if available |
| Provider orders | No service orders yet | Review inquiries |
| Provider appointments | No appointments yet | Configure consultation types/availability |
| Provider clients | No client records yet | Create/select client when beginning assisted work |
| Admin queue | No records match this view | Clear filters or change status |

Empty states must not imply an error. Keep illustration optional and restrained.

### 12.3 Error states

| Error | UI response |
|---|---|
| Invalid form field | Inline message beside the field; preserve other entered values |
| Authentication failure | Clear, non-revealing message; allow retry/password reset |
| Session expired | Explain that sign-in is required; preserve safe return destination |
| Permission denied | Explain that the user does not have access; do not reveal protected record details |
| Record unavailable | Show a not-found state; do not expose whether another user's record exists |
| Search provider unavailable | Explain that live results cannot currently be retrieved; allow retry |
| Payment failed | Show failed state, amount/context and safe retry/support route; do not mark order paid |
| Booking failed | Show failed state and current external response where safe; do not show confirmation |
| Booking pending | Show “Pending confirmation” and explain that the external provider has not confirmed yet |
| Message send failed | Keep unsent message available for retry; do not duplicate on repeated submission |
| Realtime disconnected | Show a subtle connection warning and continue to allow manual refresh/reload of persisted data |
| Unexpected error | Generic message, retry where safe, and reference/support path if available |

### 12.4 Success states

- Registration submitted: explain email verification next step.
- Email verified: confirm and provide sign-in/continue action.
- Inquiry submitted: show inquiry reference/status if available.
- Quote accepted: show accepted state and resulting next step.
- Payment succeeded: show verified payment state and linked order/appointment.
- Appointment booked: show actual requested/confirmed state and timezone.
- Milestone updated: show the updated milestone and latest update date.
- Message sent: display it in the thread with timestamp.
- Flight booking confirmed: show confirmed status and booking reference only after provider confirmation.
- Provider application submitted: show under-review/pending state.
- Admin action completed: show the result and preserve an audit trail.

### 12.5 Status-specific presentation

**Flight/hotel booking:** confirmed, pending, failed, cancelled and action-required must remain distinct.

**Service order:** use actual order status and status history; do not assume a single linear workflow.

**Appointment:** awaiting payment, requested, confirmed, rescheduling requested, completed, cancelled and no-show are distinct.

**Case milestone:** not started, in progress, waiting for you, waiting for external party, completed, skipped and blocked are distinct.

**Provider application:** use only the real application review states implemented by the platform.

---

## 13. Accessibility and usability requirements

- Target WCAG 2.2 AA as a design goal, subject to the launch-market standard being confirmed.
- All interactive elements must be keyboard accessible.
- Provide visible focus indicators.
- Use semantic headings and landmarks.
- Every form control must have a programmatic label.
- Error messages must be associated with their fields.
- Do not rely on color alone for status or validation.
- Maintain sufficient text and control contrast.
- Support zoom and reflow without horizontal page scrolling at common mobile widths.
- Use accessible names for icon-only controls.
- Respect reduced-motion preferences.
- Ensure dialogs manage focus and return focus to the triggering control.
- Tables need meaningful headers and accessible responsive alternatives.
- Date/time displays must include timezone context where relevant.

---

## 14. Interaction and prototype specification

The prototype should demonstrate the critical journeys rather than every possible screen transition.

### 14.1 Prototype A — public discovery to service inquiry

1. Home → Services.
2. Select a category.
3. View results and apply a filter.
4. Open service detail.
5. Open provider profile.
6. Select inquiry action.
7. Sign in/register if required by the approved guest-inquiry policy.
8. Submit inquiry.
9. Show confirmation and inquiry status.

### 14.2 Prototype B — service order tracking

1. Customer overview.
2. Open active service order.
3. Review progress timeline.
4. Open an action-required milestone.
5. Follow the request into the case-linked conversation.
6. Send a message.
7. Return to the order and show the latest update.

### 14.3 Prototype C — consultation booking

1. Provider profile.
2. Select consultation type.
3. Review duration, format, price and terms.
4. Select a time slot.
5. Complete payment if required.
6. Show actual requested/confirmed state.
7. Open appointment detail.

The prototype must show both free and paid branches, but must not invent the final confirmation policy.

### 14.4 Prototype D — flight booking

1. Flights page.
2. Enter route, dates and traveler count.
3. Submit search.
4. Compare returned offers.
5. Select an offer.
6. Enter traveler details.
7. Submit booking.
8. Show pending state.
9. Transition to confirmed only in the prototype's confirmed scenario.

Use sample data clearly marked as prototype data. Do not present prototype data as live inventory.

### 14.5 Prototype E — provider operations

1. Provider overview.
2. Open inquiry queue.
3. Review inquiry.
4. Compose/send quote.
5. Open resulting service order.
6. Select workflow template.
7. Adapt a case milestone.
8. Add customer-visible update.
9. Add internal note and demonstrate that it is not visible in customer view.
10. Open case-linked conversation.

### 14.6 Prototype F — admin provider review

1. Admin overview.
2. Open pending provider application.
3. Review submitted details.
4. Approve/reject using a clearly labeled action.
5. Confirm action.
6. Show resulting status and audit event.

### 14.7 Prototype quality rules

- Use realistic but clearly fictional sample records.
- Link all primary buttons to a meaningful next screen.
- Include hover, focus, disabled and loading states for primary components.
- Include at least one validation error and one success state per critical form.
- Demonstrate responsive behaviour at desktop, tablet and mobile widths.
- Do not simulate successful external payment or booking without labeling it as prototype behaviour.

---

## 15. Content and microcopy rules

### 15.1 Language

Use direct, neutral language. Avoid marketing superlatives and claims that cannot be substantiated.

Preferred:
- “Find a service provider”
- “Request a quote”
- “Waiting for your information”
- “Pending confirmation”
- “Estimated completion”
- “The relevant authority makes the visa decision”

Avoid:
- “Guaranteed visa”
- “100% approval”
- “Instant confirmation” unless the integration guarantees it
- “Verified expert” unless the verification process supports that exact claim
- “Best price” without substantiation

### 15.2 Date, time and money

- Show currency alongside monetary values.
- Show totals and applicable terms before payment.
- Use the appointment's relevant timezone and display customer-local time clearly.
- Label estimated dates as estimates.
- Do not invent a default currency or locale while the initial market/currency decision remains open.

### 15.3 Trust and safety copy

Where relevant, state that providers offer assistance and that visa decisions are made by the relevant authorities. Avoid promises about visa approval, processing times or travel outcomes.

---

## 16. Frontend implementation guidance

This section translates the design into practical frontend work without prescribing an unapproved framework or routing library.

### 16.1 Keep the interface aligned with business modules

Organize React pages/components around the same business areas used by the SDD:

- Identity and access.
- Provider onboarding and profiles.
- Service catalogue and discovery.
- Inquiries, quotes and orders.
- Consultations and appointments.
- Case progress.
- Messaging.
- Travel booking.
- Payments.
- Notifications.
- Reviews.
- Administration/support.

This is an organizational guide, not a requirement to create a separate component or file for every item.

### 16.2 Reusable components

Start with a small shared component set:

- App shell (public/customer/provider/admin variants).
- Header and sidebar navigation.
- Page header.
- Button and form controls.
- Status badge.
- Search/filter bar.
- Data table/list.
- Summary card.
- Empty/loading/error state.
- Confirmation dialog.
- Progress timeline.
- Conversation list and message thread.

Prefer existing project conventions and components when available. Do not create parallel design systems or duplicate components without a demonstrated need.

### 16.3 Data-driven UI

- Render status from server-provided business state.
- Render prices/currency from the actual listing/quote/booking record.
- Render available filters based on supported data.
- Render provider verification language only from the approved verification state.
- Render booking confirmation only from the external provider-confirmed state.
- Keep internal notes and customer-visible updates separate in the data model and UI.
- Treat Reverb as a delivery mechanism; persisted server data remains authoritative.

### 16.4 Frontend/backend boundary

The React interface must not be the authority for permissions, payment state, booking confirmation, quote acceptance, appointment slot exclusivity or case history. The Laravel application is responsible for authentication, authorization, validation and business transitions, as defined by the SDD.

---

## 17. Design deliverables checklist

| Deliverable | Definition of done |
|---|---|
| User journey maps | All critical customer, provider and admin journeys above represented in a visual journey map |
| Information architecture | Public, customer, provider and admin route trees reviewed |
| Navigation | Desktop and mobile navigation states designed for each experience area |
| Wireframes | Low-fidelity wireframes for all priority screens in Sections 7 and 8 |
| High-fidelity UI | Public home, service discovery/detail, customer case, provider workspace, flight search/results and admin queue designed |
| Design system | Color, type, spacing, buttons, forms, badges, cards, tables, timeline and messaging components documented |
| Responsive layouts | Desktop, tablet and mobile layouts for priority screens |
| UI states | Loading, empty, error, success, pending, permission-denied and unavailable states covered |
| Interactive prototype | Critical journeys in Section 14 connected and testable |
| Developer handoff | Component behaviour, responsive rules, content rules and unresolved decisions documented |

---

## 18. Open decisions that affect the design

These are not design choices to guess. They must be resolved by the product owner before the affected screens are finalized.

| Decision | Design impact |
|---|---|
| Initial launch country/market | Legal copy, provider details, payment display and localization |
| Initial language and currency | Formatting, form labels and payment summaries |
| Fixed-price, quote-based or both | Service detail and purchase flow |
| Guest inquiry allowed? | Public inquiry entry and authentication timing |
| Payment collector/merchant of record | Checkout and payment confirmation language |
| Commission and provider payout model | Provider financial screens, if any |
| Flutterwave or Paystack selection | Payment method/redirect/error details |
| Duffel booking payment arrangement | Flight checkout and payment responsibility |
| Hotel provider and API | Hotel search, results, details and booking flow |
| Hotel payment/confirmation process | Hotel booking status and checkout |
| Customer document types and retention/access rules | Upload controls and attachment previews |
| Provider verification checks | Verification label and profile content |
| Refund/cancellation/no-show/dispute policies | Customer and provider actions and confirmation copy |
| Provider-assisted booking account model | Booking ownership and payment display |
| Appointment automatic confirmation or provider approval | Booking confirmation states |
| Meeting-link provider | Appointment detail and meeting action |
| Free consultation limits | Availability and booking validation copy |
| Initial workflow templates | Provider case setup and customer tracker examples |
| Provider team roles/permission matrix | Team/access screen |
| React/Laravel integration mode | Routing, shell and navigation implementation |
| Email provider | Notification preference and delivery-error details |

---

## 19. MVP design priority

To avoid designing every module at equal depth, use the SDD's implementation sequence and the PRD's MVP scope.

### Priority 1 — Marketplace foundation

- Public home and navigation.
- Customer authentication.
- Provider application and approval states.
- Provider profile and service listing.
- Service search/results/detail.
- Inquiry, quote and service order.
- Customer and provider overview.
- Case tracker and case-linked messaging.
- Consultation types, availability and appointments.
- Notifications and initial admin/support queues.

### Priority 2 — Service transactions and operations

- Payment/checkout states after gateway and commercial decisions.
- Refund and cancellation experiences after policy decisions.
- Reviews and moderation.
- Attachments only after secure document rules are approved.
- Provider team access after permission matrix is approved.

### Priority 3 — Travel booking

- Flight search/results/details and booking with Duffel.
- Customer direct booking.
- Provider-assisted flight booking.
- Hotel experience only after a hotel provider and payment/confirmation process are selected.

---

## 20. Design acceptance criteria

The design specification is ready for frontend implementation when:

1. Every screen maps to a PRD capability or an explicitly identified design proposal.
2. Public, customer, provider and admin experiences have distinct navigation and access boundaries.
3. All critical journeys have a start, primary action, expected state and completion/exception state.
4. The service detail page communicates scope, pricing method, prerequisites, estimates and terms.
5. The case tracker distinguishes provider progress from external decisions and customer actions.
6. Internal provider notes cannot be mistaken for customer-visible updates.
7. Appointment timezones and confirmation states are clear.
8. Flight booking states never imply confirmation before Duffel confirms.
9. Hotel screens do not imply live availability before a provider is selected.
10. Loading, empty, error, success and permission states are designed.
11. Responsive behaviour is specified for desktop, tablet and mobile.
12. Proposed visual tokens are identified as proposals, not source requirements.
13. Open decisions are visible and not silently resolved by design.
14. No unsupported feature, statistic, promise, integration or business rule has been introduced.

---

## 21. Source traceability

| Design area | PRD source | SDD source |
|---|---|---|
| Roles and access | §5, §8.1, §8.12 | §2.2, §2.3, §3 |
| Public discovery and profiles | §8.2–8.3 | §2.3, marketplace modules |
| Inquiries, quotes and orders | §7.1, §8.4 | §2.4 A, §5 |
| Consultations | §7.2, §8.5 | §2.4 B, §5 |
| Case tracker | §7.3, §8.6 | §2.4 C, §5 |
| Messaging | §8.7 | §2.3 messaging module, §3.5 |
| Flight booking | §7.4–7.5, §8.8 | §2.4 D, §9.2 |
| Hotel booking boundary | §8.8 | §2.4 E, §10 |
| Payments and status | §8.9 | §2.4, §5, §10 |
| Notifications | §8.10 | §3.2, §11 |
| Reviews | §8.11 | §2.3 reviews module |
| Admin/support | §8.12 | §2.2, §2.3 administration module |
| Responsive and accessibility | §9 | §12 |
| Trust and safety | §10 | §2.1, §5 |
| Technical/frontend boundary | — | §3.1–3.5, §13 |

---

## 22. Reference links

- Product Requirements Document: https://github.com/dev-adeshina/leappoint/blob/main/docs/PRD/PRD.md
- Software Design Document: https://github.com/dev-adeshina/leappoint/blob/main/docs/SDD/travel-sdd.md
- Fast Flight — Premium Flight Booking Web Design: https://dribbble.com/shots/25501705-Fast-Flight-Premium-Flight-Booking-Web-Design
- Flight Booking — Web App: https://dribbble.com/shots/22091492-Flight-Booking-Web-App
- Modern E-Commerce Admin Dashboard — Live Monitor: https://dribbble.com/shots/27526914-Modern-E-Commerce-Admin-Dashboard-UIUX-Design-Live-Monitor

