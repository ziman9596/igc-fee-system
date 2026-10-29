# Informatics College Digital Operations Platform

> A full-stack education operations platform combining a public college website with an offline-capable desktop fee, student, voucher, reporting, and approval-management system.

> **Portfolio showcase notice:** This repository is a public showcase of a privately owned project. The underlying source code, architecture, workflows, visual design, database design, automation logic, and institutional implementation are private intellectual property and are not licensed for reuse, redistribution, or commercial use without written permission.

[![Website](https://img.shields.io/badge/website-Next.js%2014-black?logo=next.js)](./website)
[![Desktop app](https://img.shields.io/badge/desktop-Electron%20%2B%20React-47848f?logo=electron)](./desktop)
[![TypeScript](https://img.shields.io/badge/code-TypeScript-3178c6?logo=typescript)](https://www.typescriptlang.org/)
[![Database](https://img.shields.io/badge/local%20database-SQLite-003b57?logo=sqlite)](https://www.sqlite.org/)
[![Cloud](https://img.shields.io/badge/cloud-Supabase-3ecf8e?logo=supabase)](https://supabase.com/)

## Overview

College-Digitization is designed around a practical problem: a college needs a polished public presence while its administrative team needs reliable, fast, and auditable tools for daily operations.

This public repository is intended to document and showcase the product, engineering decisions, automation capabilities, and portfolio work. It should not be interpreted as a grant of access to the private production source code or institutional systems. The public showcase may contain representative documentation, sanitized assets, or selected implementation material while the complete production implementation remains private.

The project is split into two connected products:

1. **Public college website** — a responsive admissions and communications website for students, parents, faculty, and visitors.
2. **Desktop operations application** — an Electron application for accountants and administrators who manage student records, fees, vouchers, payments, approvals, reports, backups, and synchronization.

The desktop application is intentionally designed to continue working with a local SQLite database. Cloud services support synchronization and portal-related workflows without making every routine accounting action dependent on an always-on internet connection.

## Product highlights

### Public website

- College landing page with campus presentation and calls to action.
- Programs and admissions information.
- Faculty and campus information.
- Announcements and news.
- Student life and campus content.
- Wall of fame and top-performer content.
- Student portal entry points.
- Student portal dashboard and password-change flow.
- Student voucher viewing route.
- Admin login surface.
- Responsive layouts using reusable UI components.
- SEO metadata, Open Graph metadata, structured educational-organization data, sitemap, and robots configuration.
- College branding and logo assets in `website/public`.

### Desktop operations system

- Dashboard with operational statistics and visual summaries.
- Student directory with search, filters, pagination, status actions, and profile drawer.
- Student profile management, tuition packages, installments, voucher summaries, and portal-account operations.
- Charge creation with class targeting, due dates, fine rates, and fine caps.
- Principal approval workflow for sensitive changes.
- Payment entry with voucher search, student disambiguation, allocation validation, fine calculation, and atomic persistence.
- Payment history with deletion/edit-window rules and audit-friendly status handling.
- Paid-voucher printing and unpaid-voucher printing.
- Batch unpaid-voucher printing with three student vouchers per A4 page.
- Charges, fines, tuition installments, due-date requests, and class-level operations.
- Daily, monthly, and yearly reporting surfaces.
- Defaulter and class-income reporting.
- Import workflows for spreadsheet-based student and ledger data.
- Student portal-account creation and password-reset workflows.
- Local database backups and missed-backup recovery.
- Approval synchronization and snapshot synchronization.
- Day-close controls and EasyPaisa/JazzCash reconciliation reminders.
- Windows installer packaging through Electron Builder.
- Desktop logo and application/taskbar branding.

## Why this is an automation project

This project is more than a collection of CRUD screens. Its main value is the automation of repetitive, error-prone college administration.

### Automated fee and voucher lifecycle

1. An accountant creates a charge and selects the applicable classes.
2. The charge is submitted for Principal approval.
3. After approval, the system generates one voucher per eligible student.
4. Voucher numbers are assigned automatically from a shared sequence.
5. Accountants can print individual unpaid vouchers or batch-print them in a paper-efficient three-column layout.
6. Payments are allocated against vouchers with automatic balance and fine calculations.
7. Paid voucher data is available immediately for receipt printing and history.

### Automated payment validation

The payment workflow validates:

- That a student or voucher has been selected.
- That allocations belong to the selected student.
- That principal and fine allocations do not exceed the voucher balance.
- That the allocation total matches the received amount.
- That overpayments are rejected.
- That the four-o'clock payment lock is respected.
- That payment, allocation, and voucher updates are committed atomically in SQLite.

### Automated approval and synchronization

Sensitive operations are not silently applied. The system creates local approval requests for Principal review and synchronizes them periodically while online.

Examples include:

- Charge creation.
- Tuition batch operations.
- Due-date changes.
- Payment deletion outside the edit window.
- Fine-waiver requests.
- Struck-off and reactivation workflows.

The approval screen provides a read-only audit view, manual sync control, and status information. The background sync loop runs automatically while the desktop application is open.

### Automated backup and recovery

The desktop app:

- Stores operational data locally in SQLite.
- Runs scheduled backup logic.
- Checks for missed backups when the application starts.
- Maintains a canonical local ledger for reporting and export workflows.
- Pushes selected snapshots to Supabase when cloud synchronization is available.

This allows routine work to continue locally while retaining recovery and synchronization paths.

### Automated academic and financial reporting

The reporting layer calculates operational summaries from the local data model, including:

- Daily receiving and payment detail.
- Monthly and yearly totals.
- Class income summaries.
- Defaulters and outstanding balances.
- Fee-head and voucher activity.
- Opening-balance/imported ledger information.

### Automated student portal operations

The desktop system can create student portal accounts in batches and reset student passwords through Supabase Edge Functions. This avoids manually creating accounts one student at a time.

## System architecture

```text
                         ┌──────────────────────────┐
                         │ Public website            │
                         │ Next.js + React           │
                         │ Website content/SEO       │
                         └─────────────┬────────────┘
                                       │
                                       │ Supabase client / SSR
                                       ▼
                         ┌──────────────────────────┐
                         │ Supabase                  │
                         │ Auth, cloud data,         │
                         │ portal functions, sync    │
                         └─────────────▲────────────┘
                                       │
                         snapshot/approval sync
                                       │
┌────────────────────────┴─────────────────────────┐
│ Desktop operations application                    │
│ Electron main process                             │
│  - SQLite and migrations                          │
│  - IPC handlers                                   │
│  - backup and sync loops                          │
│  - approval application                           │
│                                                   │
│ React renderer                                    │
│  - dashboard                                     │
│  - students, charges, payments, reports          │
│  - voucher and report printing                   │
└───────────────────────────────────────────────────┘
```

### Desktop application layers

- `desktop/src/main` — Electron main process, database access, IPC handlers, backup logic, synchronization, and approval application.
- `desktop/src/renderer` — React user interface, routing, page components, print views, and styles.
- `desktop/src/shared` — TypeScript types, IPC names, request/response contracts, and shared domain models.
- `desktop/migrations` — SQLite schema migrations applied during application startup.
- `desktop/supabase/functions` — Supabase Edge Functions for portal accounts, password resets, and snapshot pushing.

### Website layers

- `website/app` — Next.js App Router pages and layouts.
- `website/components` — Reusable site, admin, portal, and content components.
- `website/lib` — Query and integration helpers.
- `website/public` — Logo, campus images, and other static assets.

## Technology stack

### Website

- Next.js 14
- React 18
- TypeScript
- Tailwind CSS
- Framer Motion
- Supabase SSR and JavaScript client
- Next Image and `next/font`

### Desktop application

- Electron 34
- React 18
- TypeScript
- Vite
- React Router
- SQLite through `better-sqlite3`
- Supabase JavaScript client
- ExcelJS for spreadsheet/report workflows
- Electron Builder for Windows packaging

## Repository structure

```text
College-Digitization/
├── desktop/
│   ├── migrations/              # SQLite schema migrations
│   ├── public/                  # Desktop renderer assets, including logo.png
│   ├── src/
│   │   ├── main/                # Electron main process and IPC handlers
│   │   ├── renderer/            # React desktop interface
│   │   └── shared/              # Shared TypeScript contracts
│   ├── supabase/functions/      # Cloud Edge Functions
│   ├── package.json
│   └── release/                 # Generated Windows packages
├── website/
│   ├── app/                     # Next.js routes and pages
│   ├── components/              # Reusable website components
│   ├── lib/                     # Website data/query helpers
│   ├── public/                  # Website branding and images
│   ├── package.json
│   └── .env.local.example
├── docs/
│   ├── samples/                 # Anonymized spreadsheet samples
│   └── screens/                 # Product reference material
└── README.md
```

## Prerequisites

- Node.js 20 or newer is recommended.
- npm.
- Windows is required to build the Windows installer.
- A Supabase project for cloud-backed website and desktop features.
- Native build tooling may be required by `better-sqlite3` on some machines.

## Environment configuration

### Website

Start with the example file:

```powershell
Copy-Item website\.env.local.example website\.env.local
```

Then configure the Supabase URL, public/anonymous key, and any project-specific values required by the website query helpers.

Never commit:

- `website/.env.local`
- Supabase service-role keys
- production credentials
- private database exports

### Desktop

Desktop settings are managed through the application settings flow and the local database. Review the desktop source and Supabase function configuration before deploying production credentials.

The desktop Supabase Edge Functions require their own deployed environment variables, including the Supabase project URL and service-role key where documented by the function source. Service-role credentials must remain server-side and must never be embedded in the renderer.

## Running the website locally

```powershell
Set-Location website
npm install
npm run dev
```

Open http://localhost:3000.

Production build:

```powershell
Set-Location website
npm run build
npm run start
```

Lint:

```powershell
Set-Location website
npm run lint
```

## Running the desktop application locally

```powershell
Set-Location desktop
npm install
npm run dev
```

The desktop development command starts Vite and Electron together.

Useful desktop commands:

```powershell
# Type-check renderer and main-process TypeScript
npm run typecheck

# Lint desktop source
npm run lint

# Build renderer and main process
npm run build

# Build the Windows installer
npm run dist
```

The local SQLite database is stored under Electron's `userData` directory. Database migrations run automatically during startup through the desktop database layer.

## Packaging the Windows application

From `desktop`:

```powershell
npm run dist
```

Generated files are placed in:

```text
desktop/release/
```

The package configuration creates a Windows NSIS installer and includes the packaged application. The current branding asset is:

```text
desktop/public/logo.png
```

It is used by the renderer, Electron window configuration, and Windows packaging configuration.

## Core workflows

### Student management

- Search and filter students by name, father name, roll number, class, program, gender, and status.
- Open a student profile in the drawer.
- Edit profile details.
- Manage tuition packages and installments.
- Review voucher balances and payment status.
- Create/reset student portal accounts.
- Request struck-off and reactivation actions.

### Charge and voucher generation

- Define charge category, amount, due date, fine rate, fine cap, and applicable classes.
- Submit the charge for Principal approval.
- Generate vouchers automatically after approval.
- Print unpaid vouchers individually from a student profile.
- Print an approved charge's unpaid vouchers as a batch.

### Class installment dates

- Select a class and optional session.
- Set one or more installment dates.
- Submit date changes for approval.
- Print available unpaid installment vouchers after the relevant voucher records exist.

### Payments

- Search by exact voucher number or student identity.
- Display up to 15 matching students for disambiguation.
- Select the intended student before loading pending vouchers.
- Allocate principal and fine amounts.
- Validate the received amount against allocations.
- Save payment and voucher updates atomically.
- Prompt immediately for paid-voucher printing after a successful payment.

### Payment history

- Search for a student and choose from multiple matching students.
- Review payment date, operator, amount, fee heads, edit window, and action status.
- Keep the history table focused by opening voucher print actions from the selected student's profile rather than displaying a large group of duplicate-looking buttons.

### Voucher printing

- Individual unpaid voucher printing.
- Individual paid voucher printing from the student drawer.
- Batch unpaid-voucher printing.
- A4 portrait layout with three voucher columns.
- Automatic page breaks after every three voucher cards.
- Screen-only print controls that are hidden on paper.

### Approvals

- Local request creation.
- Principal approval/rejection synchronization.
- Automatic periodic synchronization.
- Manual sync action.
- Read-only request history and status.

### Imports and data continuity

- Spreadsheet-oriented import workflows.
- Opening-balance and ledger import support.
- Local canonical ledger synchronization.
- Existing records remain available for reporting and history workflows.

## Data integrity and reliability design

The desktop application uses several deliberate safeguards:

- Atomic SQLite transactions for payment creation and voucher updates.
- Idempotent voucher generation after approval.
- Centralized voucher-number sequencing.
- Approval requests for sensitive administrative changes.
- Payment edit windows and post-window deletion approval.
- Explicit handling of imported opening-balance records.
- Local-first operation for core accounting workflows.
- Scheduled backups and startup recovery checks.
- Shared TypeScript IPC contracts between Electron and React.
- Context isolation and disabled Node integration in the renderer.

## Security considerations

- Keep service-role Supabase keys exclusively inside trusted server-side or Edge Function environments.
- Do not place private keys in `website/.env.local` committed to Git.
- Do not expose Electron database access directly to the renderer; use the existing preload/IPC boundary.
- Review Supabase Row Level Security policies before production deployment.
- Use separate development and production Supabase projects where possible.
- Treat exported spreadsheets and database backups as sensitive institutional data.
- Do not distribute real student data in screenshots, sample files, or portfolio material.

## Portfolio showcase narrative

This project demonstrates the ability to design and deliver a complete operational product rather than only a marketing interface:

- **Product thinking:** public-facing website and internal operations tool designed as one college ecosystem.
- **Workflow automation:** approvals, voucher generation, payment allocation, backups, synchronization, reporting, and portal provisioning.
- **Offline resilience:** local SQLite operations support continuity when internet access is unreliable.
- **Financial accuracy:** atomic transactions, allocation checks, fine computation, edit windows, and audit-friendly history.
- **Human-centered UX:** student disambiguation, immediate print prompts, profile-based voucher actions, navigation controls, and paper-efficient batch printing.
- **Full-stack engineering:** Next.js, React, Electron, TypeScript, SQLite, Supabase, Edge Functions, and Windows packaging.
- **Operational awareness:** backup recovery, day-close controls, imported ledgers, approval synchronization, and deployment configuration.
- **Production-minded delivery:** type checking, linting, renderer builds, Electron builds, and Windows installer packaging.

## Private property and usage terms

This project and its complete implementation are private property. Unless a separate written agreement says otherwise:

- The source code is not open source.
- No license is granted to copy, modify, fork, redistribute, resell, sublicense, or deploy the system.
- The architecture, automation workflows, database schema, UI/UX design, branding, documentation, and business rules are confidential project assets.
- Portfolio viewers may inspect the public showcase for evaluation and professional reference only.
- Screenshots, diagrams, feature descriptions, and demonstrations must not be used to recreate the system or represent it as another person's work.
- Institutional names, logos, student records, financial data, credentials, backups, exports, and production configuration must not be copied or disclosed.
- Any production access, source-code access, customization, integration, or commercial use requires prior written permission from the owner.

The public repository is a showcase layer, not a complete release of the private production codebase. If a file or implementation detail is present in this repository, it remains subject to these private-property terms unless an explicit license file states otherwise.

For collaboration, licensing, or commercial inquiries, contact the project owner directly. Do not open public issues containing credentials, student information, financial records, private infrastructure details, or confidential implementation discussions.

## Verification checklist

Before a release, run:

```powershell
Set-Location desktop
npm run typecheck
npm run lint
npm run build
npm run dist
```

For the website:

```powershell
Set-Location website
npm run lint
npm run build
```

Manually verify:

- Website navigation and responsive layouts.
- Student search and profile editing.
- Charge approval and voucher generation.
- Unpaid voucher individual and batch printing.
- Three vouchers per printed A4 page.
- Payment allocation and immediate paid-voucher prompt.
- Payment history student selection.
- Approval synchronization.
- Backup status and missed-backup recovery.
- Student portal-account workflows.
- Packaged application branding and taskbar/window icon.

## Deployment notes

### Website deployment

The website can be deployed to a Next.js-compatible host such as Vercel after configuring the required environment variables and Supabase policies.

### Desktop deployment

Build the Windows installer with Electron Builder and distribute the generated NSIS installer from `desktop/release`. Production releases should be tested on a clean Windows machine and against a backup copy of the production database before rollout.

## Data and privacy

This system handles student, financial, and institutional information. Portfolio screenshots and demo exports should use anonymized or synthetic records. The repository includes sample/reference material, but production data should never be committed to source control.

## License and ownership

This project is **proprietary and all rights reserved**. No open-source license is granted by publication of this portfolio showcase.

Before sharing this repository publicly, confirm that it contains only material approved for showcase use and does not include:

- production credentials or environment files;
- real student, parent, staff, payment, or financial data;
- private database files or backups;
- Supabase service-role keys;
- confidential institutional documents;
- private deployment configuration; or
- source files intentionally retained in the private production repository.

The project owner retains ownership of the complete source code, product implementation, automation logic, branding, and related intellectual property. Add a formal legal copyright holder, year, and contact address here if the repository will be distributed as an official public portfolio artifact.

## Acknowledgements

Built for the digital operations of Informatics College Samundri Campus, with the goal of reducing repetitive administrative work and creating a dependable bridge between public communication, student services, and financial operations.
