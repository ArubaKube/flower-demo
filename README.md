# Federated Learning on a multi-cluster environment powered by Liqo and Flower FL framework

The repo contains a demo of Federated Learning app deployed on a multi-cluster environment with **[Liqo](https://liqo.io)** and the popular FL framework **[Flower](https://flower.ai/)**.

The deployed app is a simple ML model trained on the [CIFAR-10](https://www.cs.toronto.edu/~kriz/cifar.html) dataset using a simple CNN scratched with [PyTorch](https://pytorch.org/).

## Overview

The demo leverages a multi-cluster environment to run a distributed FL training. 
To setup the environment we need a:
- a central cluster: 
    - acts as a server
    - pilots the application (offloads client pods to the leaf clusters)
    - expose a private Service (*ClusterIP*) for the client to connects
    - aggregrate the results and hosts the global model
- N leaf clusters:
    - act as clients
    - train their local model using local (sensitive) data
    - share the updated weights to server by accessing the server Service

Architecture overview:

![architecture overview](/static/images/architecture-overview.png)

## Build the images

```bash
docker build -f ./build/Dockerfile.superexec -t <IMAGE_NAME>:<IMAGE_VERSION> ./demo
docker build -f ./build/Dockerfile.clientapp -t <IMAGE_NAME>:<IMAGE_VERSION> ./demo
```

## Environment configuration

To bootstrap the environment you need 1 cluster acting as a server, and N acting as clients.

Requirements:
- [install](https://docs.liqo.io/en/latest/installation/install.html) Liqo on all clusters
- enable the liqo `RuntimeClass` [feature](https://docs.liqo.io/en/latest/usage/namespace-offloading.html#runtimeclass).
This is not strictly required, you can keep the RuntimeClass off, but you need to modify the chart adding NodeAffinities to all deployments/statefulsets (i.e., deploy `server` on local nodes, while `clients` on liqo virtual nodes)
- peer the central cluster (server) with all N client clusters. 
The central cluster acts as a *consumer*, while the leaf clusters acts as a *provider*  (no bidirectional peering is required).
Refer to the official [docs](https://docs.liqo.io/en/latest/features/peering.html) for more info.
- install [flwr cli](https://flower.ai/docs/framework/how-to-install-flower.html)


## Deploy the app

On the central cluster (the one acting as a server) run:

```bash
kubectl create ns flower-demo
liqoctl offload namespace flower-demo --namespace-mapping-strategy EnforceSameName
kubectl apply -f ./deploy/manifests -n flower-demo
```

Note: in the `manifests/clients.yaml` file make sure the number of replicas of the *StatefulSet* and the `NUM_CLIENT` env variable are equal to the number of clients (peered clusters).

## Run the app

On the central cluster, expose the server app endpoint (port 9093):

```bash
kubectl port-forward pods/<SERVER_POD> 9093:9093
```

Now you are ready to run the training with:

```bash
flwr run ./demo liqo
```

