# Software Design Document (SDD)

## Travel Services Marketplace

**Document status:** Draft --- derived from PRD v1.0\
**Architecture:** Laravel monolith + React frontend + MySQL\
**Integrations:** Duffel; Flutterwave or Paystack (selection pending);
Laravel Reverb\
**Hosting direction:** Economical VPS recommended\
**Requirements source:** Uploaded `PRD.md`

------------------------------------------------------------------------

# 1. Purpose and design approach

This document translates the supplied Product Requirements Document into
a practical software design. It does not add product capabilities that
are absent from the PRD.

The design follows these principles:

1.  One Laravel application is the backend and application boundary.
2.  React is the browser frontend.
3.  MySQL is the system's primary and only application database.
4.  Laravel's standard queue infrastructure is used for asynchronous
    work.
5.  Laravel Reverb provides WebSocket communication.
6.  Duffel is the flight-booking integration.
7.  Flutterwave or Paystack handles eligible marketplace payments; the
    final provider remains an open decision.
8.  The hotel-booking provider remains unspecified. Duffel must not be
    assumed to provide hotel functionality.
9.  Keep modules inside the monolith. Do not introduce microservices, a
    separate Node.js backend, or another database.
10. Build only what the PRD requires. Resolve the PRD's open business
    decisions before implementing the affected workflows.

## 1.1 Requirement and design notation

-   **PRD requirement:** Explicitly stated in the source PRD.
-   **Design proposal:** A technical structure proposed to implement a
    PRD requirement.
-   **Open decision:** The PRD does not provide enough information to
    make a final decision.
-   **Out of scope:** Explicitly excluded from the MVP or not supported
    by the source requirements.

# 2. System analysis

## 2.1 Product description

The application is a multi-vendor travel services marketplace connecting
travelers with independent travel and visa-service providers. Customers
can discover providers, request or purchase services, book
consultations, communicate with providers, and track service cases.
Customers can also search and book flights and hotels directly.
Providers can manage their customers and arrange travel bookings on
their behalf.

The platform facilitates discovery, communication, payments, bookings,
and progress updates. It does not make visa decisions or guarantee
outcomes.

## 2.2 Main actors

  -----------------------------------------------------------------------
  Actor                               Responsibilities
  ----------------------------------- -----------------------------------
  Visitor                             Browse publicly available providers
                                      and services; registration/contact
                                      capabilities remain subject to the
                                      PRD's open decision about guest
                                      inquiries.

  Customer                            Manage account, discover providers,
                                      inquire, accept quotes, purchase
                                      services, book consultations,
                                      communicate, submit requested
                                      information, track cases, and make
                                      travel bookings.

  Provider owner                      Apply for approval, manage provider
                                      profile and services, handle
                                      inquiries, quotes, appointments,
                                      cases, messages, and client
                                      bookings; manage team access.

  Provider team member                Perform provider-account work
                                      allowed by permissions assigned by
                                      the provider owner.

  Platform administrator/support      Review providers and listings,
  agent                               manage users and orders, handle
                                      support/report/dispute work, and
                                      access conversations only for
                                      authorized support purposes.
  -----------------------------------------------------------------------

## 2.3 Business capabilities and modules

The following modules are logical areas within one Laravel application,
not independently deployed services.

  --------------------------------------------------------------------------------
  \#                      Module                  Main responsibilities
  ----------------------- ----------------------- --------------------------------
  1                       Identity and access     Registration, login, email
                                                  verification, password reset,
                                                  profile, roles, provider-team
                                                  permissions, account suspension.

  2                       Provider onboarding and Provider application, review,
                          profiles                approval/rejection/suspension,
                                                  public profile and verification
                                                  representation.

  3                       Service catalogue and   Service categories, provider
                          discovery               listings, service details,
                                                  search and filters.

  4                       Inquiries, quotes and   Customer inquiries, provider
                          service orders          responses, quotes, quote
                                                  acceptance/decline, order
                                                  creation and status history.

  5                       Consultations and       Consultation types, provider
                          appointments            availability, appointment
                                                  booking, payment dependency,
                                                  cancellation/rescheduling and
                                                  attendance status.

  6                       Case and progress       Service cases/orders, workflow
                          management              templates, case-specific
                                                  milestones, customer actions,
                                                  provider updates and internal
                                                  notes.

  7                       Messaging               Case-linked conversations,
                                                  messages, unread indicators,
                                                  limited attachments if document
                                                  rules are approved, support
                                                  reporting.

  8                       Travel booking          Flight search/booking through
                                                  Duffel; hotel workflow only
                                                  after a hotel provider is
                                                  selected; customer and
                                                  provider-assisted booking.

  9                       Payments and financial  Payment records, payment status,
                          records                 refunds, commissions and payout
                                                  records/rules where the
                                                  commercial model is defined.

  10                      Notifications           In-app and email notifications
                                                  for the events listed in the
                                                  PRD.

  11                      Reviews                 Eligible transaction reviews,
                                                  provider responses and
                                                  moderation/reporting.

  12                      Administration and      Provider/listing/user/order
                          support                 oversight, support, disputes,
                                                  categories, workflow templates
                                                  and audit review.

  13                      Audit                   Record important administrative,
                                                  account, order, payment and
                                                  message-access actions.
  --------------------------------------------------------------------------------

## 2.4 Main business workflows

### A. Service purchase

1.  Customer searches service listings.
2.  Customer opens a provider profile and service listing.
3.  Customer sends an inquiry, requests a quote, or purchases a listed
    service.
4.  Provider responds and may request information.
5.  Where a quote is used, the customer accepts or declines it.
6.  If payment is required, the customer pays through the configured
    payment gateway.
7.  The service order is managed by the provider.
8.  Provider updates the case milestones and customer actions.
9.  Customer and provider communicate through the linked conversation.
10. Provider marks the service complete; eligible review can follow.

The PRD allows different order status names and transitions by service
type. The implementation should therefore keep the status transition
rules explicit rather than assume every order follows an identical
linear path.

### B. Consultation

1.  Provider configures a consultation type and availability.
2.  Customer chooses a consultation and time slot.
3.  Customer pays if the consultation is paid.
4.  Appointment is confirmed according to the provider's approved
    confirmation configuration.
5.  Customer and provider receive notifications/reminders.
6.  Provider records completion, cancellation, or no-show.

The system must prevent double-booking. Free consultations must not
require payment. Paid consultations must not be confirmed before
successful payment unless the approved configuration allows it.

### C. Case progress

1.  Provider creates or manages a service order/case.
2.  Provider selects a workflow template.
3.  Provider adapts milestones for that individual case.
4.  Provider updates milestone state and customer-facing notes.
5.  Customer sees completed, current and upcoming milestones and
    required actions.
6.  The system records who changed a milestone and when.
7.  Provider-internal notes remain private.

A case-specific change must not modify the reusable workflow template or
another customer's case.

### D. Flight booking

1.  Customer or provider enters search criteria.
2.  Laravel sends the search request to Duffel.
3.  Laravel returns available offers to React.
4.  Customer/provider selects an offer and enters required traveler
    details.
5.  The booking request is submitted through the integration.
6.  The application records the external booking reference and status.
7.  The booking is not shown as confirmed until the booking provider
    confirms it.

Payment handling for flight bookings must follow the final
Duffel/commercial arrangement. Do not assume that the marketplace
payment gateway is necessarily used for every flight transaction.

### E. Hotel booking

The PRD requires hotel search and booking, but the supplied technology
decisions name Duffel only. The design reserves a travel-integration
boundary for hotel booking, but does not select or invent a hotel
provider. Hotel implementation is blocked until the provider, API
capabilities, payment flow, and booking confirmation process are
decided.

