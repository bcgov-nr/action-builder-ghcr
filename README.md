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
- uses: bcgov-nr/action-conditional-container-builder@v1.0.0
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
    # Optional, defaults to nothing, which forces a build
    # Non-matching or malformed tags are rejected, which also forced a build
    tag_fallback: test

    # Bash array to diff for build triggering
    # Optional, defaults to nothing, which forces a build
    triggers: ('frontend/')

    # Sets the build context/directory, which containt, but not points to a Docker
    # Optional, defaults to package name
    build_context: ./frontend


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
    permissions:
      packages: write
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v3
      - name: Builds
        uses: bcgov-nr/action-conditional-container-builder@v1.0.0
        with:
          package: frontend
          tag: ${{ github.event.number }}
          tag_fallback: test
          token: ${{ secrets.GITHUB_TOKEN }}
          triggers: ('frontend/')
```

# Example, Single Build with build_context

Build an image overriding the build context, which is a directory containing a Dockerfile.

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
    permissions:
      packages: write
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v3
      - name: Builds
        uses: bcgov-nr/action-conditional-container-builder@v1.0.0
        with:
          package: frontend
          build_context: ./
          tag: ${{ github.event.number }}
          tag_fallback: test
          token: ${{ secrets.GITHUB_TOKEN }}
          triggers: ('frontend/')
```

# Example, Single Build with build_context and build_file

Build an image overriding the build context and the build file, which is the Dockerfile.

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
    permissions:
      packages: write
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v3
      - name: Builds
        uses: bcgov-nr/action-conditional-container-builder@v1.0.0
        with:
          package: frontend
          build_context: ./
          build_file: subdir/Dockerfile
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
    permissions:
      packages: write
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
        uses: bcgov-nr/action-conditional-container-builder@v1.0.0
        with:
          package: ${{ matrix.package }}
          tag: ${{ github.event.number }}
          tag_fallback: test
          repository: bcgov/nr-quickstart-typescript
          token: ${{ secrets.GITHUB_TOKEN }}
          triggers: ${{ matrix.triggers }}

```

# Output

If a build has been generated this action will output `true`.

```yaml
- id: meaningful_id_name
  uses: bcgov-nr/action-conditional-container-builder@v1.0.0
  ...

- if: steps.meaningful_id_name.outputs.build == 'true'
  ...
```

# Permissions

Workflows kicked off by Dependabot or a fork run with reduced permissions.  That can be addressed by setting explict permissions for the GITHUB_TOKEN.  If this is not required, then remove the lines below from these examples.

```yaml
permissions:
  packages: write
```

<!-- # Acknowledgements

This Action is provided courtesty of the Forestry Suite of Applications, part of the Government of British Columbia. -->
