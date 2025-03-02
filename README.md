# Deno Conventional Release

A GitHub Action to generate a release for a Deno project based on
conventional commits.

## Requirements

- A Deno project with a `deno.json` file.
- Conventional commits must be used in the project.
- Currently only supports "angular" preset for `conventional-changelog`.

## Inputs

- `GITHUB_TOKEN`: A GitHub token with write access to the repository.
  - You should be able to provide the `${{ secrets.GITHUB_TOKEN }}` that is
    provided to all workflows by default.

## What it does

- Bumps the version in `deno.json` based on the conventional commits found.
- Generates a changelog based on the conventional commits found.
- Creates a tag with the generated version and changelog.
- Creates a release on GitHub with the tag and generated changelog.

## Usage

Create a new workflow file in `.github/workflows` or edit an existing one.

```yaml
name: Generate Release on Merge

on:
  push:
    branches:
      - main

jobs:
  generate-release:
    name: Generate Release
    runs-on: ubuntu-latest
    permissions:
      contents: write # Required to write to the repository
      id-token: write # Required to authenticate with GitHub API
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: funnylookinhat/action-deno-conventional-release@v1
        with:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Using with JSR Publish

This action can be used with [JSR Publish](https://github.com/denoland/deno_registry_js) to automatically publish a release to the Deno registry.

**Note the name of the workflow below.** It matches the name of the workflow
in the above example.

```yaml
name: Publish to JSR on Release

on:
  workflow_run:
    workflows: ["Generate Release on Merge"]
    types: [completed]

jobs:
  publish:
    runs-on: ubuntu-latest
    # Only publish if generate release was successful.
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    permissions:
      contents: read
      id-token: write # The OIDC ID token is used for authentication with JSR.
    steps:
      - uses: actions/checkout@v4
      - run: npx jsr publish
```