# 3. System architecture

## 3.1 Architecture decision

Use a **modular monolithic architecture**:

-   One Laravel codebase and deployment.
-   React frontend served by the same application/origin for the
    simplest initial deployment.
-   Laravel controllers/routes receive browser requests.
-   Laravel application services implement business operations.
-   Laravel models and MySQL persist application data.
-   Laravel queues handle asynchronous jobs using Laravel-supported
    queue drivers.
-   Laravel Reverb provides WebSocket events.
-   Laravel notification/mail facilities deliver email and in-app
    notifications.
-   Integration adapters isolate Duffel and the selected payment
    gateway.

"Modular" means clear code organization and responsibility boundaries
within the monolith. It does not mean separately deployed services.

## 3.2 Logical architecture diagram

``` mermaid
flowchart TB
    U[Customer / Provider / Admin Browser]
    R[React Frontend]
    L[Laravel Monolith]
    AUTH[Authentication and Authorization]
    MOD[Business Modules]
    DB[(MySQL)]
    Q[Laravel Queue]
    W[Laravel Queue Worker]
    REV[Laravel Reverb]
    MAIL[Email Delivery]
    DUFFEL[Duffel Flight API]
    PAY[Flutterwave OR Paystack]
    HOTEL[Hotel Provider - Not Selected]

    U --> R
    R --> L
    L --> AUTH
    L --> MOD
    MOD --> DB
    MOD --> Q
    Q --> W
    W --> DB
    W --> MAIL
    L <--> REV
    MOD <--> DUFFEL
    MOD <--> PAY
    MOD -. pending provider decision .-> HOTEL
```

## 3.3 Application layers

  -----------------------------------------------------------------------
  Layer                               Responsibility
  ----------------------------------- -----------------------------------
  React presentation                  Pages, forms, dashboards, search
                                      results, status displays and
                                      real-time UI updates.

  Laravel routes/controllers          HTTP entry points, request
                                      validation, authentication
                                      middleware and response formatting.

  Application/business services       Execute use cases such as creating
                                      an inquiry, accepting a quote,
                                      booking an appointment or creating
                                      a booking.

  Authorization policies              Enforce customer, provider-team and
                                      administrator access to each
                                      record/action.

  Eloquent models                     Represent persisted entities and
                                      their relationships.

  MySQL                               Primary relational data store.

  Queue jobs                          Perform asynchronous work such as
                                      notifications and other explicitly
                                      approved background tasks.

  Reverb events                       Publish real-time message and
                                      status-update events to authorized
                                      clients.

  Integration adapters                Communicate with Duffel and the
                                      selected payment gateway.
  -----------------------------------------------------------------------

Controllers should remain thin: validate input, authorize the action,
invoke the relevant application operation, and return a response.
Business rules should not be duplicated across controllers.

## 3.4 Suggested Laravel code organization

Keep the code in the existing Laravel application. Organize by business
responsibility, for example:

``` text
app/
  Models/
  Http/
    Controllers/
    Requests/
    Resources/
  Policies/
  Services/
    Identity/
    Providers/
    Marketplace/
    Orders/
    Appointments/
    Cases/
    Messaging/
    Travel/
    Payments/
    Notifications/
    Administration/
  Jobs/
  Events/
  Listeners/
  Notifications/
```

This is an organizational proposal, not a requirement to create a class
for every folder or to introduce a complex framework inside Laravel.

React should be organized around the same business areas (accounts,
provider, marketplace, orders, appointments, cases, messages, travel
bookings and administration). The exact React build/integration method
is an implementation decision; the PRD does not specify Inertia, a
separate SPA deployment, or a particular React routing library.

## 3.5 Request and event flow

**Synchronous request:** React → Laravel route/controller →
authorization and validation → business operation → Eloquent/MySQL →
response to React.

**Asynchronous task:** Laravel business operation → dispatch Laravel job
→ configured Laravel queue → Laravel worker → external email/provider or
database update → completion/failure handling.

**Real-time message:** Laravel validates and stores message → Laravel
broadcasts authorized event through Reverb → subscribed participant
clients update their message view. Persist the message in MySQL;
WebSocket delivery is not the permanent message store.

# 4. Hosting and deployment design

## 4.1 Recommended option: one economical VPS

A VPS is the simplest fit for the requested stack because the
application needs long-running Laravel queue workers and a persistent
Reverb WebSocket process, in addition to the web application.

The VPS can run:

-   Laravel application and React production assets.
-   Web server and PHP runtime.
-   MySQL.
-   Laravel queue worker process.
-   Laravel Reverb process.
-   Laravel scheduler invocation.
-   TLS termination and application configuration.

This keeps the initial infrastructure consolidated. The VPS must be
sized for actual traffic and database/worker load; no exact server size
or monthly cost can be responsibly specified without usage and provider
pricing information.

## 4.2 Vercel consideration

Vercel is not the recommended primary host for this Laravel monolith. It
is primarily a frontend deployment platform and does not remove the need
to host the Laravel application, MySQL, queue workers and persistent
Reverb service elsewhere.

A split deployment (React on Vercel and Laravel plus MySQL/worker/Reverb
on a VPS) is possible, but introduces additional deployment, domain,
CORS, cookie/session and operational configuration. It is not the
simplest or necessarily the cheapest starting point.

**Decision recommendation:** Deploy the complete application on one VPS
initially. Reassess only when real usage or operational requirements
justify separation.

## 4.3 Queue standards

Use Laravel's supported queue system only:

-   Configure the queue connection through Laravel
    configuration/environment.
-   Dispatch jobs using Laravel's queue APIs.
-   Run workers using Laravel's documented worker command/process
    management.
-   Configure retries, timeouts and failed-job handling per job type.
-   Use Laravel's failed-jobs facilities.
-   Make jobs safe against duplicate execution where a provider/payment
    callback or retry can repeat work.
-   Use Laravel scheduling for scheduled tasks, with the scheduler
    invoked by the host.
-   Do not introduce a separate queue framework or custom queue server.

The actual queue driver is an open deployment choice. The PRD does not
specify one. A database-backed Laravel queue is a simple starting option
if its workload is acceptable; Redis should not be added unless measured
workload or operational needs justify it.

# 5. Database design

## 5.1 Database principles

-   MySQL is the only application database.
-   Use relational tables and foreign keys for core relationships.
-   Use appropriate unique constraints and indexes.
-   Store monetary amounts with fixed-precision decimal columns,
    together with currency.
-   Store timestamps consistently; retain the provider's time zone for
    appointment display and scheduling.
-   Store external provider identifiers separately from internal primary
    keys.
-   Store status history where the PRD requires visible status history
    or auditability.
-   Do not store payment card details.
-   Do not store sensitive uploaded documents until access, storage,
    retention and deletion rules are approved.
-   Use soft deletion only where restoration/history requirements
    justify it; do not apply it indiscriminately.

## 5.2 Proposed logical schema

The following is a proposed relational structure derived from the PRD's
core data objects. It is not a claim that every field or business rule
has already been approved. Fields marked as policy-dependent require a
decision before implementation.

