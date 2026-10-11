# Changelog

Each release pins one release of [ekbasis](https://github.com/OpenInterpretability/ekbasis) by commit SHA.
`@v1` follows the latest 1.x; pin a full tag (`@v1.0.0`) or this repository's commit SHA if you prefer.

## [1.5.0] - 2026-10-11

- Wraps ekbasis [v0.1.21](https://github.com/OpenInterpretability/ekbasis/tree/v0.1.21) (`034a95dddb2f1886ef9f570dbba2d6f726e81f39`).
- `framework: rails` is supported (opt-in; no longer behind an experimental switch).
- New input `redis` (`auto`, `on`, `off`): a throwaway Redis on the generator's isolated network for projects whose
  migrations need it.

## [1.4.0] - 2026-10-11

- Wraps ekbasis [v0.1.20](https://github.com/OpenInterpretability/ekbasis/tree/v0.1.20) (`209353139a23aa3682bd83513415abb732dec338`).
- Code probes: a key collision, a NOT NULL left empty, or rows still referenced by a foreign key are checked on a copy
  of the scratch database; when PostgreSQL reproduces the error with real rows, the verdict is "fails".
- Edited migrations now get a verdict (on the base schema before them).
- New input `setup-sql`: SQL run by the scratch admin before the base's migrations (read from the base branch, never
  from the pull request).

## [1.3.0] - 2026-10-11

- Wraps ekbasis [v0.1.19](https://github.com/OpenInterpretability/ekbasis/tree/v0.1.19) (`9caf2b222e048a1c5b4279ac643feb068029212a`).
- The model now sees the part of the schema the migration reaches, cut by a parser: the tables it touches in full,
  foreign-key neighbors in one line each, dependent views, triggers with their functions, sequences, enums and domains.
  Pre-registered on 184 labelled migrations: accuracy 0.880 → 0.897 with replica statistics, unchanged without; fewer
  false alarms; median input tokens −23% to −30%.
- States up to 200,000 characters (~64k tokens) are read (was 60,000).

## [1.2.2] - 2026-10-11

- Wraps ekbasis [v0.1.17](https://github.com/OpenInterpretability/ekbasis/tree/v0.1.17) (`ec91a416e7a6234c13670befde01958cfae9550c`). No change to the
  guard's behaviour.

## [1.2.1] - 2026-10-11

- Wraps ekbasis [v0.1.16](https://github.com/OpenInterpretability/ekbasis/tree/v0.1.16) (`bbcf44ce5d423d0e7244606d4919eeb6fe8c8761`).
- Fixed: with `framework: django|alembic`, when the SQL could not be rendered at all, the check now says it cannot
  foresee (and posts the comment) instead of stopping with an error.

## [1.2.0] - 2026-10-11

- Wraps ekbasis [v0.1.15](https://github.com/OpenInterpretability/ekbasis/tree/v0.1.15) (`fc02199c3ef00c922b808bfab57b515b4688b57f`).
- New input `framework` (`sql` by default, `alembic`, or `django`, experimental): the framework renders the SQL, running
  the pull request's code only inside a container with no network and no secrets. Migrations with code that is not SQL
  are listed for review by hand (`on-code: warn`, default) or fail the check (`on-code: block`). Also `image`, `setup`,
  `workdir`, `env`, `migrate-command`, `alembic-command`.

## [1.1.1] - 2026-10-11

- Wraps ekbasis [v0.1.14](https://github.com/OpenInterpretability/ekbasis/tree/v0.1.14) (`ed600036ae7823eb7b7aef438818e649d572bbe1`).
- Fixed: a `BEGIN` inside a function body no longer switches a file to autocommit rules; goose files run only their
  Up sections, and `-- +goose NO TRANSACTION` is read as autocommit.

## [1.1.0] - 2026-10-11

- Wraps ekbasis [v0.1.13](https://github.com/OpenInterpretability/ekbasis/tree/v0.1.13) (`70be944bb81fcedca2e444cb875cbb9a51047af2`).
- New input `verdict-cache` (default `true`): a migration already checked in the same pull request, with the same
  file, base schema, client version, transaction mode and statistics, reuses the model's answer from the Actions
  cache (30 days) with no model call and no tokens. The cache is scoped to the pull request. Reused answers are marked
  in the comment.

## [1.0.0] - 2026-10-10

- First release on its own repository, for the GitHub Marketplace.
- Wraps `OpenInterpretability/ekbasis/actions/migration-guard` at ekbasis
  [v0.1.12](https://github.com/OpenInterpretability/ekbasis/tree/v0.1.12)
  (`81f76d62e832481acccec3f68d39f7e8e3a9e3ad`), with the same inputs and outputs.
