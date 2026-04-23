# Servcy
Founder-built SaaS for freelance agencies that need project execution, time tracking, and budget visibility in one system.

> Built and shipped in 2024 as a real product, not as a starter kit or tutorial exercise.

## What problem this solves
Small agencies usually run delivery across a fragmented stack: issues in one tool, time in another, budgets in spreadsheets, conversations in Slack, and project context scattered across docs. Work gets done, but margin, ownership, and delivery risk stay blurry.

Servcy was built to close that gap. It brings project management, time tracking, budgeting, integrations, and AI-assisted workflows into one operating layer so an agency can see both execution and economics in the same place.

## Product overview
Servcy sits somewhere between Jira, Asana, Toggl, and a lightweight agency ERP.

At the product level it supports:

- Multi-tenant workspaces with role-based access control
- Projects, issues, sub-issues, cycles, modules, pages, and saved views
- Running timers, timesheets, billable vs non-billable logs, and approval workflows
- Project budgets, expenses, member rates, and cost analysis
- Integrations with GitHub, Slack, Google, Microsoft, Notion, Asana, Jira, Trello, and Figma
- Billing, notifications, webhooks, async workers, and AI-assisted actions

This workspace mirrors the three repos that made up the product:

| Repo | Responsibility |
| --- | --- |
| `ServcyServer` | Django + DRF backend, business rules, integrations, async jobs, billing, notifications |
| `ServcyClient` | Next.js customer-facing application |
| `ServcyLanding` | Nuxt.js marketing site, docs, blogs, and support content |

Each subproject has its own README. This top-level document is the system view.

## Key features
### Delivery workflow
- Workspaces, invitations, roles, and project membership
- Projects, issues, states, labels, priorities, cycles, and modules
- Saved project views and workspace-wide views
- Rich pages and embedded documentation alongside execution
- Search, dashboards, and activity tracking

### Time and commercial visibility
- Per-issue timers and manual time logs
- Billable vs non-billable tracking
- Approval flows for time logs
- Project budgets and explicit expense tracking
- Member rates, estimated cost, and actual cost analysis

### Integrations and automation
- OAuth-based integrations with common delivery tools
- Webhook ingestion for external events
- Async notifications, analytics exports, and provider sync
- Billing and subscription lifecycle management through Paddle

### Product surfaces around the core workflow
- In-app notifications and inbox-style event handling
- AI-assisted issue and content workflows
- Separate landing site for positioning, onboarding, docs, and support

## System architecture
Servcy was intentionally split into three deployable surfaces instead of one giant repository.

```text
Users
  |
  +--> ServcyClient (Next.js app)
  |       |
  |       +--> JWT-authenticated REST calls
  |       v
  |   ServcyServer (Django + DRF modular monolith)
  |       |
  |       +--> PostgreSQL for product state
  |       +--> Redis for cache and Celery broker
  |       +--> Celery workers for async jobs
  |       +--> S3-compatible storage for uploads
  |       +--> Paddle, SendGrid, Twilio, OpenAI
  |       `--> Webhook endpoints for GitHub, Slack, Google, Microsoft, Figma, Paddle, etc.
  |
  `--> ServcyLanding (Nuxt.js)
          |
          `--> Marketing pages, docs, blogs, support, and product narrative
```

A few architectural choices matter:

- The backend is a modular monolith. That kept business rules, permissions, billing, and integrations in one place while the product was still evolving quickly.
- The frontend and landing site are separate repos because they ship on different cadences and fail differently.
- The main tenant boundary is the workspace. You can see that in routing, API design, permissions, billing, and analytics.

## Tech stack
| Layer | Stack |
| --- | --- |
| Customer app | Next.js 14, React 18, TypeScript, Tailwind, MobX, SWR, TipTap |
| Backend API | Django 4.2, Django REST Framework, Python 3.11, SimpleJWT |
| Data | PostgreSQL, Redis |
| Async | Celery |
| Marketing site | Nuxt 3, Vue 3, Nuxt Content, Nuxt UI, Nuxt Image |
| Integrations | GitHub, Slack, Google, Microsoft, Notion, Asana, Jira, Trello, Figma |
| Billing / comms / AI | Paddle, SendGrid, Twilio, OpenAI |
| Storage / infra | S3-compatible object storage, webhook endpoints, structured logging |

## Repository structure
```text
.
|-- README.md
|-- ServcyServer/
|   |-- app/
|   |-- iam/
|   |-- integration/
|   |-- project/
|   |-- dashboard/
|   |-- notification/
|   |-- billing/
|   \-- webhook/
|-- ServcyClient/
|   |-- web/
|   |-- packages/ui/
|   |-- packages/types/
|   \-- packages/editor/
\-- ServcyLanding/
    |-- pages/
    |-- components/
    |-- content/
    \-- public/
```

## Getting started
This workspace is not a single-command monorepo. Each application runs independently.

### Prerequisites
- Node.js 18+
- Yarn 1.22.x
- Python 3.11.1
- Poetry
- PostgreSQL
- Redis

### Install and configure
```bash
cd ServcyServer
poetry install
cp config/config.ini.example config/config.ini
mkdir -p logs
poetry run python manage.py migrate

