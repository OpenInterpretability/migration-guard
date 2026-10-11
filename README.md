# Ekbasis Migration Guard

Before a pull request is merged: will its new PostgreSQL migrations **lose data** or **fail** on the existing database?
The guard builds your schema from the base branch's migrations, runs each new migration on it, asks the Ekbasis model
the two questions, and posts one comment on the pull request.

Pre-registered evaluation on **184 migrations the model never saw**:

| | Ekbasis | Regex rules |
|---|---|---|
| Verdict right (risky vs ok) | **87.5%** | 69.6% |
| Recall on risky migrations | **94.6%** (53 of 56) | |
| False alarms on safe migrations | **15.6%** | 43.0% |

Limits, method and data: [docs/MIGRATION_GUARD.md](https://github.com/OpenInterpretability/ekbasis/blob/v0.1.12/docs/MIGRATION_GUARD.md).
A low probability is not proof of safety.

## Free for open source

In a **public** repository you need no key and no secret: grant `id-token: write` and leave `api-key` out.

```yaml
name: migration guard
on:
  pull_request:                     # never pull_request_target
    paths: ["**/migrations/**", "**/*.sql"]
permissions:
  contents: read
  pull-requests: write              # the comment
  id-token: write                   # the free tier: the repository's GitHub OIDC token instead of a key
jobs:
  guard:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: OpenInterpretability/migration-guard@v1   # no api-key
```

The action requests the job's OIDC token for the audience `openinterp.org` (masked in the logs, valid a few minutes,
requested only for the hosted API) and the API verifies GitHub's signature and that the repository is public.
Repositories created at least 7 days ago qualify, for up to 2,000 migrations a month (200 a day) each. Past the quota,
or on a fork's pull request (no OIDC token there), the verdict is "cannot foresee", as with no key. Private
repositories use an API key.

## Install in one minute

