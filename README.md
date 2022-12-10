# color-service

This project is a sample project for [Quarkus](https://quarkus.io/), [Kubernetes](https://kubernetes.io/), [OpenShift](https://www.openshift.com/), [Helm](https://helm.sh), [Tekton](https://tekton.dev/) and [Knative](https://knative.dev/).

## Prerequisites

In order to run this example, you'll need:

* An installation of Git to clone this repository
* A working installation of a JVM
* The Client binaries for
  * OpenShift (oc)
  * KNative
  * Tekton
  * Helm
* Podman if you want to run and build the container locally. Docker should also work, but it has not been tested.
* Access to a Red Hat OpenShift with OpenShift Pipelines (Tekton) and Serverless (Knative) enabled. The installation should work on the [OpenShift Sandbox](https://developers.redhat.com/developer-sandbox), which provides a temporary development environment.

The setup has been tested and verified on a Fedora 37 installation, deploying to OpenShift Sandbox.

When running the commands below, it is assumed that you run them while your shell working directory points to the root of this Git repository.

## Development

### Running the Application in dev mode

You can run your application in dev mode that enables live coding using:

```bash
./mvnw quarkus:dev
```

You can access the application by pointing your browser to http://localhost:8080. In Quarkus Dev mode, you can also explore the internals of your application by navigating to http://localhost:8080/q/dev/.

### Packaging and running the application

The application can be packaged using `./mvnw package`.
It produces the `color-service-<version>.jar` file in the `/target` directory.
Be aware that it’s not an _über-jar_ as the dependencies are copied into the `target/lib` directory.

The application is now runnable using `java -jar target/quarkus-app/quarkus-run.jar`.

### Creating a native executable

You can create a native executable using: `./mvnw package -Pnative`.

Or, if you don't have GraalVM installed, you can run the native executable build in a container using: `./mvnw package -Pnative -Dquarkus.native.container-build=true -Dquarkus.native.container-runtime=podman`.

You can then execute your native executable with: `./target/blue-green-deployment-1.0.0-SNAPSHOT-runner`

If you want to learn more about building native executables, please consult https://quarkus.io/guides/building-native-image.

## Build Container Image

### JVM

```bash
buildah bud -f src/main/docker/Dockerfile.jvm -t quay.io/jritter/color-service:latest
```

### Native

```bash
buildah bud -f src/main/docker/Dockerfile.native -t quay.io/jritter/color-service:latest-native
```

## Run the color-service locally

Color service can be run just as any other container. It has been tested using podman. The container exposes the Quarkus default port 8080 for its webservice:

### JVM

```bash
podman run -p 8080:8080 -e COLOR_SERVICE_COLOR=blue -it quay.io/jritter/color-service:latest
```

### Native

```bash
podman run -p 8080:8080 -e COLOR_SERVICE_COLOR=blue -it quay.io/jritter/color-service:latest-native
```

## Deploying color-service on Kubernetes

It would be nice if we could run this example application on a Kubernetes Cluster. This chapter outlines a bunch of options to deploy this application.

### Helm

This chapter covers the deployment of the color service using [Helm](https://helm.sh).

#### Deploy color-service using Helm

This service can be deployed using a Helm chart.

```bash
helm upgrade --install my-color-service helm/color-service
```

#### Change the color using Helm

```bash
helm upgrade my-color-service helm/color-service --set color=yellow
```

Possible colors are:

* red
* green
* blue
* yellow

## Delete color-service using Helm

```bash
helm uninstall my-color-service
```

### Knative

This chapter covers the deployment of the color service using [Knative](https://knative.dev).

#### Deploy color-service using Knative

```bash
kn service create color-service --image quay.io/jritter/color-service:2.3.0 --request cpu=100m,memory=256Mi --limit cpu=200m,memory=512Mi -l app.openshift.io/runtime=quarkus -a app.openshift.io/vcs-ref=refs/heads/develop -a app.openshift.io/vcs-uri=https://github.com/jritter/color-service -a prometheus.io/scrape=true -a prometheus.io/path=/q/metrics -a prometheus.io/port=8080 -a autoscaling.knative.dev/target=20 -a autoscaling.knative.dev/metric=rps
```

Note how this configuration configures the resources, the Prometheus scrape parameters and the autoscaler.

## Change the color using Knative

```bash
kn service update color-service -e COLOR_SERVICE_COLOR=blue
```

Possible colors are:

* red
* green
* blue
* yellow

## Exploring the autoscaler

```bash
siege -c 10 <url>
```

## Delete color-service using Knative

```bash
kn service delete color-service
```
