# Store Container Image in a Registry with Dragonfly

Dragonfly is a file distribution system based on peer-to-peer (P2P) networking.

### What is peer-to-peer networking?

In contrast to conventional client-server network systems, with P2P networking all of the computers are connected to one another, and also through a single server computer. The role of the server is usually to help the computers ("peers") find one another. After the peers are found, they communicate directly - the traffic between the peers will not pass through the main server. Regarding sharing information within the network, each device serves the purpose of both the server and the client. Each peer plays an independent role, and has access to save the entire set of shared data in its own database. This enables the speedy transfer of files to and from anywhere in the system.

### Nydus

Dragonfly additionally has a sub-project called [Nydus](https://nydus.dev/) that is an open source filesystem solution that aids in high-efficiency image distribution.

The Dragonfly/Nydus combo is powerful. 

First because **Nydus-formatted images are made up of smaller pieces that can be moved around independently of one another.** This combined with using Dragonfly's P2P network can take advantage of every node’s bandwidth, greatly reducing image pull time.

Second because **Nydus knows which part of the image is essential for the start-up of the container** - maybe only 50 MB of data of a 1 GB image - **and it will begin by just pulling that part so the container can be started.** In this way it greatly reduces the container startup time. This is also good for serverless scenerios.  

It is important to note here that the Nydus image format is different from the OCI image format - in order to take advantage of the benefits of Nydus, your OCI images will need to be converted to Nydus images. However the Nydus ecosystem has a number of tools to help with the conversion, such as [a CLI tool](https://github.com/dragonflyoss/image-service/blob/master/docs/nydusify.md), a [Harbor acceleration service](https://github.com/goharbor/acceleration-service), and [Buildkit](https://github.com/moby/buildkit/issues/2046) provides the ability to build and export Nydus images directly from a Dockerfile.

## Setup

```bash
export REGISTRY=index.docker.io

# Replace `[...]` with your Docker Hub username
export DOCKERHUB_USERNAME=[...]

export IMAGE=$REGISTRY/$DOCKERHUB_USERNAME/cncf-demo

yq --inplace ".image = \"$IMAGE\"" settings.yaml

# Make sure that Docker is running

docker login --username $DOCKERHUB_USERNAME $REGISTRY

# Create the `values.yaml` configuration file with different container runtime,
# refer to https://d7y.io/docs/setup/install/helm-charts#runtime-configuration-guide-for-dragonfly-helm-chart.
cat > values.yaml << EOF
containerRuntime:
  docker:
    enable: true
    injectHosts: true
    registryDomains:
      - 'index.docker.io'
EOF

helm repo add dragonfly https://dragonflyoss.github.io/helm-charts/

helm repo update

helm install --create-namespace --namespace dragonfly-system dragonfly dragonfly/dragonfly -f values.yaml
```

## Do

```bash
echo $IMAGE

docker image tag cncf-demo:$TAG $IMAGE:$TAG

docker image tag cncf-demo:$TAG ${IMAGE}:latest

docker image push $IMAGE:$TAG

docker image push ${IMAGE}:latest

# Pull image through dragonfly.
docker image pull $IMAGE:$TAG

# Pull image through dragonfly.
docker image pull $IMAGE:latest
```

## Continue The Adventure

[Define And Deploy The App To Dev](../define-deploy-dev/README.md)
