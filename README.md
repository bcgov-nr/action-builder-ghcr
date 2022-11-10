<!-- Badges -->
[![Issues](https://img.shields.io/github/issues/bcgov-nr/action-conditional-container-builder)](/../../issues)
[![Pull Requests](https://img.shields.io/github/issues-pr/bcgov-nr/action-conditional-container-builder)](/../../pulls)
[![MIT License](https://img.shields.io/github/license/bcgov-nr/action-conditional-container-builder.svg)](/LICENSE)
[![Lifecycle](https://img.shields.io/badge/Lifecycle-Experimental-339999)](https://github.com/bcgov/repomountie/blob/master/doc/lifecycle-badges.md)

# Conditional Container Builder with Fallback

This action builds Docker/Podman containers conditionally using a set of directories.  If any files were changed matching that, then build a container.  If those files were not changed, retag an existing build.

This is useful in CI/CD pipelines where not every package/app needs to be rebuilt.

This tool is currently strongly opinionated and generatess images with a rigid structure below.  This is intended to become more flexible in future.

Package name: `<organization>/<repository>/<package>:<tag>`

Pull with: `docker pull ghcr.io/<organization>/<repository>/<package>:<tag>` 

Only GitHub Container Registry (ghcr.io) is supported so far.

# Usage

```yaml
- uses: bcgov-nr/action-conditional-container-builder@v1
  with:
    ### Required

    # Package name
    package: frontend

    # Tag name (<package>:<tag>)
    tag: ${{ github.event.number }}

    # Usually ${{ secrets.GITHUB_TOKEN }}, but a personal access token works too
    token: ${{ secrets.GITHUB_TOKEN }}


    ### Typical / recommended

    # Fallback tag, used if no build was generated
    # Non-matching tags do nothing, which forces a build
    # Defaults to nothing, which also forces a build
    tag_fallback: test

    # Bash array to diff for build triggering
    # Defaults to nothing, which forces a build
    triggers: ('frontend/')


    ### Usually a bad idea / not recommended

    # Overrides the default branch to diff against
    # Defaults to the default branch, usually `main`
    diff_branch: ${{ github.event.repository.default_branch }}

    # Repository to clone and process
    # Useful for consuming other repos, like in testing
    # Defaults to the current one
    repository: ${{ github.repository }}
```

# Example, Single Build

Build a single subfolder with a Dockerfile in it.  Runs on pull requests (PRs).

Create or modify a GitHub workflow, like below.  E.g. `./github/workflows/pr-open.yml`

```yaml
name: Pull Request

on:
  pull_request:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  builds:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v3
      - name: Builds
          uses: bcgov-nr/action-conditional-container-builder@v1
          with:
            package: frontend
            tag: ${{ github.event.number }}
            tag_fallback: test
            token: ${{ secrets.GITHUB_TOKEN }}
            triggers: ('frontend/')
```

# Example, Matrix Build

Build from multiple subfolders with Dockerfile in them.  This time an outside repository is used.  Runs on pull requests (PRs).

Create or modify a GitHub workflow, like below.  E.g. `./github/workflows/pr-open.yml`

```yaml
name: Pull Request

on:
  pull_request:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  builds:
    runs-on: ubuntu-22.04
    strategy:
      matrix:
        package: [backend, frontend]
        include:
          - package: backend
            triggers: ('backend/')
          - package: frontend
            triggers: ('frontend/')
    steps:
      - uses: actions/checkout@v3
      - name: Test Builds
        uses: ./
        with:
          package: ${{ matrix.package }}
          tag: ${{ github.event.number }}
          tag_fallback: test
          repository: bcgov/nr-quickstart-typescript
          token: ${{ secrets.GITHUB_TOKEN }}
          triggers: ${{ matrix.triggers }}

```

# Output

If a build has been generated this action will output 'true`.

```yaml
- id: meaningful_id_name
  uses: bcgov-nr/action-conditional-container-builder@v1
  ...

- if: steps.meaningful_id_name.outputs.build == 'true'
  ...
```

# Acknowledgements

This Action is provided courtesty of the Forestry Suite of Applications, part of the Government of British Columbia.
