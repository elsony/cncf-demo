# Build Container Image With Cloud Native Buildpacks (CNB)

TODO: Intro

## Setup

* [Install the `pack` CLI](https://buildpacks.io/docs/tools/pack/#pack-cli)
* Make sure that Docker is up and running

## Do

```bash
export TAG=v0.0.1

pack build cncf-demo:$TAG

docker image ls
```

## Continue The Adventure

[Store Container Image In A Registry](../registry/story.md)