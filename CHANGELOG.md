# Changelog

Each release pins one release of [ekbasis](https://github.com/OpenInterpretability/ekbasis) by commit SHA.
`@v1` follows the latest 1.x; pin a full tag (`@v1.0.0`) or this repository's commit SHA if you prefer.

## [Unreleased]

- Free for public repositories: with `api-key` empty and `permissions: id-token: write`, the repository's GitHub OIDC
  token is sent instead of a key (no secret). `api-key` is no longer required.
- Wraps ekbasis at `6d2a02f7bff9810c009f9d33dd6ec0cef5028340` (branch `oss-oidc`, PR #15); to be repinned to the ekbasis release tag before this
  action is tagged.

## [1.0.0] - 2026-10-10

- First release on its own repository, for the GitHub Marketplace.
- Wraps `OpenInterpretability/ekbasis/actions/migration-guard` at ekbasis
  [v0.1.12](https://github.com/OpenInterpretability/ekbasis/tree/v0.1.12)
  (`81f76d62e832481acccec3f68d39f7e8e3a9e3ad`), with the same inputs and outputs.
