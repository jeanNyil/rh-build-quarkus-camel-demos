# Camel-Quarkus-Http

This project leverages **Red Hat Build of Quarkus version 1.3**, the Supersonic Subatomic Java Framework.

It exposes the following RESTful service endpoints  using the **Apache Camel Quarkus extensions**: 
- `/fruits` : returns a list of hard-coded fruits (`name` and `description`) in JSON format. It also allows to add a `fruit` though the `POST` HTTP method
- `/legumes` : returns a list of hard-coded legumes (`name`and `description`) in JSON format.
- `/health` : returns the Camel Quarkus MicroProfile metrics 
- `/metrics` : the Camel Quarkus MicroProfile health checks

## Prerequisites
- JDK 11 installed with `JAVA_HOME` configured appropriately
- Apache Maven 3.6.2+

## Running the application in dev mode

You can run your application in dev mode that enables live coding using:
```
./mvnw quarkus:dev
```

## Packaging and running the application locally

The application can be packaged using `./mvnw package`.
It produces the `camel-quarkus-http-1.0-SNAPSHOT-runner.jar` file in the `/target` directory.
Be aware that it’s not an _über-jar_ as the dependencies are copied into the `target/lib` directory.

The application is now runnable using `java -jar target/camel-quarkus-http-1.0-SNAPSHOT-runner.jar`.

## Packaging and running the application on Red Hat OpenShift