### A. Identity and provider accounts

  --------------------------------------------------------------------------------
  Table                         Important columns          Purpose
  ----------------------------- -------------------------- -----------------------
  `users`                       `id`, `name`, `email`      All authenticated
                                (unique), `password`,      people.
                                `email_verified_at`,       
                                `status`, timestamps       

  `user_profiles`               `id`, `user_id` (unique    Basic account profile.
                                FK), profile/contact       
                                fields, notification       
                                preferences                

  `provider_accounts`           `id`, `owner_user_id`      Provider
                                (FK), `provider_type`,     business/individual
                                `status`, timestamps       account and approval
                                                           state.

  `provider_applications`       `id`,                      Provider application
                                `provider_account_id`      and review.
                                (FK), submitted details,   
                                review status,             
                                reviewed_by, reviewed_at   

  `provider_profiles`           `id`,                      Public provider
                                `provider_account_id`      profile.
                                (unique FK), public        
                                description, operating     
                                details, visibility        

  `provider_members`            `id`,                      Team membership.
                                `provider_account_id`      
                                (FK), `user_id` (FK),      
                                status                     

  `provider_roles`              `id`,                      Roles within a provider
                                `provider_account_id`      account.
                                (FK), name                 

  `provider_member_roles`       `provider_member_id` (FK), Role assignment.
                                `provider_role_id` (FK)    

  `provider_permissions`        `id`, permission key,      Permission definitions,
                                description                if granular permissions
                                                           are implemented.

  `provider_role_permissions`   `provider_role_id` (FK),   Permissions assigned to
                                `provider_permission_id`   provider roles.
                                (FK)                       
  --------------------------------------------------------------------------------

The PRD requires provider owners to invite team members and assign
roles, but it does not define the exact role catalogue or permission
matrix. Those must be agreed before implementation. Platform
administrator/support authorization is separate from provider-team
authorization.

### B. Service catalogue and discovery

  ----------------------------------------------------------------------------------
  Table                            Important columns       Purpose
  -------------------------------- ----------------------- -------------------------
  `service_categories`             `id`, `parent_id`       Service categories.
                                   (nullable self-FK),     
                                   `name`, `status`        

  `service_listings`               `id`,                   Provider-published
                                   `provider_account_id`   services.
                                   (FK),                   
                                   `service_category_id`   
                                   (FK), `name`,           
                                   `description`,          
                                   `pricing_method`,       
                                   `price` (nullable),     
                                   `currency` (nullable),  
                                   `status`                

  `service_listing_destinations`   `id`,                   Supported
                                   `service_listing_id`    locations/destinations.
                                   (FK), destination value 

  `service_listing_languages`      `id`,                   Languages offered for a
                                   `service_listing_id`    listing.
                                   (FK), language value    
  ----------------------------------------------------------------------------------

The PRD requires listing details such as deliverables, prerequisites,
estimated duration, required customer information, and
cancellation/refund terms. The final column structure should reflect the
chosen representation for these fields. Do not assume fixed-price-only
listings: the PRD explicitly permits fixed price, quote, or both.

### C. Inquiries, quotes and service orders

  --------------------------------------------------------------------------------
  Table                            Important columns       Purpose
  -------------------------------- ----------------------- -----------------------
  `inquiries`                      `id`,                   Initial
                                   `customer_user_id`      customer-provider
                                   (FK),                   inquiry.
                                   `provider_account_id`   
                                   (FK),                   
                                   `service_listing_id`    
                                   (nullable FK),          
                                   `status`, `subject`,    
                                   timestamps              

  `quotes`                         `id`, `inquiry_id`      Provider's commercial
                                   (FK),                   proposal.
                                   `provider_account_id`   
                                   (FK),                   
                                   `customer_user_id`      
                                   (FK), `amount`,         
                                   `currency`, `terms`,    
                                   `status`, `expires_at`  
                                   (nullable)              

  `service_orders`                 `id`, `order_number`    Purchased service and
                                   (unique),               case anchor.
                                   `customer_user_id`      
                                   (FK),                   
                                   `provider_account_id`   
                                   (FK),                   
                                   `service_listing_id`    
                                   (nullable FK),          
                                   `quote_id` (nullable    
                                   FK), `status`,          
                                   `amount`, `currency`,   
                                   timestamps              

  `service_order_status_history`   `id`,                   Customer-visible order
                                   `service_order_id`      status history and
                                   (FK), `status`,         trace.
                                   `changed_by` (FK),      
                                   `note` (nullable),      
                                   `created_at`            
  --------------------------------------------------------------------------------

A quote accepted by a customer can be associated with the resulting
service order. The PRD does not define whether quote revisions are
required; do not add a versioning model unless the business confirms it
is needed.

### D. Workflow templates and case milestones

  -------------------------------------------------------------------------------------
  Table                            Important columns            Purpose
  -------------------------------- ---------------------------- -----------------------
  `workflow_templates`             `id`, `provider_account_id`  Reusable service
                                   (nullable if                 workflow.
                                   platform-managed), `name`,   
                                   `service_category_id`        
                                   (nullable FK), `status`      

  `workflow_template_milestones`   `id`, `workflow_template_id` Template milestone
                                   (FK), `name`, `description`, definitions.
                                   `sequence`, `is_required`,   
                                   `responsible_party`          

  `case_milestones`                `id`, `service_order_id`     Per-order milestone
                                   (FK),                        instance.
                                   `template_milestone_id`      
                                   (nullable FK), `name`,       
                                   `description`, `sequence`,   
                                   `state`,                     
                                   `responsible_party`,         
                                   `due_at` (nullable),         
                                   `visibility`,                
                                   `customer_action_required`   

  `case_milestone_history`         `id`, `case_milestone_id`    Records milestone
                                   (FK), `old_state`,           changes.
                                   `new_state`, `changed_by`    
                                   (FK), `note` (nullable),     
                                   `created_at`                 

  `case_notes`                     `id`, `service_order_id`     Customer-visible notes
                                   (FK), `author_user_id` (FK), and private provider
                                   `body`, `visibility`,        notes, separated by
                                   timestamps                   visibility and
                                                                authorization.
  -------------------------------------------------------------------------------------

When a template is applied, create case-specific milestone rows.
Subsequent edits to those rows must not alter the template or other
orders. Customer-visible and internal notes must be filtered on the
server, not merely hidden in React.

### E. Consultations and appointments

  -------------------------------------------------------------------------------
  Table                          Important columns        Purpose
  ------------------------------ ------------------------ -----------------------
  `consultation_types`           `id`,                    Provider consultation
                                 `provider_account_id`    offerings.
                                 (FK), `title`,           
                                 `description`,           
                                 `is_free`,               
                                 `duration_minutes`,      
                                 `price` (nullable),      
                                 `currency` (nullable),   
                                 `meeting_format`,        
                                 `timezone`, `status`     

  `consultation_availability`    `id`,                    Provider availability.
                                 `consultation_type_id`   
                                 (FK), availability       
                                 day/time or date range,  
                                 timezone                 

  `appointments`                 `id`,                    Consultation booking.
                                 `appointment_number`     
                                 (unique),                
                                 `consultation_type_id`   
                                 (FK),                    
                                 `provider_account_id`    
                                 (FK), `customer_user_id` 
                                 (FK), `starts_at`,       
                                 `ends_at`, `timezone`,   
                                 `status`,                
                                 `meeting_details`        
                                 (nullable)               

  `appointment_status_history`   `id`, `appointment_id`   Appointment status
                                 (FK), `status`,          history.
                                 `changed_by` (FK),       
                                 `note` (nullable),       
                                 `created_at`             
  -------------------------------------------------------------------------------

The exact availability representation (weekly recurring availability,
date-specific availability, or both) is not fully prescribed by the PRD.
Confirm it before finalizing the schema. Prevent double-booking with a
transaction and appropriate locking/overlap validation; do not rely only
on a frontend availability check.

