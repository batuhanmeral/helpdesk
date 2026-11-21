# Security Policy

## Supported versions

Security fixes land on `main` and ship in the next `v*` release image
published to `ghcr.io/batuhanmeral/helpdesk`. Older releases are not patched;
please upgrade before reporting an issue.

## Reporting a vulnerability

Please **do not open a public issue** for security problems.

Report privately through GitHub:
[Security → Report a vulnerability](https://github.com/batuhanmeral/helpdesk/security/advisories/new).

If private vulnerability reporting is not enabled yet, contact the maintainer
through their [GitHub profile](https://github.com/batuhanmeral) and ask for a
private channel before sharing details.

Include the affected commit or image digest, the role of the account used
(employee, agent, manager or admin), the steps or request that reproduce the
problem, and what you expected to happen. Remove real credentials, personal
data and private hostnames.

You can expect an acknowledgement within 7 days. Confirmed issues are fixed on
`main`, released as a new image and credited in the advisory unless you prefer
otherwise.

## Scope

Examples of issues we treat as vulnerabilities:

- A user reads, changes or exports tickets, internal notes, attachments or
  reports outside their role or department.
- Authentication or login-throttling bypass, session fixation, CSRF on a
  state-changing request.
- Stored or reflected XSS, SQL injection, or formula injection in CSV/Excel
  exports.
- Uploaded files that execute, escape the media directory or are served to
  users who should not see them.
- Secrets or personal data exposed in responses, logs, PDF exports or the
  container image.

Out of scope: findings that need `DEBUG=True` or a deployment that ignores the
settings in `.env.example`, missing hardening on a self-hosted reverse proxy,
and denial of service through traffic volume.