### Pre-requisites
- Access to a [Red Hat OpenShift](https://access.redhat.com/documentation/en-us/openshift_container_platform) cluster v3 or v4
- User has self-provisioner privilege or has access to a working OpenShift project

1. Login to the OpenShift cluster
    ```zsh
    oc login ...
    ```
2. Create an OpenShift project or use your existing OpenShift project. For instance, to create `camel-quarkus`
    ```zsh
    oc new-project camel-quarkus --display-name="Apache Camel Quarkus Apps"
    ```
3. Use either the _**S2I binary workflow**_ or _**S2I source workflow**_ to deploy the `camel-quarkus-http` app as described below.

### OpenShift S2I binary workflow 

This leverages the _Quarkus OpenShift_ extension and is only recommended for development and testing purposes.

```zsh
./mvnw clean package -Dquarkus.kubernetes.deploy=true
```
```zsh
[...]
[INFO] [io.quarkus.deployment.pkg.steps.JarResultBuildStep] Building thin jar: /Users/jeannyil/Workdata/myGit/Quarkus/rh-build-quarkus-camel-demos/camel-quarkus-http/target/camel-quarkus-http-1.0-SNAPSHOT-runner.jar
[INFO] [io.quarkus.kubernetes.deployment.KubernetesDeploy] Kubernetes API Server at 'https://api.cluster-fc38.sandbox840.opentlc.com:6443/' successfully contacted.
[...]
[INFO] [io.quarkus.container.image.s2i.deployment.S2iProcessor] Performing s2i binary build with jar on server: https://api.cluster-fc38.sandbox840.opentlc.com:6443/ in namespace:camel-quarkus.
[...]
[INFO] [io.quarkus.container.image.s2i.deployment.S2iProcessor] Successfully pushed image-registry.openshift-image-registry.svc:5000/camel-quarkus/camel-quarkus-http@sha256:3b09519ea46094a6e5c9381e67956cb64697e0151efe1c64fda89b2756f14386
[INFO] [io.quarkus.container.image.s2i.deployment.S2iProcessor] Push successful
[INFO] [io.quarkus.kubernetes.deployment.KubernetesDeployer] Deploying to openshift server: https://api.cluster-fc38.sandbox840.opentlc.com:6443/ in namespace: camel-quarkus.
[...]
```

### OpenShift S2I source workflow (recommended for PRODUCTION use)

1. Make sure the latest supported OpenJDK 11 image is imported in OpenShift
    ```zsh
    oc import-image --confirm openjdk/openjdk-11-rhel7 \
    --from=registry.access.redhat.com/openjdk/openjdk-11-rhel7 \
    -n openshift
    ```
2. Create the `camel-quarkus-http` OpenShift application from the git repository
    ```zsh
    oc new-app https://github.com/jeanNyil/rh-build-quarkus-camel-demos.git \
    --context-dir=camel-quarkus-http \
    --name=camel-quarkus-http \
    --image-stream="openshift/openjdk-11-rhel7"
    ```
3. Follow the log of the S2I build
    ```zsh
    oc logs bc/camel-quarkus-http -f
    ```
    ```zsh
    Cloning "https://github.com/jeanNyil/rh-build-quarkus-camel-demos.git" ...
        Commit: be8d82141951e547ac05fc7cba2aeaa2f163ee64 (Added Camel-Quarkus-Http demo project)
        Author: Jean Armand Nyilimbibi <jean.nyilimbibi@gmail.com>
        Date:   Fri Jul 31 17:41:52 2020 +0200
    [...]
   Successfully pushed image-registry.openshift-image-registry.svc:5000/camel-quarkus/camel-quarkus-http@sha256:1d551cc7a9d2aeab55ae202eaef1cc5f39843f72d08939a9e692783670766d33
Push successful
    ```
4. Create a non-secure route to expose the camel-quarkus-http service outside the OpenShift cluster
    ```zsh
    oc expose svc/camel-quarkus-http
    ```

## Testing the application on OpenShift

1. Get the OpenShift route hostname
    ```zsh
    URL="http://$(oc get route camel-quarkus-http -o jsonpath='{.spec.host}')"
    ```
2. Test the `/fruits` endpoint
    ```zsh
    curl -w '\n' $URL/fruits
    ```
    ```json
    [ {
        "name" : "Apple",
        "description" : "Winter fruit"
    }, {
        "name" : "Pineapple",
        "description" : "Tropical fruit"
    }, {
        "name" : "Mango",
        "description" : "Tropical fruit"
    }, {
        "name" : "Banana",
        "description" : "Tropical fruit"
    } ]
    ```
3. Test the `/legumes` endpoint
    ```zsh
    curl -w '\n' $URL/legumes
    ```
    ```json
    [ {
        "name" : "Carrot",
        "description" : "Root vegetable, usually orange"
    }, {
        "name" : "Zucchini",
        "description" : "Summer squash"
    } ]
    ```
4. Test the `/health` endpoint
    ```zsh
    curl -w '\n' $URL/health
    ```
    ```json
    {
        "status": "UP",
        "checks": [
            {
                "name": "camel",
                "status": "UP",
                "data": {
                    "contextStatus": "Started",
                    "name": "camel-quarkus-http"
                }
            },
            {
                "name": "camel-liveness-checks",
                "status": "UP",
                "data": {
                    "route:legumes-restful-route": "UP",
                    "route:fruits-restful-route": "UP"
                }
            },
            {
                "name": "camel",
                "status": "UP",
                "data": {
                    "contextStatus": "Started",
                    "name": "camel-quarkus-http"
                }
            },
            {
                "name": "camel-readiness-checks",
                "status": "UP",
                "data": {
                    "route:legumes-restful-route": "UP",
                    "route:fruits-restful-route": "UP"
                }
            }
        ]
    }
    ```
5. Test the `/metrics` endpoint
    ```zsh
    curl -w '\n' $URL/metrics
    ```
    ```zsh
    [...]
    # HELP application_camel_context_exchanges_total The total number of exchanges for a route or Camel Context
    # TYPE application_camel_context_exchanges_total counter
    application_camel_context_exchanges_total{camelContext="camel-quarkus-http"} 9.0
    [...]
    # HELP application_camel_route_exchanges_total The total number of exchanges for a route or Camel Context
    # TYPE application_camel_route_exchanges_total counter
    application_camel_route_exchanges_total{camelContext="camel-quarkus-http",routeId="fruits-restful-route"} 7.0
    application_camel_route_exchanges_total{camelContext="camel-quarkus-http",routeId="legumes-restful-route"} 2.0
    [...]
    ```

## Testing using [Postman](https://www.postman.com/)

Import the provided Postman Collection for testing: [tests/Camel-Quarkus-Http.postman_collection.json](./tests/Camel-Quarkus-Http.postman_collection.json) 
![Camel-Quarkus-Http.postman_collection.png](../_images/Camel-Quarkus-Http.postman_collection.png)

## Creating a native executable

You can create a native executable using: `./mvnw package -Pnative`.

Or, if you don't have GraalVM installed, you can run the native executable build in a container using: `./mvnw package -Pnative -Dquarkus.native.container-build=true`.

You can then execute your native executable with: `./target/camel-quarkus-http-1.0-SNAPSHOT-runner`

If you want to learn more about building native executables, please consult https://quarkus.io/guides/building-native-image.