### F. Messaging and attachments

  -----------------------------------------------------------------------------
  Table                         Important columns       Purpose
  ----------------------------- ----------------------- -----------------------
  `conversations`               `id`, `context_type`,   Conversation linked to
                                `context_id`,           an inquiry, service
                                `created_at`            order, appointment or
                                                        booking.

  `conversation_participants`   `id`, `conversation_id` Authorized participants
                                (FK), `user_id` (FK),   and unread calculation.
                                `last_read_at`          
                                (nullable)              

  `messages`                    `id`, `conversation_id` Persisted text
                                (FK), `sender_user_id`  messages.
                                (FK), `body`,           
                                `created_at`            

  `attachments`                 `id`, `message_id`      Optional limited
                                (FK), storage           attachments after
                                reference, original     secure handling rules
                                filename, MIME type,    are approved.
                                size, uploaded_by (FK), 
                                status                  
  -----------------------------------------------------------------------------

The context relationship can be implemented using a polymorphic relation
or explicit nullable foreign keys. Choose one approach consistently. A
polymorphic relation is flexible but cannot have a normal database
foreign key to every possible context table; explicit context-specific
foreign keys provide stronger relational integrity. The simpler choice
should be made based on the final conversation ownership rules.

Do not enable sensitive document uploads until allowed file types, size
limits, private storage, access control, retention and deletion are
defined.

### G. Client records and travel bookings

  --------------------------------------------------------------------------
  Table                   Important columns          Purpose
  ----------------------- -------------------------- -----------------------
  `provider_clients`      `id`,                      Provider's client
                          `provider_account_id`      record for assisted
                          (FK), `user_id` (nullable  bookings.
                          FK), client details        

  `flight_searches`       `id`,                      Search record where
                          `requested_by_user_id`     retention is required
                          (FK),                      by the booking
                          `provider_account_id`      workflow.
                          (nullable FK), search      
                          criteria, external search  
                          reference (nullable),      
                          status, timestamps         

  `flight_bookings`       `id`, `booking_reference`  Flight booking and
                          (unique),                  external reference.
                          `customer_user_id`         
                          (nullable FK),             
                          `provider_account_id`      
                          (nullable FK),             
                          `provider_client_id`       
                          (nullable FK),             
                          `external_provider`,       
                          `external_booking_id`,     
                          `status`, `amount`         
                          (nullable), `currency`     
                          (nullable), timestamps     

  `hotel_searches`        `id`, requester/client     Hotel search record
                          association, search        once a hotel
                          criteria, external         integration is
                          reference (nullable),      selected.
                          status, timestamps         

  `hotel_bookings`        `id`, booking reference,   Hotel booking once a
                          customer/provider/client   hotel integration is
                          association, external      selected.
                          provider and booking IDs,  
                          status, amount/currency,   
                          timestamps                 
  --------------------------------------------------------------------------

The PRD requires search and booking capabilities but does not explicitly
require storing every search. The search tables above are proposed only
if search history or operational traceability is needed. They may be
omitted if searches are transient.

Passenger/guest details should be collected only to the extent required
by the external booking provider and booking flow. The PRD does not
specify a passenger profile feature or long-term retention policy.

### H. Payments, refunds, reviews, support and audit

  -----------------------------------------------------------------------
  Table                   Important columns       Purpose
  ----------------------- ----------------------- -----------------------
  `payments`              `id`, `user_id` (FK),   Payment record linked
                          `payable_type`,         to an eligible
                          `payable_id`,           order/appointment or
                          `gateway`,              other approved payable.
                          `gateway_reference`,    
                          `amount`, `currency`,   
                          `status`, timestamps    

  `refunds`               `id`, `payment_id`      Refund record.
                          (FK),                   
                          `gateway_reference`     
                          (nullable), `amount`,   
                          `currency`, `status`,   
                          reason, timestamps      

  `commission_records`    `id`,                   Commission accounting
                          `service_order_id`      record once commission
                          (nullable FK), payment  rules are defined.
                          reference, amount,      
                          currency, status        

  `payout_records`        `id`, provider account, Provider payout record
                          amount, currency,       if platform-managed
                          status, external        payouts are approved.
                          reference (nullable)    

  `reviews`               `id`, `user_id` (FK),   Eligible customer
                          reviewable_type,        review.
                          reviewable_id, rating,  
                          content, status,        
                          timestamps              

  `support_cases`         `id`,                   Support
                          `opened_by_user_id`     issue/report/dispute
                          (FK), related context   handling.
                          (nullable), status,     
                          subject, timestamps     

  `audit_events`          `id`, `actor_user_id`   Important security and
                          (nullable FK), action,  administrative audit
                          subject_type,           events.
                          subject_id, metadata,   
                          created_at              

  `notifications`         Laravel notification    In-app notification
                          storage fields          records using Laravel's
                                                  notification
                                                  facilities.
  -----------------------------------------------------------------------

Polymorphic payable/review/context relationships should be constrained
by application authorization and validation. Do not treat a polymorphic
ID as proof of access.

The PRD requires commissions and provider payouts to be configurable but
leaves the commercial rules open. The schema is a starting
representation, not an instruction to implement a particular
split-payment or marketplace-payout model. Confirm who collects funds,
who is merchant of record, payout timing, fees, refunds and disputes
before implementing financial settlement.

## 5.3 Main entity relationships

``` mermaid
erDiagram
    USERS ||--o| USER_PROFILES : has
    USERS ||--o{ PROVIDER_ACCOUNTS : owns
    PROVIDER_ACCOUNTS ||--o{ PROVIDER_MEMBERS : has
    USERS ||--o{ PROVIDER_MEMBERS : joins
    PROVIDER_ACCOUNTS ||--o{ SERVICE_LISTINGS : publishes
    SERVICE_CATEGORIES ||--o{ SERVICE_LISTINGS : classifies
    USERS ||--o{ INQUIRIES : creates
    PROVIDER_ACCOUNTS ||--o{ INQUIRIES : receives
    INQUIRIES ||--o{ QUOTES : receives
    USERS ||--o{ SERVICE_ORDERS : purchases
    PROVIDER_ACCOUNTS ||--o{ SERVICE_ORDERS : fulfils
    QUOTES ||--o| SERVICE_ORDERS : may_create
    SERVICE_ORDERS ||--o{ CASE_MILESTONES : contains
    WORKFLOW_TEMPLATES ||--o{ WORKFLOW_TEMPLATE_MILESTONES : defines
    SERVICE_ORDERS ||--o{ CONVERSATIONS : may_have
    CONVERSATIONS ||--o{ MESSAGES : contains
    USERS ||--o{ APPOINTMENTS : books
    CONSULTATION_TYPES ||--o{ APPOINTMENTS : receives
    USERS ||--o{ PAYMENTS : makes
    SERVICE_ORDERS ||--o{ REVIEWS : may_receive
    USERS ||--o{ AUDIT_EVENTS : performs
```

The diagram shows the principal relationships, not every supporting
table. A conversation may also be linked to an inquiry, appointment or
travel booking as specified in the PRD.

## 5.4 Important constraints and indexes

Recommended constraints include:

-   Unique user email.
-   Unique service order number, appointment number and booking
    reference.
-   Unique provider membership per provider account and user.
-   Foreign keys for ordinary parent-child relationships.
-   Indexes on provider status, listing category/status, order
    customer/provider/status, appointment provider/start time/status,
    conversation participants, message conversation/time, payment
    gateway reference and external booking identifiers.
-   Indexes on foreign keys and commonly filtered status/date columns.
-   Unique gateway reference where the gateway contract guarantees
    uniqueness.
-   Validate amount/currency and status transitions in the application.
-   Use database transactions for operations that must succeed or fail
    together, such as accepting a quote and creating its order, or
    reserving an appointment slot.

# 6. API requirements

## 6.1 API design principles

