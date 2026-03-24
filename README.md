# MarineOS_open_vessel_PMS
Project of open sourse PMS for vessels
# MarineOS

**Open-core platform for ship management — built for the 21st century**

> AMOS costs $80k/year and was designed before smartphones existed.
> ShipNet requires a dedicated IT team to install.
> Most mid-size operators run their fleet on Excel and WhatsApp.
>
> MarineOS is the alternative.

---

## What is MarineOS?

MarineOS is a modular, web-based ship management platform for shipping companies operating 10–50 vessels. It covers the full operational lifecycle of a vessel — from daily engine room logs to drydock repairs to crew certification tracking — in a single coherent system.

**Core principle:** Every module works standalone. All modules work better together.

The codebase is open source. A managed cloud version will be offered commercially. You can self-host indefinitely for free.

---

## The problem with maritime software today

The market leaders (AMOS, ShipNet, Sertica, STAR) share the same problems:

- **Expensive** — $30k–$100k/year for a mid-size fleet
- **Old** — desktop-first UIs designed in the 2000s, bolted onto the web
- **Rigid** — you pay for the whole suite whether you need it or not
- **Opaque** — shipowners and managers see reports, not live data
- **Closed** — no API, no integrations, no customization without the vendor

Mid-size operators are stuck: they can't afford enterprise systems, but outgrow spreadsheets quickly. MarineOS is built specifically for this gap.

---

## Modules

MarineOS is a modular monolith. Each module is independently useful but shares a common data model, authentication layer, and UI shell.

### v0.1 — MVP

#### `pms` — Planned Maintenance System

The technical heart of any ship management operation.

