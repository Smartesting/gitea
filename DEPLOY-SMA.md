# Gitea - Deployment, Smoke, and Mutation Notes

## Quick Start

```bash
./tester-env deploy    # Build image, start container, create admin user
./tester-env seed      # Populate with deterministic seed data (Atlas Labs)
./tester-env verify    # Check seeded state via API
./tester-env reset     # Clean slate (stop + remove container + volume)
```

## CLI Commands

| Command | Description |
|---------|-------------|
| `deploy` | Build image, start container, create admin user |
| `seed` | Populate app with deterministic Atlas Labs seed data |
| `verify` | Run API verification against seeded state |
| `reset` | Stop container, remove container and data volume |
| `stop` | Stop container (preserves data) |
| `logs` | Tail container logs |
| `status` | Show container status |

Options:
- `--run-id <id>` isolates container, image, and volume names for parallel scenario runs.
- `--port <port>` binds the app to a specific host port. Default: 3000.
- `--ref <git-ref>` checks out a ref before deployment for manual runs.

## Build

```bash
./tester-env deploy
```

Parallel scenario example:

```bash
./tester-env deploy --run-id run-001 --port 3100
./tester-env seed --run-id run-001
./tester-env verify --run-id run-001
./tester-env reset --run-id run-001
```

Uses the root `Dockerfile` (multi-stage Go + PNPM). Builds with `make frontend` then `make backend`.

- Port: `3000`
- URL: `http://localhost:3000`
- Requires pinned `SECRET_KEY` and `INTERNAL_TOKEN` for deterministic behavior.
- `DISABLE_SSH=true` and `INSTALL_LOCK=true` skip setup wizards.

## Credentials

- Username: `admin`
- Password: `admin`

Seed users: `alice` / `alice123`, `bob` / `bob123`, `charlie` / `charlie123`

## Seed Data

```bash
./tester-env seed
```

Seeded entities (Atlas Labs scenario):
- 3 users: Alice Chen, Bob Martinez, Charlie Kim
- 1 org: Atlas Labs (`atlas-labs`)
- 1 team: Engineering
- 3 repos: checkout-service, data-pipeline, frontend-app
- checkout-service: 4 issues (3 open, 1 closed), 5 labels, 1 milestone
- checkout-service: 1 branch (`feature/refund-audit-log`), 1 PR, 1 release (`v1.4.0`), 1 wiki page

Issues:
1. "Implement payment gateway integration" (open, enhancement)
2. "Add refund audit log notes" (open, enhancement)
3. "Fix checkout timeout on large carts" (open, bug, priority-high)
4. "Set up CI/CD pipeline" (closed, documentation)

Milestone: "v1.4.0 Gateway Stabilization"
Release: "v1.4.0 Gateway Stabilization"
Wiki page: "Incident Response"

## Reset

```bash
./tester-env reset
```

Then run `deploy` and `seed` to start fresh from a clean slate.

## API Verification

```bash
./tester-env verify
```

Checks:
- org atlas-labs exists
- repos count = 3
- checkout-service issues = 4 (3 open)
- branches = 2 (main + feature/refund-audit-log)
- PRs = 1
- releases = 1
- wiki pages = 1

## Browser Smoke

Build from source, login as admin/admin, create a repo through UI, verify it appears. App is deterministic with pinned env vars.

## Mutation Smoke

Change a locale string in `options/locale/locale_en-US.json`, rebuild image, restart container. The change is visibly reflected in the served HTML (e.g. signed-out landing page).

## Reset Path

```bash
docker stop tester-env-gitea && docker rm -v tester-env-gitea && docker volume rm tester-env-gitea-data
```

Then `./tester-env deploy` to start fresh.

## Baseline

Pinned SHA: `bf1b54c3e34df542ea7448f0962009b3951bd48c`
Branch: `tester-env-baseline`
