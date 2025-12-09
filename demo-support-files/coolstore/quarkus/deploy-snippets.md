
## Links and snippets for creating and deploying the quarkus app
*for showing different ways to build/deploy during demo*

### Basic steps
- log into OCP and get in the quarkus branch
- create the database
- build/deploy the quarkus app using any of the following suggested approaches, or via a pipeline

https://quarkus.io/guides/deploying-to-kubernetes#defining-a-docker-registry  
https://quarkus.io/guides/deploying-to-kubernetes#openshift  
https://quarkus.io/guides/deploying-to-openshift  
**buildpacks**  
https://developers.redhat.com/articles/2025/08/20/build-container-image-quarkus-project-using-buildpacks  


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
** Depending upon quarkus app build/deploy approach and adjustments to application.properties file and project extensions **  
** some of the following snippets may be useful **    

```
mvn clean compile package -Dquarkus.kubernetes.deploy=true

mvn clean package -Dquarkus.container-image.build=true -Dquarkus.container-image.push=true  
*if creds aren't set for mvn push, but podman login established to quay.io, or make repo public*  
podman push quay.io/dbrugger946/quarkus-coolstore:1.0  

mvn clean package -Pnative -Dquarkus.native.container-build=true -Dquarkus.container-image.build=true -Dquarkus.kubernetes.deploy=true
podman push quay.io/dbrugger946/quarkus-coolstore:1.0  


```