React communicates with Laravel through same-origin HTTP routes. Use
JSON request/response bodies for the application endpoints. Keep route
names and response shapes consistent.

-   Validate all input on the server.
-   Authenticate protected routes.
-   Authorize access to the specific record on every request.
-   Use pagination for list endpoints.
-   Return clear validation and authorization errors.
-   Do not expose internal notes, provider secrets, payment credentials
    or private documents.
-   Use idempotency protection for payment/booking creation and provider
    webhook processing where supported by the relevant provider.
-   Keep third-party API credentials on the server.

The route list below is a logical API contract proposal. Exact URL
prefixes, response envelopes and versioning conventions can be settled
during implementation without changing the business scope.

## 6.2 Authentication and account endpoints

  -----------------------------------------------------------------------------------------------
  Method            Endpoint                             Purpose           Access
  ----------------- ------------------------------------ ----------------- ----------------------
  POST              `/register`                          Register customer Public
                                                         account           

  POST              `/login`                             Authenticate      Public

  POST              `/logout`                            End session       Authenticated

  POST              `/email/verification-notification`   Resend            Authenticated
                                                         verification      

  GET               `/email/verify/{id}/{hash}`          Verify email      Signed/authenticated
                                                                           as appropriate

  POST              `/forgot-password`                   Request reset     Public

  POST              `/reset-password`                    Reset password    Public with valid
                                                                           token

  GET               `/api/me`                            Current           Authenticated
                                                         user/profile      

  PATCH             `/api/me`                            Update permitted  Authenticated
                                                         profile fields    
  -----------------------------------------------------------------------------------------------

Use Laravel's established authentication facilities. The exact
session/token mechanism depends on whether React is served same-origin
or separately. For the recommended same-origin deployment, a secure
Laravel session/cookie approach is the simpler starting design. Do not
introduce a separate identity service.

## 6.3 Provider endpoints

  ---------------------------------------------------------------------------------------------
  Method            Endpoint                               Purpose           Access
  ----------------- -------------------------------------- ----------------- ------------------
  POST              `/api/provider-applications`           Submit provider   Authenticated
                                                           application       

  GET               `/api/provider-applications/current`   View own          Applicant
                                                           application       

  GET               `/api/provider/profile`                View own provider Provider member
                                                           profile           

  PATCH             `/api/provider/profile`                Update permitted  Authorized
                                                           profile fields    provider member

  GET               `/api/provider/team`                   List provider     Provider
                                                           team              owner/authorized
                                                                             role

  POST              `/api/provider/team/invitations`       Invite team       Authorized
                                                           member            provider owner

  PATCH             `/api/provider/team/{member}`          Change permitted  Authorized
                                                           team access       provider owner

  GET               `/api/providers`                       Search public     Public
                                                           approved          
                                                           providers         

  GET               `/api/providers/{provider}`            View public       Public
                                                           provider profile  
  ---------------------------------------------------------------------------------------------

Administrative provider-approval endpoints are listed under
administration.

## 6.4 Service catalogue and discovery endpoints

  ----------------------------------------------------------------------------------------------------
  Method            Endpoint                                       Purpose           Access
  ----------------- ---------------------------------------------- ----------------- -----------------
  GET               `/api/service-categories`                      List active       Public
                                                                   categories        

  GET               `/api/services`                                Search/filter     Public
                                                                   published         
                                                                   services          

  GET               `/api/services/{service}`                      View service      Public
                                                                   details           

  POST              `/api/provider/services`                       Create listing    Authorized
                                                                                     provider

  PATCH             `/api/provider/services/{service}`             Update own        Authorized
                                                                   listing           provider

  POST              `/api/provider/services/{service}/publish`     Publish listing   Authorized
                                                                                     provider

  POST              `/api/provider/services/{service}/unpublish`   Unpublish listing Authorized
                                                                                     provider
  ----------------------------------------------------------------------------------------------------

Public discovery must return only approved/visible provider profiles and
published listings.

## 6.5 Inquiry, quote and service-order endpoints

  -----------------------------------------------------------------------------------------------
  Method            Endpoint                                Purpose           Access
  ----------------- --------------------------------------- ----------------- -------------------
  POST              `/api/inquiries`                        Create inquiry    Customer

  GET               `/api/inquiries`                        List accessible   Customer/provider
                                                            inquiries         

  GET               `/api/inquiries/{inquiry}`              View inquiry      Authorized
                                                                              participant

  POST              `/api/inquiries/{inquiry}/responses`    Provider response Authorized provider

  POST              `/api/inquiries/{inquiry}/quotes`       Submit quote      Authorized provider

  POST              `/api/quotes/{quote}/accept`            Accept quote      Quote customer

  POST              `/api/quotes/{quote}/decline`           Decline quote     Quote customer

  GET               `/api/service-orders`                   List accessible   Customer/provider
                                                            orders            

  GET               `/api/service-orders/{order}`           View order        Authorized
                                                                              participant

  GET               `/api/service-orders/{order}/history`   View status       Authorized
                                                            history           participant
  -----------------------------------------------------------------------------------------------

Quote acceptance and order creation should execute atomically when the
selected business process creates an order on acceptance. Prevent
duplicate order creation if the same acceptance request is retried.

## 6.6 Appointment endpoints

  -------------------------------------------------------------------------------------------------------
  Method            Endpoint                                         Purpose           Access
  ----------------- ------------------------------------------------ ----------------- ------------------
  GET               `/api/providers/{provider}/consultation-types`   List consultation Public
                                                                     offerings         

  GET               `/api/consultation-types/{type}/availability`    Retrieve          Public
                                                                     available slots   

  POST              `/api/appointments`                              Request/book      Customer
                                                                     appointment       

  GET               `/api/appointments`                              List own/provider Authorized user
                                                                     appointments      

  GET               `/api/appointments/{appointment}`                View appointment  Authorized
                                                                                       participant

  POST              `/api/appointments/{appointment}/cancel`         Request/apply     Authorized
                                                                     cancellation      participant,
                                                                                       policy-dependent

  POST              `/api/appointments/{appointment}/reschedule`     Request/apply     Authorized
                                                                     rescheduling      participant,
                                                                                       policy-dependent

  POST              `/api/provider/consultation-types`               Create            Authorized
                                                                     consultation type provider

  PATCH             `/api/provider/consultation-types/{type}`        Update            Authorized
                                                                     consultation type provider

  PATCH             `/api/provider/appointments/{appointment}`       Confirm/update    Authorized
                                                                     appointment       provider
  -------------------------------------------------------------------------------------------------------

The server must recheck slot availability at booking time and prevent
overlapping confirmed reservations. Payment must be completed before
confirmation for paid appointments unless the approved configuration
explicitly allows another sequence.

## 6.7 Case and milestone endpoints

  -------------------------------------------------------------------------------------------------------------------------
  Method            Endpoint                                                           Purpose            Access
  ----------------- ------------------------------------------------------------------ ------------------ -----------------
  GET               `/api/service-orders/{order}/milestones`                           View               Authorized
                                                                                       customer-visible   participant
                                                                                       milestones         

  POST              `/api/provider/service-orders/{order}/milestones`                  Add case milestone Authorized
                                                                                                          provider

  PATCH             `/api/provider/milestones/{milestone}`                             Update milestone   Authorized
                                                                                                          provider

  GET               `/api/provider/workflow-templates`                                 List templates     Authorized
                                                                                                          provider

  POST              `/api/provider/workflow-templates`                                 Create template    Authorized
                                                                                                          provider

  PATCH             `/api/provider/workflow-templates/{template}`                      Update template    Authorized
                                                                                                          provider

  POST              `/api/provider/service-orders/{order}/apply-workflow/{template}`   Apply template to  Authorized
                                                                                       case               provider
  -------------------------------------------------------------------------------------------------------------------------

