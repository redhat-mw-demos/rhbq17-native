# rhbq17-native project

This project showcases to package a native executable using Mandrel base image then deploy to OpenShift as Knative Service.

If you want to learn more about Quarkus, please visit its website: https://access.redhat.com/documentation/en-us/red_hat_build_of_quarkus .

## Running the application in dev mode

You can run your application in dev mode that enables live coding using:
```
./mvnw quarkus:dev
```

## Edit an application.properties

In order to package a native binary based on mandrel image:
```
quarkus.native.container-build=true
quarkus.native.builder-image=quay.io/quarkus/ubi-quarkus-mandrel:20.1-java11
```

## Creating a native executable

You can create a native executable using: `./mvnw package -Pnative`.

## Build using OC CLI

Make sure to log in the OpenShift Cluster. Then create a new project:

```
oc new-project rhbq17-native
```

Run the following commands:

```
oc new-build quay.io/quarkus/ubi-quarkus-native-binary-s2i:1.0 --binary --name=native -l app=native

oc start-build native --from-file=target/rhbq17-native-1.0.0-SNAPSHOT-runner --follow
```

## Dev Console in OCP cluster

Navigate Developer Console > +Add > Container Image then select `Image stream tag from internal registry`:

 * Project : rhbq17-native
 * ImageStreams : native
 * Tag : latest

Select `Knative Service` in Resources then click on `Create`.

Once the Knative service is up in Topology view, click on `Open URL`. Then access the endpoint(i.e. _/hello_), you should see `hello`.

## Change the code

Open the `ExampleResource.java` then repalce `hello` with `Congrats! Red Hat Build Of Quarkus 1.7 GA!` in the return code.

## Re-Build using OC CLI

Make sure to log in the OpenShift Cluster. Run the following commands:
```
oc new-build quay.io/quarkus/ubi-quarkus-native-binary-s2i:1.0 --binary --name=native -l app=native-2

oc start-build native-2 --from-file=target/rhbq17-native-1.0.0-SNAPSHOT-runner --follow
```

## Update the existing Knative Service using KN CLI

Find the ImageStream that you just pushed:

```
oc get is
```

You should see like this:

```
NAME                            IMAGE REPOSITORY                                                                               TAGS     UPDATED
native                          image-registry.openshift-image-registry.svc:5000/rhbq17-native/native                          latest   20 minutes ago
native-2                        image-registry.openshift-image-registry.svc:5000/rhbq17-native/native-2                        latest   14 minutes ago
ubi-quarkus-native-binary-s2i   image-registry.openshift-image-registry.svc:5000/rhbq17-native/ubi-quarkus-native-binary-s2i   1.0      21 minutes ago
```

Update the knative service:

```
kn service update native --image image-registry.openshift-image-registry.svc:5000/rhbq17-native/native-2:latest
```

## Reload the webpage

You should see in the web page:

```
Congrats! Red Hat Build Of Quarkus 1.7 GA!
```