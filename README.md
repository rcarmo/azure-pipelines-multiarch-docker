# azure-pipelines-multiarch-docker

[![Docker Stars](https://img.shields.io/docker/stars/insightful/ubuntu-gambit.svg)](https://hub.docker.com/r/insightful/ubuntu-gambit)
[![Docker Pulls](https://img.shields.io/docker/pulls/insightful/ubuntu-gambit.svg)](https://hub.docker.com/r/insightful/ubuntu-gambit)
[![](https://images.microbadger.com/badges/image/insightful/ubuntu-gambit.svg)](https://microbadger.com/images/insightful/ubuntu-gambit "Get your own image badge on microbadger.com")
[![](https://images.microbadger.com/badges/version/insightful/ubuntu-gambit.svg)](https://microbadger.com/images/insightful/ubuntu-gambit "Get your own version badge on microbadger.com")
[![Build Status](https://dev.azure.com/ruicarmo/insightfulsystems/_apis/build/status/rcarmo.azure-pipelines-multiarch-docker?branchName=master)](https://dev.azure.com/ruicarmo/insightfulsystems/_build/latest?definitionId=5&branchName=master)

This is a shim/template repo for an Azure DevOps pipeline to build multi-architecture images:

## Pipeline Structure

At the Azure Pipelines level, this creates:

![stages](https://github.com/rcarmo/azure-pipelines-multiarch-docker/blob/master/img/stages.png?raw=true)

* One independent stage for each CPU architecture
* A "wrap-up" stage that runs after all the others that builds and publishes the Docker manifest file

## Why?

Because I needed a simple, re-usable example of how to build multi-architecture images (in, this case, Linux for several different CPU architectures) that I could both re-use and share with customers.

## How?

Images are created atop a local base (in this case Ubuntu) with a corresponding `qemu-user-static` binary embedded, which allows most CI systems to build ARM images atop an `amd64` CPU.

The sample `src/Dockerfile` only installs `zsh` into that image, but the intent is that you can build this up from there.

## Internals

Most of the image generation logic (including embedding QEMU and building the final manifest that allows for automatic discovery of architecture-specific tags) lives inside the `Makefile`.

This is done because:

* Having a `Makefile` allows for easy local testing
* It also allows for easy movement between different CI systems
* The actual architecture tagging (and mapping between different styles of architecture references) can be maintained inside the `Makefile`
* Encapsulating that logic makes the CI YAML files considerably more readable

## Caveats

* This does not cover multi-OS (Linux/Windows) images--the principles are the same, but that needs to split on the CI side to allow for different stages inside different VM agents.
* This only uses Docker Hub (and requires `DOCKER_USERNAME` and `DOCKER_PASSWORD` to be set as private global variables for the pipeline--I recommend using linked variables at the organization level).
* This does _not_ use the built-in Docker actions in Azure Pipelines (because they do not _yet_ support the experimental mode settings that are still required, as of June 2019, to build manifests).
