# Weaveworks Sock Shop OCM

This is a repository containing a sample application to be deployed using Flux plus OCM.

The component that will be deployed is the Weaveworks Sock-Shop application.

This is the cloud native approach to using OCM with flux plus a kind cluster.

## Controllers

There are several controllers that are involved in the installation and the setup process.
These are:

- [ocm-controller](https://github.com/open-component-model/ocm-controller)
- [replication-controller](https://github.com/open-component-model/replication-controller)
- [diffusion-controller](https://github.com/open-component-model/diffusion-controller)

Deploying them is not part of the Flux charts, they should already be present in the cluster.

## Flux

Flux needs to be bootstrapped onto a cluster. To do that, follow flux's [Getting Started guide](https://fluxcd.io/flux/get-started/).

### Installation

There are multiple ways to get a controller into the cluster. The three main approaches are:

- running them locally
- pushing them into a local oci repository
- pushing the changes into their public ghcr.io repository

#### Running them locally

To run them locally, simply clone the repository and then run:

```bash
make && make install
./bin/manager -zap-log-level 4
```

Bare in mind, if you do this with multiple controllers, you will have to re-define the port that they expose such as
metrics and health ports since they will collide with each other.

To do that, just run:

```
./bin/manager -zap-log-level 4 --metrics-bind-address :8083 --health-probe-bind-address :8084
```

And then increase as needed.

#### Local Registry

A simpler and cleaner approach is using a local registry. This requires a re-write of the image address, but this
approach will also test RBAC access of the controller and proper in-cluster communication.

To do this, start up a local kind cluster using a local registry described [here](https://kind.sigs.k8s.io/docs/user/local-registry/).

Once the registry is alive, you can build and push images to it with:

```bash
# build
docker build -t localhost:5001/open-component-model/ocm-controller:v0.0.1 .
# push
docker push localhost:5001/open-component-model/ocm-controller:v0.0.1
```

And edit the image tag in the deployment yaml to point to the above localized controller image url.

Before the next step, make sure to run `make install && make deploy` on each controller so the CRDs and RBAC permissions
and service account is installed. `make deploy` will also configure the deployment and namespace for the respective
controller.

To update the image name, edit the kustomization.yaml under `manager/kustomization.yaml` such as:

```yaml
- name: open-component-model/ocm-controller
  newName: localhost:5001/open-component-model/ocm-controller
  newTag: v0.0.1
```

#### ghcr.io

This requires pushing access to the public repository of these controllers.
