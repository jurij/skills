# Database

`packages/db`: Drizzle + Postgres, migrations generated with drizzle-kit, committed, and applied explicitly. Native and Desktop reach the database only through a Web or Server backend.

## Local Postgres

- One root `compose.yaml` with a Postgres service, a `pg_isready` healthcheck, and a named volume. Host port `${POSTGRES_PORT:-5432}`, bound to `127.0.0.1`.
- `.env.example` documents `POSTGRES_PORT=5432` and `DATABASE_URL` built from it.
- Root scripts: `"db:up": "docker compose up -d --wait"`, `"db:down": "docker compose down"` (keeps the volume), plus migrate/generate scripts. `--wait` returns once the healthcheck passes, so migrations can follow directly.
- Before the first `db:up`, check that the port is free; if not, set a free `POSTGRES_PORT` in `.env`.
- drizzle-kit is a devDependency of `packages/db` with `generate`/`migrate` scripts there (`bun --env-file=../../.env drizzle-kit migrate`); the root `db:generate`/`db:migrate` call them via `bun run --cwd packages/db`.
- Leave the Compose project name at its default, the folder name: each worktree then gets its own containers and volume.

AGENTS.md and README state: "Parallel worktree: change `POSTGRES_PORT` in that worktree's `.env`." Add a TODO there: automate per-worktree port assignment if manual ports become painful.

## Code

Each backend entry builds its own connection with a driver documented for its runtime and injects the db into shared code. Schema and migration config are importable without opening a connection. Pin drizzle-orm, drizzle-kit, and the driver from one supported set; an RC generator never pairs with a stable runtime without evidence.

Acceptance: migrations apply to an empty database and a rerun is a no-op; an application query works under the selected runtime; a second worktree with another `POSTGRES_PORT` runs `db:up` alongside the first.

Pitfall (2026-11-02): schema push is not a migration; production uses committed migrations only.
Pitfall (2026-11-02): under the isolated linker, a root-level drizzle-kit cannot resolve the `pg` driver of `packages/db` and `migrate` exits 1 without an error message.

Sources: [Drizzle migrations](https://orm.drizzle.team/docs/migrations), [Compose project name](https://docs.docker.com/compose/how-tos/project-name/).
