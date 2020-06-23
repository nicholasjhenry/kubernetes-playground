# Kubernetes Playground

## Setup

1. Install [Docker for Mac (Edge)](https://docs.docker.com/docker-for-mac/edge-release-notes/)
2. Enable Kubernetes (context: `docker-desktop`)
3. Run the following Make target

  make dev.setup

## Tools

- kubectl
- kubectx

## Commands

    kubectx --help
    kubectl get deployments.apps -A

## Definitions

|    Term    | Definition |
| ---------- | ---------- |
| container  | a resource |
| deployment |            |
| node       | a resource |
| pod        | a resource |
| resource   |            |