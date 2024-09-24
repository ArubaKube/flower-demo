# Federated Learning on a multi-cluster environment powered by Liqo and Flower FL framework

The repo contains a demo of Federated Learning app deployed on a multi-cluster environment with **[Liqo](https://liqo.io)** and the popular FL framework **[Flower](https://flower.ai/)**.

The deployed app is a simple ML model trained on the [CIFAR-10](https://www.cs.toronto.edu/~kriz/cifar.html) dataset using a simple CNN scratched with [PyTorch](https://pytorch.org/).

## Overview

## Build the images

```bash
docker build -f ./build/Dockerfile.superexec -t <IMAGE_NAME>:<IMAGE_VERSION> ./demo
docker build -f ./build/Dockerfile.clientapp -t <IMAGE_NAME>:<IMAGE_VERSION> ./demo
```

## Environment configuration

To bootstrap the environment you need 1 cluster acting as a server, and N acting as clients.

Requirements:
- liqo is installed on all clusters
- enable the liqo `RuntimeClass`. This is not strictly required, you can keep the RuntimeClass off, but you need to modify the chart adding NodeAffinities to all deployments/statefulsets (i.e., deploy `superexec` and `superlink` on local nodes, while `supernodes` on liqo virtual nodes)
- peer the central cluster (server) with all N client clusters (no bidirectional peering is required)

## Deploy the app

On the central cluster (the one acting as a server) run:

```bash
kubectl create ns demo
liqoctl offload namespace demo --namespace-mapping-strategy EnforceSameName
kubectl apply -f ./manifests
```

Note: in the `manifests/supernodes.yaml` file make sure the number of replicas of the *StatefulSet* and the `NUM_CLIENT` env variable are equal to the number of clients (peered clusters).

## Run the app

Prerequisite: install [flwr cli](https://flower.ai/docs/framework/how-to-install-flower.html)

```bash
flwr run ./demo liqo
```

