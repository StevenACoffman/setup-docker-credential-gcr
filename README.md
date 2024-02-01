# GitHub Action to install and setup [`ko`](https://github.com/GoogleCloudPlatform/docker-credential-gcr)

[![Build](https://github.com/StevenACoffman/setup-docker-credential-gcr/actions/workflows/use-action.yaml/badge.svg)](https://github.com/StevenACoffman/setup-docker-credential-gcr/actions/workflows/use-action.yaml)
<a href="https://gcr.io"><img src="https://avatars2.githubusercontent.com/u/21046548?s=400&v=4" height="120"/></a>

This action will just install [docker-credential-gcr](https://github.com/GoogleCloudPlatform/docker-credential-gcr). 

### What is docker-credential-gcr?
[docker-credential-gcr](https://github.com/GoogleCloudPlatform/docker-credential-gcr) [Google Container Registry](https://cloud.google.com/container-registry/)'s _standalone_, `gcloud` SDK-independent Docker credential helper. It allows for **v18.03+ Docker clients** to easily make authenticated requests to GCR's repositories (gcr.io, eu.gcr.io, etc.).

**Note:** `docker-credential-gcr` is primarily intended for users wishing to authenticate with GCR in the **absence of `gcloud`**, though they are [not mutually exclusive](#gcr-credentials). For normal development setups, users are encouraged to use [`gcloud auth configure-docker`](https://cloud.google.com/sdk/gcloud/reference/auth/configure-docker), instead.

The helper implements the [Docker Credential Store](https://docs.docker.com/engine/reference/commandline/login/#/credentials-store) API, but enables more advanced authentication schemes for GCR's users. In particular, it respects [Application Default Credentials](https://developers.google.com/identity/protocols/application-default-credentials) and is capable of generating credentials automatically (without an explicit login operation) when running in App Engine or Compute Engine.

For even more authentication options, see GCR's documentation on [advanced authentication methods](https://cloud.google.com/container-registry/docs/advanced-authentication).


## Example usage

```yaml
name: Publish

on:
  push:
    branches: ['main']

jobs:
  publish:
    name: Publish
    runs-on: ubuntu-latest
    steps:
      - uses: actions/setup-go@v4
        with:
          go-version: '1.20.x'
      - uses: actions/checkout@v3
      - uses: GoogleCloudPlatform/docker-credential-gcr@v0.6
      - name: 'Configure GCP Artifact Registry'
        run: docker-credential-gcr configure-docker --registries="us-central1-docker.pkg.dev"

```

_That's it!_ This workflow will install docker-credential-gcr so you can use ko or docker (or whatever).

### Select `docker-credential-gcr` version to install

By default, `setup-docker-credential-gcr` installs the [latest released version of `ko`](https://github.com/GoogleCloudPlatform/docker-credential-gcr/releases).

You can select a version with the `version` parameter:

```yaml
- uses: GoogleCloudPlatform/docker-credential-gcr@v0.6
  with:
    version: v0.14.1
```

To build and install `ko` from source using `go install`, specify `version: tip`.

### Pushing to other registries

By default, `GoogleCloudPlatform/docker-credential-gcr` configures `ko` to push images to [GitHub Container Registry](https://ghcr.io), but you can configure it to push to other registries as well.

If `KO_DOCKER_REPO` is already set when `setup-ko` runs, it will skip logging in to ghcr.io and will propagate `KO_DOCKER_REPO` for subsequent steps.

To do this, you should provide credentials to authorize the push.
You can use [encrypted secrets](https://docs.github.com/en/actions/reference/encrypted-secrets) to store the authorization token, and pass it to `ko login` before pushing:

You probably want to [set up Workload Identity](https://github.com/google-github-actions/auth#usage) between your GitHub Actions workflow and your GCP project.



```yaml
steps:
...
- uses: StevenACoffman/setup-docker-credential-gcr@v0.6
- name: 'Configure Docker Credentials for Artifact Registry'
  run: |
    docker-credential-gcr configure-docker --registries="us-central1-docker.pkg.dev"
```

### Why?

`gcloud` is great.
It's like a Swiss army knife for the cloud, if a knife could do anything to a cloud.

It does so much.
_It does soooo much!_

Too much.

This leads it to be really huge.
Hundreds of megabytes of sweet delicious Python.
Python that has to be interpreted before it can even start running anything.

If you're downloading and installing and running `gcloud` just to execute `docker-credential-gcr configure-docker ` so you can switch to using `ko` -- _especially_ in a CI environment -- you're wasting a lot of time.

Installing `gcloud` can take _minutes_, compared to just a few seconds with `setup-docker-credential-gcr`, even if you have to build it from source.
