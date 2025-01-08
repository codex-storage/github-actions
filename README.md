# GitHub Actions helpers

 We are using [GitHub Actions](https://github.com/features/actions) for binaries and Docker builds and sometimes we can reuse existing workflows for similar cases.

 Repository contains [reusable workflows](https://docs.github.com/en/actions/sharing-automations/reusing-workflows) which can be used across other projects.


## Workflows

| Workflow                                | File                                                            | Description               |
| --------------------------------------- | --------------------------------------------------------------- | ------------------------- |
| [Docker - Reusable](#docker---reusable) | [`docker-reusable.yml`](/.github/workflows/docker-reusable.yml) | Docker amd64/arm64 builds |


### Docker - Reusable

 This workflow is used for multi-arch builds and it uses native runner, because QEMU is slower.


#### Inputs

| Variable         | Description                        |
| ---------------- | ---------------------------------- |
| `docker_file`    | Path to Dockerfile                 |
| `dockerhub_repo` | DockerHub repository               |
| `build_args`     | Build arguments                    |
| `tag_latest`     | Set latest tag for Docker images   |
| `tag_sha`        | Set Git short commit as Docker tag |
| `tag_suffix`     | Suffix for Docker images tag       |
| `amd64_builder`  | Builder for amd64                  |
| `arm64_builder`  | Builder for arm64                  |


#### Secrets

 Workflow uses `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN` secrets to push images to the DockerHub.

 Also, we can pass secrets required by `build_args`. For that, we should set arg with a value as a variable of the secret name
 ```yaml
 VITE_CODEX_API_URL=${VITE_CODEX_API_URL}
 ```


#### How to use

 1. If we are using non GitHub-hosted runners (for ARM builds), we should configure repository to use them.

 2. Create DockerHub related secrets in the repository
    ```shell
    DOCKERHUB_USERNAME
    DOCKERHUB_TOKEN
    ```

 3. Add a workflow to the repository, with the content from examples and adjust for you needs
    ```shell
    .github/workflows/docker.yml
    ```


#### Examples
```yaml
name: Docker


on:
  push:
    branches:
      - master
    tags:
      - 'v*.*.*'
  workflow_dispatch:


jobs:
  build-and-push:
    name: Build and Push
    uses: codex-storage/github-actions/.github/workflows/docker-reusable.yml@master
    with:
      docker_file: docker/Dockerfile
      dockerhub_repo: codexstorage/test
      tag_latest: ${{ github.ref_name == github.event.repository.default_branch || startsWith(github.ref, 'refs/tags/') }}
      amd64_builder: ubuntu-24.04
      build_args: |
        VITE_CODEX_API_URL=${VITE_CODEX_API_URL}
        VITE_GEO_IP_URL="Plain text"
    secrets: inherit
```