# Changelog

Each release pins one release of [ekbasis](https://github.com/OpenInterpretability/ekbasis) by commit SHA.
`@v1` follows the latest 1.x; pin a full tag (`@v1.0.0`) or this repository's commit SHA if you prefer.

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
