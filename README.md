# Common Workflows

![GitHub release (with filter)](https://img.shields.io/github/v/release/sentrysoftware/workflows)
![GitHub top language](https://img.shields.io/github/languages/top/sentrysoftware/workflows)
![License](https://img.shields.io/github/license/sentrysoftware/workflows)

This repository provides common workflows to be used in GitHub actions in repositories in GitHub.

## Maven Build

The `maven-build.yml` workflow simply builds a Maven project with the `mvn verify` goal. It's designed to be invoked on `push` on PRs, so that this action is considered as a *Check* on the PR, i.e. it will prevent the build from being merged if the build fails.

### Inputs

| Input | Description | Default value |
|---|---|---|
| `jdkVersion` | Version of the JDK to setup to run Maven. See [supported syntax](https://github.com/actions/setup-java#supported-version-syntax). | `17` |
| `nodeVersion` | Version of the NodeJS to setup for this project. **Optional** (if not specified, NodeJS won't be installed). See [supported syntax](https://github.com/actions/setup-node#supported-version-syntax). | *None* |
| `debug` | Whether to run Maven in debug mode (with `-X` option) | `"false" |
| `ssh` | Whether to open a SSH session in the GitHub Actions runner, to let you interact with the system that performed the build. **SSH session automatically closes after 10 minutes of inactivity.** | `"false"` |

### How to use it?

To use this workflow, you must define a `job` that `uses: sentrysoftware/workflows/.github/workflows/maven-build.yml@main`. 

> [!IMPORTANT]
> Instead of `@main`, it is recommended to use an actual version, to avoid breaking your build if the workflow required inputs change in the future, like `@v1`.

To run this workflow as an automatic check in the PRs or a repository, and also allow users to trigger the workflow manually (with debug options), create a `.github/workflows/build.yml` with the below content:

```yaml
name: Maven Build

on:
  # Run on all PRs to "main" (update to include the protected branches of your repository)
  pull_request:
    branches: [ "main" ]

  # Run the workflow manually
  workflow_dispatch:
    inputs:
      debug:
        type: boolean
        description: Maven debug mode
        required: false
        default: false
      ssh:
        type: boolean
        description: Open SSH session in the runner
        required: false
        default: false
        
jobs:
  build:
    # Call this shared workflow (use `@v1` to use a specific version, instead of the latest)
    uses: sentrysoftware/workflows/.github/workflows/maven-build.yml@main
    with:
      jdkVersion: "17"
      nodeVersion: "20.x"
      debug: ${{ github.event_name == 'workflow_dispatch' && inputs.debug }}
      ssh: ${{ github.event_name == 'workflow_dispatch' && inputs.ssh }}
```