cd ../ServcyClient
yarn install
# create web/.env.local with the NEXT_PUBLIC_* values your local setup needs

cd ../ServcyLanding
yarn install
```

### Run the system
```bash
# terminal 1
cd ServcyServer
poetry run python manage.py runserver

# terminal 2
cd ServcyServer
poetry run celery -A app worker -l info

# terminal 3
cd ServcyClient
yarn dev

# terminal 4
cd ServcyLanding
yarn dev
```

Notes:

- The backend can start with blank credentials for providers you are not exercising locally, but full product behavior requires real integration keys.
- The landing site is optional for app development, but it is part of the shipped product story.

## Design decisions
- **Modular monolith first.** A founder-led product in 2024 did not need microservices. It needed fast iteration, shared business rules, and fewer distributed failure modes.
- **Separate repos by deployment surface.** The customer app, backend, and marketing site have different release cadences and different operational risks. Splitting them reduced blast radius.
- **Workspace as the core tenant boundary.** Slug-based routing made URLs human, kept scoping explicit, and forced multi-tenant concerns into the design early.
- **Async for external work.** Notifications, exports, provider sync, and webhook-driven side effects do not belong on the critical request path.
- **Editors as product features, not bolt-ons.** The custom TipTap packages exist because notes, pages, and structured content were part of how teams actually worked inside the product.
- **Budgeting inside project management.** Time tracking without rate and budget context becomes activity data. Agencies needed financial visibility next to execution.

## Tradeoffs and challenges
- Supporting many integrations expanded the operational surface area fast. Every provider has its own OAuth quirks, webhook retry behavior, and schema drift.
- Permissions and tenancy touched almost every feature. That is the cost of building a real multi-workspace product instead of a single-tenant app with role labels sprinkled on top.
- Separate repos improved isolation but increased cross-repo coordination. API changes, type drift, and environment parity all needed active discipline.
- The frontend shipped with a pragmatic mix of custom UI, Tailwind, Ant Design, and MUI. That accelerated delivery but created a real consistency tax.
- Self-hosting is possible, but not trivial. Once you add Postgres, Redis, workers, object storage, provider credentials, and billing, this stops looking like a demo very quickly.

## What didn't work
- Treating integrations like feature checkboxes was naive. Every serious integration behaved like a product of its own.
- I underestimated the maintenance cost of custom editor surfaces. They were differentiating, but expensive in polish, compatibility, and QA.
- Automated coverage did not grow as fast as the feature set. Too much confidence still came from manual regression testing across permissions, billing, and integrations.
- Some frontend decisions were optimized for shipping speed instead of long-term coherence. That made the product real faster, but the design-system debt is visible.
- Early AI work was most useful when constrained to specific actions. Broad assistant-style positioning was less valuable than targeted help inside existing workflows.

## What I'd do differently
- Add contract tests and end-to-end coverage around auth, billing, time tracking, and webhook ingestion earlier.
- Standardize the frontend on a tighter design system instead of carrying multiple component ecosystems.
- Move backend configuration toward typed, env-driven settings and a more container-friendly local setup.
- Introduce a stronger outbox and idempotency strategy for integration-heavy flows.
- Narrow the ICP harder. The product is most compelling when it is opinionated about agency operations, not when it tries to be generic project management software.

## Screenshots and demo
Placeholder for a portfolio pass:

- Product walkthrough GIF: dashboard, issues, timesheet, and costing
- Integration demo: inbound event to issue or notification flow
- Workspace admin flow: invites, roles, billing, and plan management
- AI-assisted workflow demo inside issue or page creation

Useful existing assets already live in `ServcyLanding/public/shots/`:

- `dashboard.webp`
- `issues.webp`
- `timesheet.webp`
- `costing.webp`
- `integrations.webp`
- `inbox-to-issue.webp`

## Contributing
This workspace is primarily a product portfolio and source snapshot, not a generic community starter kit.

If you want to contribute:

1. Open an issue against the specific subproject you want to change.
2. Keep backend contracts and frontend expectations aligned.
3. Treat permissions, billing, and integrations as high-risk areas.
4. Follow the repo-specific contribution guides:

- [ServcyServer/CONTRIBUTING.md](./ServcyServer/CONTRIBUTING.md)
- [ServcyClient/CONTRIBUTING.md](./ServcyClient/CONTRIBUTING.md)
- [ServcyLanding/CONTRIBUTING.md](./ServcyLanding/CONTRIBUTING.md)

## License
The root workspace is licensed under the [MIT License](./LICENSE).

Each shipped product repo also carries its own license file:

- [ServcyServer/LICENSE.txt](./ServcyServer/LICENSE.txt)
- [ServcyClient/LICENSE.txt](./ServcyClient/LICENSE.txt)
- [ServcyLanding/LICENSE.txt](./ServcyLanding/LICENSE.txt)

If you reuse code from a specific subproject, follow the license shipped with that subproject.