- **Equipment registry** — full inventory of machinery and systems per vessel, with maker, model, serial number, location, and running hours
- **Maintenance schedules** — jobs defined by interval (calendar days, running hours, nautical miles) with automatic due-date calculation
- **Job execution** — engineers record completed maintenance with findings, parts used, man-hours, and sign-off
- **Daily parameters** — chief engineer enters watchkeeping data manually: RPM, temperatures, pressures, fuel consumption, running hours per machinery. Data is stored, trended, and alerted against defined limits
- **Overdue tracking** — dashboard showing all overdue and upcoming jobs fleet-wide
- **Class integration** — flag jobs required by classification society (DNV, Lloyd's, BV) and track their survey status

#### `drydock` — Drydock Management

Full transparency for ship repair in drydock. *

- **Work order cards** — superintendent creates repair cards with equipment details, scope, drawings, photos, deadline, and budget
- **Contractor sharing** — public tokenized link lets any contractor see the full work scope and submit a quote without creating an account
- **Budget tracking** — supply department sets planned budget; actual cost updated on completion
- **Owner dashboard** — shipowner sees every repair, contractor, cost, and photo in real time — no calls, no waiting for reports
- **Completion documentation** — before/after photos attached to each work card
- **PDF report** — full drydock completion report generated automatically

---

### v0.2

#### `inventory` — Inventory & Procurement

- Spare parts catalog with maker references and vessel-specific stock levels
- Requisition workflow: vessel requests → office approval → purchase order
- Supplier database with price history
- Auto-requisition trigger from PMS when a maintenance job consumes parts
- Stock alerts when critical spares fall below minimum levels
- Delivery tracking and goods receipt

#### `crew` — Crew Management

- Seafarer registry with full personal and professional profile
- Certificate tracking — STCW, flag state, medical — with expiry alerts
- Voyage planning: sign-on/sign-off scheduling by vessel and rank
- Contract management and payroll history
- Crew change coordination with port agents
- Vaccination records and medical fitness tracking

#### `safety` — Safety, Risk & ISM/SMS

- Risk register — hazard identification, consequence, mitigation, residual risk scoring
- Incident and near-miss reporting with investigation workflow
- Drill records — lifeboat, fire, abandon ship, SOPEP — with attendance log
- Inspection management — PSC, flag state, vetting, internal audits — with finding tracker
- Non-conformity (NC) management with root cause and corrective action
- Checklist library (departure, arrival, bunkering, cargo, hot work, etc.)

#### `analytics` — Reports & Dashboards

- Fleet KPI dashboard — vessel availability, maintenance compliance rate, overdue jobs, cost per vessel
- Shipowner view — clean read-only dashboard summarizing fleet status across all modules
- Cross-vessel comparison within the fleet
- Trend charts for daily parameters (engine performance, fuel consumption)
- Scheduled reports delivered by email (weekly, monthly)
- Export: PDF, Excel, REST API

---

### v1.0

#### `voyage` — Voyage & Operations
Port calls, cargo, fuel consumption reporting, CII/EEXI carbon intensity tracking, laytime and demurrage calculation.

#### `finance` — Budgeting & Finance
Vessel OPEX budget, invoice management, cost allocation by department, P&L reporting per vessel and fleet.

#### `class` — Class & Statutory Certificates
Registry of all vessel certificates with survey dates, renewal tracking, and planned integrations with DNV Veracity and Lloyd's Register APIs.

---

## Architecture

MarineOS is a **modular monolith** — one application, one deployment, clearly separated internal modules. This is a deliberate choice over microservices: easier to develop, easier to self-host, easier to contribute to.

```
┌─────────────────────────────────────────────────────────────┐
│                        Web Client                           │
│           Next.js 14 · TypeScript · Tailwind CSS            │
│  ┌─────────────────────────┐  ┌──────────────────────────┐  │
│  │   Authenticated app     │  │  Public pages (no login) │  │
│  │   /app/...              │  │  /share/{token}          │  │
│  └─────────────────────────┘  └──────────────────────────┘  │
└───────────────────────┬─────────────────────────────────────┘
                        │  tRPC (type-safe, no codegen)
┌───────────────────────▼─────────────────────────────────────┐
│                    API + Business Logic                      │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                   core/                              │   │
│  │  auth · tenancy · vessel registry · notifications   │   │
│  │  file storage · audit log · billing · permissions   │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────┐   │
│  │  pms/   │ │drydock/ │ │inventory/│ │   crew/      │   │
│  └──────────┘ └──────────┘ └──────────┘ └──────────────┘   │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────┐   │
│  │ safety/ │ │analytics/│ │ voyage/ │ │  finance/    │   │
│  └──────────┘ └──────────┘ └──────────┘ └──────────────┘   │
└───────────────────────┬─────────────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────────────┐
│          PostgreSQL · Redis · S3-compatible storage          │
└─────────────────────────────────────────────────────────────┘
```

### Adding a new module

A new module is a folder. It contains its own database schema (Prisma models), API routes (tRPC router), and UI pages. It declares what permissions it needs from `core`. It does not import from other business modules — cross-module communication goes through `core` events.

```
src/
  modules/
    pms/
      schema.prisma   ← extends the shared schema
      router.ts       ← tRPC router, mounted at /api/trpc/pms.*
      service.ts      ← business logic, no HTTP concerns
      pages/          ← Next.js pages for this module
      components/     ← UI components
    drydock/
      ...
    crew/             ← add this for v0.2, same structure
      ...
```

---

## Data model — core entities

```
Tenant (shipping company)
├── subscription_plan
└── Vessel (one per ship)
    ├── imo_number, flag, class, vessel_type
    ├── Equipment[]            ← pms module
    │   └── MaintenanceJob[]
    │       └── JobExecution[]
    ├── DailyLog[]             ← pms module (watchkeeping data)
    ├── DrydockProject[]       ← drydock module
    │   └── WorkOrder[]
    ├── CrewMember[]           ← crew module
    ├── InventoryItem[]        ← inventory module
    └── Incident[]             ← safety module

User
├── belongs to Tenant
├── role: owner | superintendent | chief_engineer | supply | safety_officer
└── vessel_access: all | [vessel_id, ...]
```

---

## User roles

| Role | Access |
|---|---|
| `owner` | Read-only fleet dashboard across all modules and vessels |
| `technical_manager` | Full access to PMS, drydock, and safety for assigned vessels |
| `superintendent` | Drydock module — create, manage, close work orders |
| `chief_engineer` | PMS — log maintenance, enter daily parameters, request parts |
| `supply_officer` | Inventory and procurement — approve requisitions, manage orders |
| `safety_officer` | Safety module — incidents, inspections, drills, risk register |
| `crew_manager` | Crew module — seafarer records, scheduling, certificates |

Roles are scoped per tenant. Access to individual vessels within a fleet can be restricted per user.

---

## Tech stack

| Layer | Technology | Rationale |
|---|---|---|
| Language | TypeScript | One language across the entire stack |
| Frontend | Next.js 14 (App Router) | SSR where needed, SPA behaviour otherwise |
| API | tRPC | End-to-end type safety without code generation |
| ORM | Prisma | Schema-as-code, excellent migration tooling |
| Database | PostgreSQL | Multi-tenancy, JSON fields, full-text search |
| Cache / Queue | Redis + BullMQ | Session cache, background jobs (PDF, email) |
| File storage | S3-compatible (Cloudflare R2) | Drawings, photos, documents |
| Auth | NextAuth.js | Flexible, supports SSO for enterprise |
| Email | Resend | React-based email templates |
| PDF | Puppeteer | Drydock reports, maintenance certificates |
| UI | Tailwind CSS + shadcn/ui | Fast to build, easy for contributors |
| Billing | Stripe | Subscription management for hosted version |
| Deploy (self-host) | Docker Compose | Single command setup |
| Deploy (cloud) | Vercel + Railway | Managed hosting option |
| CI/CD | GitHub Actions | Tests, type check, lint on every PR |

**Why not microservices?** A modular monolith is faster to develop, easier to self-host (one `docker-compose up`), and sufficient for fleets up to several hundred vessels. The module boundaries are enforced at the code level, not the network level. Splitting into services is an option later — not a requirement now.

---

## Business model

**Open-core.** The full codebase is MIT-licensed. Anyone can self-host for free.

**Hosted (commercial):**


Revenue funds ongoing development. Early adopters who contribute to the project get lifetime discounted pricing.

**What stays open source forever:** the core platform, all modules, the data model, the API. The commercial layer is hosting, support, and enterprise features (audit logs export, SSO, advanced role customization).

---

## Roadmap

### v0.1 — Foundation (current focus)
- [ ] Core: multi-tenant auth, vessel registry, RBAC, file storage
- [ ] `pms`: equipment registry, maintenance schedules, job execution log, daily parameter entry
- [ ] `drydock`: work orders, contractor public links, owner dashboard, PDF report
- [ ] Self-host: single `docker-compose up` deployment
- [ ] Basic email notifications

### v0.2 — Operations
- [ ] `inventory`: spare parts catalog, requisition workflow, supplier management
- [ ] `crew`: seafarer registry, certificates, sign-on/off planning
- [ ] `safety`: incident log, drill records, inspection management, risk register
- [ ] `analytics`: fleet KPI dashboard, cross-vessel comparison, scheduled reports
- [ ] API documentation (OpenAPI)

### v1.0 — Full platform
- [ ] `voyage`: port calls, fuel reporting, CII tracking
- [ ] `finance`: OPEX budgeting, invoice management
- [ ] `class`: statutory certificates, survey tracking
- [ ] Mobile-friendly PWA
- [ ] Webhook system for third-party integrations
- [ ] Marketplace for community modules

---

## Self-hosting

```bash
git clone https://github.com/your-org/marineos
cd marineos
cp .env.example .env        # fill in your database URL, S3 credentials, etc.
docker-compose up -d        # starts postgres, redis, app
```


Full self-hosting documentation: `docs/self-hosting.md` *(coming soon)*

---

## Contributing

The project is pre-v0.1. The architecture is being finalized and the first modules are being built.

**What we need right now:**

**Full-stack developer (TypeScript / Next.js / Prisma)**
The core of what we need. If you're comfortable with tRPC, Prisma, and building data-heavy UIs, you'll find a lot to work on. Good first areas: PMS equipment registry, maintenance job UI, daily parameter entry form.

**Designer (product / UI)**
The maritime industry is full of ugly software. We want to be the exception. If you can design dense, data-heavy interfaces that are actually pleasant to use — talk to us.

**Domain expert (maritime operations)**
If you've worked as a superintendent, chief engineer, technical manager, or fleet manager — your knowledge of how things actually work is worth more than code. Open an issue, describe your experience, and let's talk about what the system needs to get right.

**How to start:**
1. Read `ARCHITECTURE.md` *(coming soon)*
2. Pick an open issue labeled `good first issue`
3. Open a discussion before writing code — the data model is still evolving

---

## Maritime glossary for developers

| Term | Meaning |
|---|---|
| **PMS** | Planned Maintenance System — scheduling and tracking of all vessel maintenance |
| **Drydock** | Vessel taken out of water for hull repairs — expensive, time-limited window |
| **Superintendent** | Shipping company's technical rep managing repairs at the shipyard |
| **Chief Engineer** | Senior engineer on board responsible for machinery and maintenance logs |
| **Running hours** | Hours a piece of equipment has operated — primary maintenance interval unit |
| **Class / Classification society** | DNV, Lloyd's, Bureau Veritas — certify vessel seaworthiness |
| **PSC** | Port State Control — government inspection of foreign vessels in port |
| **ISM / SMS** | International Safety Management / Safety Management System — regulatory framework |
| **STCW** | Convention defining minimum training and certification standards for seafarers |
| **IMO number** | Permanent 7-digit vessel identifier (equivalent of a VIN for ships) |
| **CII** | Carbon Intensity Indicator — IMO environmental rating for vessels |
| **Vetting** | Oil major inspection of tankers (SIRE) — separate from class |
| **Requisition** | Formal request from vessel to office to purchase spare parts or supplies |

---

## Why this matters

The global merchant fleet is ~100,000 vessels. The companies operating them range from single-vessel owner-operators to conglomerates with 500 ships. The middle segment — 10 to 50 vessels — is large, under-served by software, and increasingly required by charterers and regulators to demonstrate documented, digital maintenance and safety management.

AMOS has been the default for decades. It works, but it's expensive, it's locked down, and it doesn't reflect how people actually work today — on mobile, expecting real-time visibility, used to software that doesn't require a training course.

MarineOS is a bet that the maritime industry is ready for something better.

---

## License

MIT — the core platform and all modules are free to use, modify, and distribute.

The hosted service at `marineos.io` *(planned)* is commercial. The code powering it is the same code in this repository.

---

*If you're building something in maritime software and want to talk — open an issue or reach out directly.*
