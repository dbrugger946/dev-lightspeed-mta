
## Links and snippets for creating and deploying the quarkus app
*for showing different ways to build/deploy during demo*

### Basic steps
- log into OCP and get in the quarkus branch of the demo repo
- create the database service
- build/deploy the quarkus app using any of the following suggested approaches, or via a pipeline  

*This version has the fixed **quarkus** branch*  
https://github.com/dbrugger946/kai-coolstore

https://quarkus.io/guides/deploying-to-kubernetes#defining-a-docker-registry  
https://quarkus.io/guides/deploying-to-kubernetes#openshift  
https://quarkus.io/guides/deploying-to-openshift  
**buildpacks**  
https://developers.redhat.com/articles/2025/08/20/build-container-image-quarkus-project-using-buildpacks  
**native builds / image builds**  
https://quarkus.io/guides/building-native-image  



- Can also do a local app build and use generated dockerfiles to create a container, and then deploy different ways to OCP   
- s2i, tekton, 


### Some example snippets using different approaches
*some approaches will require  changes to application.properties file and adding/deleting extensions in the pom.xml ie. project*  

```
oc new-app --name=coolstore-database openshift/postgresql:latest \
-e POSTGRESQL_USER=quarkus \
-e POSTGRESQL_PASSWORD=quarkus \
-e POSTGRESQL_DATABASE=coolstore   
```
**Depending upon quarkus app build/deploy approach and adjustments to application.properties file and project extensions**  
**some of the following snippets may be useful**  
*Be careful when using an approach on Mac that builds native  and/or the container image locally*  
*It probably won't run on linux based OCP, unless made compatible*  

```
# s2i many options
# https://quarkus.io/guides/deploying-to-kubernetes#openshift  
mvn clean package -Dquarkus.container-image.build=true  
# creates image on and imagestream in ocp project , local yml files are in target dir for deployment or use oc commands

# builds and deploys via s2i
mvn clean compile package -Dquarkus.openshift.deploy=true

# OCP Binary build - upload app, ocp merges it to builder image (also other approaches here jib, docker, buildpacks...)
# https://quarkus.io/guides/container-image#openshift 
mvn clean package -Dquarkus.container-image.build=true -Dquarkus.container-image.push=true  
# if creds aren't set for mvn push, but podman login established to quay.io, or make repo public 
podman push quay.io/dbrugger946/quarkus-coolstore:1.0  

# native build approaches, some assume "native" profile set in pom.xml -Pnative else -Dnative
mvn clean package -Pnative -Dquarkus.native.container-build=true -Dquarkus.container-image.build=true -Dquarkus.kubernetes.deploy=true
podman push quay.io/dbrugger946/quarkus-coolstore:1.0  

# if you have a native build profile and also want to skip tests
# this just builds a native executable lots of options
./mvnw install -Dnative -DskipTests -Dquarkus.native.container-build=true
# see https://quarkus.io/guides/building-native-image
# include creating container image
mvn package -Dnative -Dquarkus.native.container-build=true -Dquarkus.container-image.build=true

```

scratchpad some of these examples are linux specfic Won't work on Mac  
```
mvn clean compile package
podman build -f src/main/docker/Dockerfile.jvm -t quarkus/code-with-quarkus-jvm .
podman tag localhost/quarkus/code-with-quarkus-jvm:latest quay.io/dbrugger946/quarkus-coolstore:1.0
podman push quay.io/dbrugger946/quarkus-coolstore:1.0
 ```  

### Tekton Approaches

**Using Buildpacks**  
see: https://developers.redhat.com/articles/2025/07/16/build-container-images-cicd-tekton-and-buildpacks  