Internal notes require a separate authorized endpoint or an explicit
server-side visibility policy. A customer request must never be able to
retrieve private notes by changing an ID or query parameter.

## 6.8 Messaging endpoints and real-time events

  ---------------------------------------------------------------------------------------------------------
  Method            Endpoint                                       Purpose                Access
  ----------------- ---------------------------------------------- ---------------------- -----------------
  GET               `/api/conversations`                           List accessible        Participant
                                                                   conversations          

  GET               `/api/conversations/{conversation}/messages`   Retrieve message       Participant
                                                                   history                

  POST              `/api/conversations/{conversation}/messages`   Send message           Participant

  POST              `/api/conversations/{conversation}/read`       Update read position   Participant

  POST              `/api/conversations/{conversation}/reports`    Report                 Authorized
                                                                   conversation/message   participant
  ---------------------------------------------------------------------------------------------------------

Reverb broadcasts should use private, authorized channels. A user must
be authorized as a conversation participant before subscribing. The
event should contain only the information needed to update the UI;
clients can retrieve the authoritative record through the API.

## 6.9 Travel endpoints

  -------------------------------------------------------------------------------------------
  Method            Endpoint                            Purpose           Access
  ----------------- ----------------------------------- ----------------- -------------------
  POST              `/api/flights/search`               Search flights    Authenticated
                                                        through Duffel    customer/provider

  POST              `/api/flights/bookings`             Create flight     Authorized
                                                        booking through   customer/provider
                                                        Duffel            

  GET               `/api/flights/bookings`             List accessible   Owner/authorized
                                                        flight bookings   provider

  GET               `/api/flights/bookings/{booking}`   View booking and  Owner/authorized
                                                        status            provider

  POST              `/api/hotels/search`                Hotel search      Not implementable
                                                        through provider  until provider
                                                        not yet selected  decision

  POST              `/api/hotels/bookings`              Hotel booking     Not implementable
                                                        through provider  until provider
                                                        not yet selected  decision

  GET               `/api/hotels/bookings`              List hotel        Authorized user,
                                                        bookings          after integration
  -------------------------------------------------------------------------------------------

For provider-assisted bookings, Laravel must verify that the provider
has access to the associated client record. The booking must retain both
the platform's internal reference and the external provider's booking
identifier.

## 6.10 Payment endpoints and callbacks

  ---------------------------------------------------------------------------------------------------
  Method            Endpoint                             Purpose            Access
  ----------------- ------------------------------------ ------------------ -------------------------
  POST              `/api/payments/initialize`           Start an eligible  Authenticated payer
                                                         payment            

  GET               `/api/payments/{payment}`            View own payment   Payer/authorized admin
                                                         status             

  POST              `/api/payments/{payment}/verify`     Verify payment     Payer/server-controlled
                                                         with gateway where 
                                                         applicable         

  POST              `/api/webhooks/payments/{gateway}`   Receive gateway    Provider signature
                                                         callback/webhook   verification
  ---------------------------------------------------------------------------------------------------

The callback endpoint must verify the gateway signature/authenticity
according to the selected provider's current documentation. Do not trust
a browser redirect as proof of payment. Verify amount, currency,
transaction reference and associated payable before marking a payment
successful.

## 6.11 Administration and support endpoints

  --------------------------------------------------------------------------------------------------------
  Method            Endpoint                                           Purpose           Access
  ----------------- -------------------------------------------------- ----------------- -----------------
  GET               `/api/admin/provider-applications`                 List applications Admin

  PATCH             `/api/admin/provider-applications/{application}`   Approve/reject    Admin

  PATCH             `/api/admin/providers/{provider}`                  Suspend/manage    Admin
                                                                       provider          

  GET               `/api/admin/users`                                 Search users      Admin/support
                                                                                         permission

  GET               `/api/admin/service-orders`                        Search orders     Admin/support
                                                                                         permission

  GET               `/api/admin/payments`                              View payment      Finance/admin
                                                                       status            permission

  GET               `/api/admin/support-cases`                         Manage support    Authorized
                                                                       cases             support

  PATCH             `/api/admin/support-cases/{case}`                  Update support    Authorized
                                                                       case              support

  GET               `/api/admin/audit-events`                          Review audit      Authorized admin
                                                                       events            
  --------------------------------------------------------------------------------------------------------

Administrative conversation access must be limited to authorized support
purposes and audited.

# 7. Authentication and authorization

## 7.1 Authentication

Use Laravel authentication, password hashing, email verification,
password reset and session management.

For the recommended same-origin React/Laravel deployment:

-   Use secure, HTTP-only session cookies.
-   Enable HTTPS.
-   Apply CSRF protection to state-changing browser requests.
-   Configure secure cookie settings for production.
-   Regenerate the session after login and privilege changes.
-   Rate-limit login, password reset and verification endpoints.

If a separate React origin is chosen later, revisit the
session/CORS/CSRF design rather than casually switching to long-lived
browser-stored bearer tokens.

## 7.2 Authorization model

Use role-based access for broad capabilities and Laravel policies for
record-level authorization.

  -----------------------------------------------------------------------
  Role                                Access boundary
  ----------------------------------- -----------------------------------
  Customer                            Own profile, inquiries, quotes,
                                      orders, appointments,
                                      conversations, payments and
                                      bookings.

  Provider owner                      Own provider account, listings,
                                      customers/cases, appointments,
                                      conversations and associated
                                      bookings; manages team access
                                      within approved limits.

  Provider team member                Only provider-account resources and
                                      actions permitted by assigned role.

  Platform administrator              Platform-wide administrative
                                      actions explicitly granted.

  Support agent                       Support operations and narrowly
                                      scoped access to related
                                      customer/provider records.
  -----------------------------------------------------------------------

A role alone is not enough. Every request must also verify ownership,
provider-account membership, conversation participation, or the relevant
support authorization.

## 7.3 Required authorization checks

Examples:

-   A customer cannot view another customer's order by guessing its ID.
-   A provider cannot access another provider's inquiries, appointments
    or clients.
-   A provider team member cannot perform actions outside assigned
    permissions.
-   A provider cannot publish a profile before approval.
-   A customer cannot change provider-owned listing data.
-   A conversation is visible only to its authorized participants,
    except narrowly authorized and audited support access.
-   Internal notes are never returned to customers.
-   A provider-assisted booking is accessible only to the relevant
    provider and authorized customer/client context.
-   A payment status cannot be changed by a customer-submitted request.

# 8. Security architecture

## 8.1 Security boundaries

1.  **Public boundary:** Public provider profiles, published service
    listings and public discovery endpoints.
2.  **Authenticated customer boundary:** Customer-owned account and
    transaction records.
3.  **Provider tenant boundary:** Data belonging to a provider account
    and its authorized team.
4.  **Platform administration boundary:** Administrative operations,
    with separate permissions.
5.  **Integration boundary:** Server-to-server calls to Duffel and the
    selected payment gateway.
6.  **Realtime boundary:** Private Reverb channels authorized against
    application records.
7.  **File boundary:** Private attachment storage, only after
    file-handling rules are approved.

## 8.2 Data protection

-   Use HTTPS for all production traffic.
-   Hash passwords using Laravel's supported password hashing.
-   Keep API keys and secrets in environment/secret configuration, never
    in React source or repository.
-   Never store card numbers or CVV.
-   Limit customer data collection to what the service or booking
    requires.
-   Restrict access to travel/passenger details and any sensitive
    service information.