1. Get an API key at [openinterp.org/console](https://openinterp.org/console). Price: $0.30 per 1M input tokens;
   a migration is about 4k tokens, so about $0.0012 per migration checked.
2. Add it to the repository: **Settings → Secrets and variables → Actions → New repository secret**, name
   `EKBASIS_API_KEY`.
3. Add `.github/workflows/migration-guard.yml`:

```yaml
name: migration guard
on:
  pull_request:                     # never pull_request_target
    paths: ["**/migrations/**", "**/*.sql"]
permissions:
  contents: read
  pull-requests: write              # the comment
jobs:
  guard:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0            # the base branch's migrations build the schema
      - uses: OpenInterpretability/migration-guard@v1
        with:
          api-key: ${{ secrets.EKBASIS_API_KEY }}
          # db-stats-url: ${{ secrets.READ_REPLICA_URL }}   # optional: planner statistics only
          # mode: both              # comment | check | both
          # fail-on: risky          # risky | never
```

Migrations are found by default for Prisma (`migrations/*/migration.sql`), Diesel (`migrations/*/up.sql`),
golang-migrate and others (`*.up.sql`, `*migrations/*.sql`, `*migrate/*.sql`). Set `paths` (one glob per line) for
your own layout.

## What the comment looks like

One comment per pull request, updated on each push. A pull request that drops a column:

> ### Ekbasis migration guard
>
> **Risky:** at least one migration may lose data or fail on the existing database.
>
> | Migration | Verdict | Why |
> |---|---|---|
> | db/migrations/0002_drop_phone.up.sql (golang-migrate, one transaction) | **risky** | may lose existing data (96%) |

A pull request that adds a nullable column and an index:

> **No data loss or failure foreseen.**
>
> | Migration | Verdict | Why |
> |---|---|---|
> | db/migrations/0002_add_shipped_at.up.sql (golang-migrate, one transaction) | ok | no data loss or failure foreseen (lose 0%, fail 1%) |

With `mode: check` or `both` and `fail-on: risky`, the job fails on **risky** and on **cannot foresee**, so it can be
a required status check. The result also goes to the job summary and to the outputs `verdict` (`ok`, `risky`,
`cannot-foresee`) and `result` (path of the JSON).

## Inputs

| Input | Default | |
|---|---|---|
| `api-key` | empty | Ekbasis API key, from a secret; empty in a public repository with `id-token: write` (free tier) |
| `api-url` | `https://openinterp.org/api/v1` | the hosted API, or your own server |
| `base-ref` | the PR's base | branch the PR merges into |
| `paths` | Prisma, Diesel, `*.up.sql`, `*migrations/*.sql` | one fnmatch glob per line |
| `lost-threshold` / `fail-threshold` | `0.5` | risky when P(data lost) / P(fails) reaches this |
| `autocommit` | `false` | `true` for `psql -f` semantics; default is one transaction per file |
| `mode` | `both` | `comment`, `check` or `both` |
| `fail-on` | `risky` | `risky` or `never` |
| `db-stats-url` | empty | read-only replica, from a secret; planner statistics only |
| `postgres-image` | `postgres:16` | the scratch server |
| `github-token` | `github.token` | writes the comment |

## Security

- **Trigger on `pull_request`, never on `pull_request_target`** with a checkout of the pull request's code. Under
  `pull_request_target`, untrusted code runs with your secrets and a write token.
- **Forks get no secrets** and no OIDC token. On a fork's pull request the key is empty, the verdict is "cannot foresee", and the
  comment step only warns (the fork's token is read-only); the result is still in the job summary. Choose one:
  - `fail-on: ${{ github.event.pull_request.head.repo.full_name == github.repository && 'risky' || 'never' }}`
  - or run the job only for your own branches: `if: github.event.pull_request.head.repo.full_name == github.repository`
- **Minimal permissions:** `contents: read` and `pull-requests: write`, plus `id-token: write` for the free tier
  (it lets the job prove which repository it runs in and grants no access to the repository), nothing else.
- **The pull request's SQL is untrusted, and it runs**, but only in a throwaway PostgreSQL container on an internal
  Docker network (no route out, no published port), as a role made for the check that is not a superuser and cannot
  create databases or roles. `COPY ... PROGRAM`, `pg_read_file`, untrusted languages and the like fail with
  "permission denied", and the guard reports "cannot foresee". It never retries with more privileges.
- **Pinned code.** Each release of this action pins one release of
  [ekbasis](https://github.com/OpenInterpretability/ekbasis) by full commit SHA (see [CHANGELOG.md](CHANGELOG.md)
  and `.github/ekbasis-pin`). For the strictest setup, pin this action by its own commit SHA instead of `@v1`.

### What is sent to the API

- the **schema**: DDL of the tables a new migration names and of the tables linked to them by foreign keys, as
  built from your migration files;
- the **new migration's SQL**, with comments removed;
- **optional planner statistics** when you set `db-stats-url`: estimated rows per table (`pg_class.reltuples`), NULL
  fraction and distinct count per column (`pg_stats.null_frac`, `n_distinct`).

**Never data values.** No row is read, and `most_common_vals` is never queried. The API key and the replica URL
travel only as environment variables, and the replica's URL and password are removed from any error the guard prints.

## Self-host with the open weights

The model is open (Apache-2.0): [Ekbasis-27B](https://huggingface.co/caiovicentino1/Ekbasis-27B) (bf16, 80 GB GPU),
[FP8](https://huggingface.co/caiovicentino1/Ekbasis-27B-FP8) (48 GB) and
[INT4](https://huggingface.co/caiovicentino1/Ekbasis-27B-INT4) (32 GB). Serve it as in the
[ekbasis quick start](https://github.com/OpenInterpretability/ekbasis#quick-start), then point the action at it:

```yaml
      - uses: OpenInterpretability/migration-guard@v1
        with:
          api-url: https://ekbasis.internal.example.com   # reachable from the runner (e.g. a self-hosted runner)
          api-key: ${{ secrets.EKBASIS_API_KEY }}         # any value if your server does not check keys
```

Your schema and SQL then never reach OpenInterpretability.

## License

Apache-2.0. See [LICENSE](LICENSE).
