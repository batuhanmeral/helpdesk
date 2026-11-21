# Dispatch

> Track internal support requests from open to close — with role-based access, business-hours SLAs, auto-assignment, and full TR/EN support, all in a hardened, server-rendered Django app.

Dispatch is a Django help-desk platform that takes an internal support request from open to close. Employees raise tickets, the system routes each one to the right department and auto-assigns an available agent, and managers watch the whole pipeline against business-hours SLAs — with escalation, first-response targets, satisfaction ratings, and CSV/Excel/PDF reporting built in.

## Features

- **Full ticket lifecycle** — Open → In Progress → Resolved → Closed (+ Escalated), with reopen limits, requester confirmation, and auto-close of stale tickets.
- **Kanban board** — drag-and-drop tickets across status columns for an at-a-glance view of the whole queue.
- **Smart auto-assignment** — routes each ticket to the **least-loaded** agent in the department (round-robin as a tie-breaker), with a per-agent active-ticket cap so no one gets overloaded.
- **Role-based access** — Employee, Agent, Manager, Admin; each role sees and does only what it should.
- **Security hardening** — brute-force lockout (django-axes), uploads validated by extension, size, and real content, CSRF on every form, and auto-enabled HTTPS redirect, HSTS, and `Secure`/`HttpOnly` cookies in production.
- **Business-hours SLAs** — due dates from working hours (Mon–Fri, 09:00–18:00) per priority (Urgent 4h / High 24h / Normal 72h / Low 168h), with overdue tracking and early warnings.
- **First-response tracking (FRT)** — stamped the moment an agent first replies, measured in *business* hours against a per-priority target (Urgent 1h / High 4h / Normal 8h / Low 24h), with average, target compliance, and still-unanswered counts on the dashboard.
- **Internal notes** — agents discuss a ticket in-thread without the requester ever seeing it; filtered out server-side, never counted as a first response, never sent to the requester.
- **Canned responses** — reusable reply templates, global or per department, with placeholders (`{{talep_sahibi}}`, `{{bilet_kodu}}`, …) filled in from the ticket, and usage counted per template.
- **Collaboration** — threaded comments, file attachments, color-coded tags, and a full per-ticket audit trail.
- **In-app notifications** — live unread badge with alerts for assignments, status changes, and SLA warnings.
- **Reporting** — metrics dashboard with one-click **CSV / Excel / PDF** export.
- **Bilingual (TR / EN) + dark mode**, remembered per user.

## Tech Stack

| Layer | Technologies |
| --- | --- |
| **Backend** | Python 3.13, Django 6.0 |
| **Frontend** | Server-rendered templates, vanilla JS, custom CSS (light/dark) |
| **Database** | PostgreSQL 17 (`psycopg2`) |
| **Security** | django-axes (brute-force lockout), Django password validators, hardened upload validation, HSTS / secure cookies |
| **Documents** | WeasyPrint (PDF), openpyxl (Excel), Pillow (images) |
| **DevOps** | Gunicorn, Docker, Docker Compose, gettext (i18n) |

## Installation

**With Docker** (app + PostgreSQL 17, migrations run on boot, served on `127.0.0.1:8000`):

```bash
docker compose up -d --build
```

**Manual setup** (requires Python ≥ 3.13 and PostgreSQL; PDF export needs [WeasyPrint's system libraries](https://doc.courtbouillon.org/weasyprint/stable/first_steps.html#installation)):

```bash
git clone https://github.com/batuhanmeral/helpdesk.git
cd helpdesk
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env              # fill in SECRET_KEY, ALLOWED_HOSTS, DB_*
python manage.py migrate && python manage.py createsuperuser
python manage.py seed_demo        # optional demo data — admin / admin123
python manage.py runserver        # http://localhost:8000
```

## Testing

The suite uses Django's test runner against PostgreSQL. Django creates and drops a throwaway `test_<DB_NAME>` database, so the `DB_*` user in `.env` needs permission to create databases.

```bash
python manage.py test                 # whole suite
python manage.py test tickets         # one app
```

| Area | Covered |
| --- | --- |
| `tickets` | Business-hours SLA math, SLA due dates, auto-assignment, lifecycle transitions, first-response stamping, internal-note visibility, canned responses |
| `reports` | First-response metrics on the dashboard, export data and PDF summaries in both TR and EN, spreadsheet-formula neutralization in CSV/Excel exports |

## CI and security

[CI](.github/workflows/ci.yml) runs on every pull request and push to `main`: Ruff (`pip install ruff==0.17.5 && ruff check .`), `manage.py check`, `check --deploy --fail-level WARNING` with `DEBUG=False`, a missing-migration check and the full suite on Python 3.13 + PostgreSQL 17.

[Security](.github/workflows/security.yml) runs on every pull request, on `main`, monthly and on demand:

| Job | Tool | Fails on |
| --- | --- | --- |
| `sast` | Semgrep (`p/python`, `p/django`, `p/dockerfile`), zizmor | Semgrep ERROR/WARNING, zizmor medium+ |
| `secrets` | gitleaks, full history | any finding |
| `dependencies` | Trivy fs (`requirements.txt`), Trivy config (Dockerfile) | fixable HIGH/CRITICAL |
| `dependency-review` | GitHub dependency review (PRs only) | new HIGH+ dependency |
| `image` | Trivy scan of the locally built image | fixable HIGH/CRITICAL |
| `dast` | compose stack with throwaway settings, `check --deploy`, ZAP baseline (not on PRs) | deploy warning, ZAP High alert |

Outside pull requests the `image` job also keeps a CycloneDX SBOM as an artifact. CodeQL runs as GitHub's default setup and [Scorecard](.github/workflows/scorecard.yml) reports monthly. Dependabot opens grouped monthly PRs for Actions, pip and Docker images (7-day cooldown, majors by hand). Scanner images pinned inside the workflows and the ruff version are bumped by hand, tag and digest together.

[Release](.github/workflows/release.yml) runs only for `vX.Y.Z[-pre]` tags on a commit in `main`. It reruns CI and Security, pushes the image to `ghcr.io/batuhanmeral/helpdesk` by digest, blocks on fixable HIGH/CRITICAL vulnerabilities, attests provenance, Cosign-signs the digest and only then adds the version tags. Verify a release with:

```bash
gh attestation verify oci://ghcr.io/batuhanmeral/helpdesk@sha256:DIGEST --repo batuhanmeral/helpdesk
cosign verify ghcr.io/batuhanmeral/helpdesk@sha256:DIGEST \
  --certificate-identity-regexp '^https://github\.com/batuhanmeral/helpdesk/\.github/workflows/release\.yml@refs/tags/v.*$' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com
```

To report a vulnerability, see [SECURITY.md](SECURITY.md). Design and operational notes live in `.docs/` (Turkish, kept local).

## Project Structure

| App | Responsibility |
| --- | --- |
| `identity` | User model, authentication, roles, audit logging |
| `departments` | Departments, categories, auto-assignment config |
| `tickets` | Ticket lifecycle, SLAs, first response, assignment, comments, internal notes, canned responses, attachments, history |
| `notifications` | In-app notifications and the unread-count badge |
| `reports` | Dashboard metrics and CSV / Excel / PDF exports |
| `config` | Project settings, URLs, role-based landing dashboard |

## Screenshots

**Dashboard**

<img src="docs/dashboard.png" alt="Dashboard" width="850">

**Reports**

<img src="docs/reports.png" alt="Reports" width="850">

**Kanban Board**

<img src="docs/kanban.png" alt="Kanban Board" width="850">

## License

[MIT License](LICENSE). © 2026 Batuhan Meral.