-   Keep private documents outside public web paths.
-   Validate file type and size; scan files if a suitable operational
    process is selected before enabling uploads.
-   Define document retention and deletion before enabling sensitive
    uploads.
-   Do not log passwords, payment secrets, full payment credentials or
    sensitive document contents.

## 8.3 Payment security

-   Initialize payments from Laravel, not directly from trusted browser
    input.
-   Store gateway transaction references and payment status.
-   Verify callbacks/webhooks using the gateway's documented signature
    mechanism.
-   Reconcile gateway reference, amount, currency and payable.
-   Make webhook processing idempotent.
-   Do not mark payment successful based only on a frontend redirect.
-   Record refunds and disputes and reflect their state in the relevant
    order/appointment.
-   Do not assume marketplace split payments or provider payouts until
    the commercial model and gateway capabilities are confirmed.

## 8.4 Duffel integration security and correctness

-   Make Duffel calls from Laravel only.
-   Store credentials server-side.
-   Validate all search and booking inputs.
-   Persist internal and external booking references.
-   Treat external booking status as authoritative for
    provider-confirmed booking state.
-   Do not display a booking as confirmed until Duffel confirms it.
-   Handle provider errors/timeouts without falsely reporting success.
-   Do not assume flight payment mechanics; confirm the selected Duffel
    booking/payment arrangement.

## 8.5 Audit and operational security

Audit important events specified in the PRD:

-   Account and access changes.
-   Provider approval, rejection and suspension.
-   Listing changes affecting publication.
-   Order and appointment status changes.
-   Payment/refund state changes.
-   Administrative actions.
-   Administrative access to customer conversations.
-   Milestone changes and responsible actor.

Apply least privilege to database credentials, VPS access, deployment
secrets and integration credentials. Maintain backups and test recovery
rather than assuming that a successful backup job proves recoverability.

# 9. Data flow diagrams

## 9.1 Service purchase and payment

``` mermaid
sequenceDiagram
    actor Customer
    participant React
    participant Laravel
    participant MySQL
    participant Gateway as Flutterwave/Paystack
    participant Queue as Laravel Queue

    Customer->>React: Select service / accept quote
    React->>Laravel: Submit request
    Laravel->>Laravel: Authenticate, authorize, validate
    Laravel->>MySQL: Create/update inquiry, quote or order
    Laravel-->>React: Return order/payment state
    React->>Laravel: Initialize payment
    Laravel->>Gateway: Create payment transaction
    Gateway-->>Laravel: Payment initialization result
    Laravel->>MySQL: Save gateway reference and pending status
    Laravel-->>React: Return payment instructions
    Gateway->>Laravel: Payment webhook/callback
    Laravel->>Laravel: Verify authenticity and transaction
    Laravel->>MySQL: Update payment status
    Laravel->>Queue: Dispatch notification job
    Laravel-->>Gateway: Acknowledge valid event
```

The exact order/payment relationship and settlement process depend on
the open commercial decisions.

## 9.2 Flight search and booking

``` mermaid
sequenceDiagram
    actor User
    participant React
    participant Laravel
    participant Duffel
    participant MySQL

    User->>React: Enter flight search
    React->>Laravel: Search request
    Laravel->>Laravel: Authenticate and validate
    Laravel->>Duffel: Search flight offers
    Duffel-->>Laravel: Offers and provider terms
    Laravel-->>React: Return offers
    User->>React: Select offer and submit traveler details
    React->>Laravel: Booking request
    Laravel->>Laravel: Authorize customer/provider-client access
    Laravel->>Duffel: Submit booking request
    Duffel-->>Laravel: Booking result/status
    Laravel->>MySQL: Store internal and external references/status
    Laravel-->>React: Return current booking status
```

A pending or failed external result must not be represented as a
confirmed booking.

## 9.3 Case progress and realtime update

``` mermaid
sequenceDiagram
    actor Provider
    actor Customer
    participant React
    participant Laravel
    participant MySQL
    participant Reverb
    participant Queue

    Provider->>React: Update case milestone
    React->>Laravel: Submit milestone change
    Laravel->>Laravel: Authorize provider and validate transition
    Laravel->>MySQL: Save milestone and history
    Laravel->>Reverb: Broadcast authorized update
    Reverb-->>React: Deliver event to authorized customer
    React-->>Customer: Refresh visible progress
    Laravel->>Queue: Dispatch notification
```

The database remains the authoritative record. Reverb is the delivery
mechanism for real-time updates.

## 9.4 Message flow

``` mermaid
sequenceDiagram
    actor Sender
    actor Recipient
    participant React
    participant Laravel
    participant MySQL
    participant Reverb

    Sender->>React: Send message
    React->>Laravel: POST message
    Laravel->>Laravel: Verify conversation membership
    Laravel->>MySQL: Persist message
    Laravel->>Reverb: Broadcast message event
    Reverb-->>React: Deliver to authorized participant channel
    React-->>Recipient: Display new message
```

# 10. Third-party integrations

  ------------------------------------------------------------------------------
  Integration       Purpose             Design boundary     Status
  ----------------- ------------------- ------------------- --------------------
  Duffel            Flight search and   Laravel server-side Selected by project
                    booking             adapter; external   owner; API workflow
                                        booking             and
                                        reference/status    commercial/payment
                                        stored locally      details to confirm

  Flutterwave or    Eligible service    One payment adapter Provider selection
  Paystack          and                 boundary; selected  pending
                    paid-consultation   gateway called from 
                    payments            Laravel             

  Laravel Reverb    WebSocket delivery  Laravel broadcasts  Selected
                                        events to           
                                        authorized private  
                                        channels            

  Email delivery    PRD email           Laravel             Provider not
  provider          notifications       mail/notification   specified
                                        facilities          

  Hotel booking     Hotel search and    Separate adapter    Not specified;
  provider          booking             boundary when       decision required
                                        provider is         
                                        selected            
  ------------------------------------------------------------------------------

Do not implement both Flutterwave and Paystack merely because both are
listed as options. Select one for the initial release unless there is a
documented business requirement for both.

# 11. Queue, notification and real-time design

## 11.1 Queue use

Use Laravel queues for work that does not need to block the user's HTTP
response, such as:

-   Sending email notifications.
-   Sending notification messages after order, appointment, payment or
    booking events.
-   Other provider calls that are explicitly designed as asynchronous
    workflows.

Do not move every operation to a queue. Operations that need an
immediate response (such as returning a validated search result) can
remain synchronous, subject to provider API response characteristics.

## 11.2 Reverb use

Use Reverb for:

-   New case-linked messages.
-   Unread/message updates.
-   Relevant case progress updates.
-   Other real-time status changes explicitly required by the PRD.

Reverb is not a database and does not replace persisted message history
or status history.

## 11.3 Notification channels

The PRD specifies email and in-app notifications. Use Laravel
notification facilities and the configured queue. Do not add SMS, push
notifications or other channels without a new requirement.

# 12. Non-functional requirements

  -----------------------------------------------------------------------
  Area                                Design requirement
  ----------------------------------- -----------------------------------
  Responsive UI                       React interface works on mobile,
                                      tablet and desktop.

  Performance                         Paginate lists and avoid
                                      unnecessary database queries; use
                                      indexes for common search/filter
                                      paths.

  Reliability                         Queue retries and failed-job
                                      handling; external API
                                      timeout/error handling.

  Consistency                         Use database transactions for
                                      multi-record business operations.

  Security                            HTTPS, authorization policies,
                                      private channels, server-side
                                      validation and least privilege.

  Auditability                        Record important account, order,
                                      payment, milestone and
                                      administrative actions.

  Recovery                            MySQL backups and documented
                                      restore process.

  Maintainability                     One Laravel codebase with clear
                                      business module boundaries.

  Extensibility                       Add service categories, providers
                                      and workflow templates without
                                      redesigning the application.

  Accessibility                       Meet the launch-market
                                      accessibility standard once
                                      selected.
  -----------------------------------------------------------------------

