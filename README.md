# Kits for Kids tutoring portal

This repository contains a complete, viewable tutoring-platform base built from the actively maintained [UPchieve](https://upchieve.org/) open-source application.

## Start the no-sign-in demo

The demo exposes the Student, Parent, Volunteer Tutor, and Administrator portals without authentication or external services.

```bash
pnpm install
pnpm dev
```

Open the local URL printed by Vite. Use the **Viewing as** menu in the top-right corner to switch roles.

The demo includes:

- Student tutoring requests, upcoming sessions, learning content, progress, and accessibility preferences
- Parent/guardian overview, consent, reports, and supervised messaging
- Tutor dashboard, student requests, availability, training, certifications, preparation, and volunteer hours
- Administrator matching approvals, tutor verification, live-session oversight, safety, content, and reports
- Shared classroom whiteboard, chat, notes, captions, timer, uploads, and safety reporting
- Larger-text, captions, and low-stimulation controls

Demo preferences are stored only in the browser. No account or cloud configuration is required.

## Full UPchieve source

The complete upstream application is preserved under:

- `apps/upchieve-frontend` — the current Vue 3 SPA from `upchieve/application/high-line`
- `apps/upchieve-backend` — the current TypeScript API, workers, database schema, migrations, and seeds from `upchieve/subway`

The production application requires Node 24, pnpm 11, PostgreSQL, Redis, MongoDB, and service credentials. Refer to the README in each upstream directory for its development setup.

## Repository structure

```text
apps/
  demo/                Immediately viewable no-auth portal
  upchieve-frontend/   Full current UPchieve web client
  upchieve-backend/    Full current UPchieve server, worker, and database
```

## Attribution and license

UPchieve source code is distributed under the Non-Profit Open Software License 3.0. Its original license, copyright notices, contribution documentation, and security documentation are retained in both upstream directories. This adaptation is distributed under the same terms; see [LICENSE](LICENSE) and [UPSTREAM.md](UPSTREAM.md).

UPchieve names and trademarks belong to UPchieve, Inc. The **Kits for Kids** name and demonstration interface are adaptation work and do not imply endorsement by UPchieve.
