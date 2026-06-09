# Rust Cut Tag

![Latest Release](https://img.shields.io/github/v/release/p6m-actions/rust-cut-tag?style=flat-square&label=Latest%20Release&color=blue)

## Description

A GitHub Action that bumps the version in `Cargo.toml` and creates an annotated git tag for Rust projects. It uses [`cargo-release`](https://github.com/crate-ci/cargo-release) to perform the version bump and handles committing, tagging, and pushing to the remote.

Supports both single-crate projects and Cargo workspaces.

## Usage

```yaml
- uses: p6m-actions/rust-cut-tag@v1
  with:
    version-level: patch
```

## Inputs

| Input | Description | Required | Default |
|---|---|---|---|
| `version-level` | Version level to bump: `patch`, `minor`, or `major` | No | `patch` |
| `working-directory` | Path to the directory containing `Cargo.toml` | No | `.` |
| `commit-changes` | Whether to commit the version bump changes | No | `true` |
| `commit-message` | Commit message. Use `{version}` as a placeholder for the new version | No | `Bump version to {version} [skip ci]` |
| `skip-push` | Skip pushing the commit and tag to the remote (useful for testing) | No | `false` |
| `workspace` | Apply version bump to all workspace members | No | `true` |

## Outputs

| Output | Description |
|---|---|
| `version` | The new version number (e.g. `1.2.3`) |
| `tag` | The created git tag (e.g. `v1.2.3`) |

## Examples

### Basic patch release

```yaml
name: Cut Release

on:
  workflow_dispatch:

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: p6m-actions/rust-cut-tag@v1
```

### Minor release with custom commit message

```yaml
- uses: p6m-actions/rust-cut-tag@v1
  with:
    version-level: minor
    commit-message: "chore: release {version}"
```

### Use the outputs in a subsequent step

```yaml
- id: cut-tag
  uses: p6m-actions/rust-cut-tag@v1
  with:
    version-level: patch

- name: Print new version
  run: echo "Released ${{ steps.cut-tag.outputs.tag }}"
```

### Non-root workspace

```yaml
- uses: p6m-actions/rust-cut-tag@v1
  with:
    working-directory: services/my-service
```

### Dry run (bump locally without pushing)

```yaml
- uses: p6m-actions/rust-cut-tag@v1
  with:
    skip-push: true
```

## How it works

1. Runs `p6m-actions/token-exchange` to configure git credentials for pushing.
2. Installs `cargo-release` via `cargo install`.
3. Runs `cargo release version <level> --execute --no-confirm` to bump version numbers in `Cargo.toml` (and all workspace members when `workspace: true`).
4. Reads the new version from `cargo metadata` (falls back to parsing `Cargo.toml` directly).
5. Commits the changed files with the configured commit message (when `commit-changes: true`).
6. Creates an **annotated** git tag (`v<version>`).
7. Pushes the commit (using `--force-with-lease`) and the tag to the remote (unless `skip-push: true`).