# 13. Technical decisions and constraints

  ------------------------------------------------------------------------
  Topic                   Decision                 Reason / constraint
  ----------------------- ------------------------ -----------------------
  Application             Laravel monolith         Explicit project
  architecture                                     constraint; simplest
                                                   deployment and
                                                   maintenance model.

  Frontend                React                    Explicit project
                                                   constraint.

  Database                MySQL                    Explicit project
                                                   constraint; no second
                                                   application database.

  Flight integration      Duffel                   Explicit project
                                                   constraint.

  Hotel integration       Not selected             PRD requires hotels but
                                                   no hotel provider is
                                                   named.

  Payments                Flutterwave or Paystack; Both are options; no
                          choose one               final selection or
                                                   commercial model
                                                   supplied.

  Realtime                Laravel Reverb           Explicit project
                                                   constraint.

  Queues                  Laravel queue standards  Explicit project
                                                   constraint; no separate
                                                   queue framework.

  Hosting                 One VPS recommended      Supports Laravel,
                                                   MySQL, workers and
                                                   persistent Reverb in
                                                   one economical initial
                                                   deployment.

  Vercel                  Not recommended as the   Laravel, MySQL, workers
                          complete application     and Reverb still need
                          host                     hosting elsewhere.

  Authentication          Laravel                  Avoids unnecessary
                          authentication/session   separate identity
                          for same-origin          infrastructure.
                          deployment               

  Authorization           Laravel policies +       Enforces role and
                          provider                 record ownership
                          roles/permissions        boundaries.

  File uploads            Deferred until rules are PRD makes secure
                          defined                  handling and retention
                                                   a prerequisite.

  Voice/video             Out of MVP               Explicit PRD exclusion.

  Mobile apps             Out of MVP               Explicit PRD exclusion.

  Microservices           Not used                 No requirement
                                                   justifies the added
                                                   operational complexity.
  ------------------------------------------------------------------------

# 14. Open decisions and implementation blockers

These questions come from the PRD or are necessary technical
clarifications. They should be resolved before the affected
functionality is built.

  ------------------------------------------------------------------------
  \#                      Decision required        Why it matters
  ----------------------- ------------------------ -----------------------
  1                       Initial launch           Provider checks, legal
                          country/market           terms, payments and
                                                   operational policy.

  2                       Initial currency and     Price storage/display,
                          language                 gateway configuration
                                                   and localization scope.

  3                       Fixed-price,             Listing and order
                          quote-based, or both     purchase flow.

  4                       Who collects             Payment ownership and
                          service/consultation     settlement
                          payments?                architecture.

  5                       Commission and provider  Financial records,
                          payout model             gateway capabilities
                                                   and operational
                                                   reconciliation.

  6                       Choose Flutterwave or    API, webhook, payment
                          Paystack                 verification and refund
                                                   implementation.

  7                       Duffel                   Flight booking payment
                          commercial/payment       and confirmation
                          arrangement              workflow.

  8                       Select hotel provider    Hotel search and
                                                   booking cannot be
                                                   implemented against an
                                                   unspecified API.

  9                       Hotel provider payment   Correct booking
                          and booking confirmation lifecycle and
                          process                  reconciliation.

  10                      Customer document types  Secure attachment
                          and retention/access     implementation.
                          rules                    

  11                      Provider verification    Approval workflow and
                          checks                   public verification
                                                   wording.

  12                      Refund, cancellation,    Order and appointment
                          no-show and dispute      transitions and
                          policies                 financial handling.

  13                      Can visitors inquire     Public routes and
                          without accounts?        identity requirements.

  14                      Provider booking         Duffel account and
                          commercial account vs    provider-assisted
                          customer account         booking permissions.

  15                      Appointment automatic    Slot reservation and
                          confirmation or provider payment sequence.
                          approval                 

  16                      Meeting-link provider    Online consultation
                                                   details.

  17                      Free consultation limits Booking validation
                                                   rules.

  18                      Initial workflow         Provider case setup and
                          templates                initial data.

  19                      Provider team roles and  Authorization
                          permission matrix        implementation.

  20                      React                    Same-origin
                          integration/deployment   Laravel-served React is
                          mode                     recommended; confirm
                                                   the chosen build
                                                   approach.

  21                      Email delivery provider  Production email
                                                   configuration and
                                                   delivery monitoring.

  22                      VPS provider, size and   Hosting budget and
                          backup policy            recovery expectations.
  ------------------------------------------------------------------------

# 15. Suggested implementation sequence

This sequence follows the PRD's phases while respecting the selected
architecture.

## Phase 1 --- Marketplace foundation

1.  Laravel application foundation, MySQL schema and React integration.
2.  Authentication and customer accounts.
3.  Provider application, review, approval and profile.
4.  Provider team membership and authorization.
5.  Service categories, listings and discovery.
6.  Inquiries, quotes and service orders.
7.  Consultation types, availability and appointments.
8.  Workflow templates and case milestones.
9.  Case-linked text messaging and Reverb updates.
10. Email and in-app notifications.
11. Initial admin/support tools and audit events.

## Phase 2 --- Service transactions and operations

1.  Select and integrate Flutterwave or Paystack.
2.  Implement service and paid-consultation payments.
3.  Implement confirmed commission/refund/payout rules.
4.  Reviews and moderation.
5.  Expanded workflow templates and case reporting.
6.  Secure attachments only after document rules are approved.

## Phase 3 --- Travel booking

1.  Implement Duffel flight search.
2.  Implement flight booking and external status synchronization.
3.  Implement customer direct booking.
4.  Implement provider-assisted booking and client association.
5.  Select a hotel provider.
6.  Implement hotel search and booking after provider selection.
7.  Implement booking support and cancellation information within the
    selected provider's capabilities.

Voice/video calling remains a later evaluation, as stated in the PRD.

# 16. Acceptance criteria for the design

The design is ready to guide implementation when:

-   The Laravel monolith and React integration approach are agreed.
-   MySQL is confirmed as the sole application database.
-   Module ownership and core entity relationships are accepted.
-   Customer, provider, provider-team and administrator access
    boundaries are defined.
-   Service order, quote, appointment and case lifecycle rules are
    agreed.
-   Payment gateway and commercial settlement rules are selected.
-   Duffel flight booking and payment responsibilities are confirmed.
-   A hotel provider is selected before hotel integration work begins.
-   Document handling and retention rules are approved before sensitive
    uploads.
-   Queue, scheduler, Reverb and VPS deployment responsibilities are
    documented.
-   Required API operations and authorization rules are accepted.

# 17. Final architecture recommendation

Build one Laravel monolithic application with React as its frontend and
MySQL as its only database. Keep the business modules inside the Laravel
codebase. Use Laravel authentication, policies, notifications, queues,
scheduler and Reverb. Integrate Duffel from the Laravel server side for
flights. Select either Flutterwave or Paystack for eligible marketplace
payments. Do not implement hotel booking until a hotel integration
provider is selected.

Deploy the initial system on one economical VPS so the web application,
MySQL, Laravel queue worker, scheduler and Reverb can run together. This
is the most direct fit for the requested technology and avoids the extra
operational complexity of splitting the frontend and backend across
hosting platforms.

The architecture should remain deliberately simple: one application, one
database, Laravel-managed background processing, and clearly bounded
external integrations.
