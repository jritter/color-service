# color-service

This project is a sample project for [Quarkus](https://quarkus.io/), [Kubernetes](https://kubernetes.io/), [OpenShift](https://www.openshift.com/), [Tekton](https://tekton.dev/) and [Knative](https://knative.dev/).

## Running the application in dev mode

You can run your application in dev mode that enables live coding using:

```bash
./mvnw quarkus:dev
```

## Packaging and running the application

The application can be packaged using `./mvnw package`.
It produces the `color-service-<version>.jar` file in the `/target` directory.
Be aware that it’s not an _über-jar_ as the dependencies are copied into the `target/lib` directory.

The application is now runnable using `java -jar target/quarkus-app/quarkus-run.jar`.

## Creating a native executable

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

## Deploy color-service using Knative

```bash
kn service create color-service --image quay.io/jritter/color-service --request cpu=100m,memory=256Mi --limit cpu=200m,memory=512Mi -l app.openshift.io/runtime=quarkus -a app.openshift.io/vcs-ref=refs/heads/develop -a app.openshift.io/vcs-uri=https://github.com/jritter/color-service -a prometheus.io/scrape=true -a prometheus.io/port=8080 -a autoscaling.knative.dev/target=20 -a autoscaling.knative.dev/metric=rps
```

## Change the color using Knative

```bash
kn service update color-service -e COLOR_SERVICE_COLOR=blue
```

Possible colors are:

* red
* green
* blue
* yellow

## Delete color-service using Knative

```bash
kn service delete color-service
```
