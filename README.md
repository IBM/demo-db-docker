# demo-db-docker

This repo was created to use in the Tutorial [here](name/link TBD).

The code here is a copy of the janusgraph-docker repo. We chose to make a copy so that the JanusGraph community wouldn't be bothered by anything specific to our tutorial and not part of their very real project. 

## We added...

* /chart
* /0.4/Dockerfile.rhos
* /0.5/docker_entry_rhos.sh

# Review the Dockerfile modifications for OpenShift

We already provided a [Dockerfile-rhos](https://github.com/markstur/your-db-docker/blob/master/0.5/Dockerfile-rhos) and [docker-entrypoint-rhos.sh](https://github.com/markstur/your-db-docker/blob/master/0.5/docker-entrypoint-rhos.sh) which are modified versions of the original Dockerfile and docker-entrypoint.sh from the janusgraph-docker repo. You should be able to use our repo as-is, but since this is an important part of creating an image that can pass certification, let's review those changes.

When modifying the Docker file, we have 2 goals in mind:

* Use best practices to create a Universal Application Image (UAI), as described in this [article](https://github.ibm.com/TT-ISV-org/images/tree/main/article/image-qualities).

* Build an image that is in compliance with Red Hat Container certification requirements.

As described in the best practices article, the first thing we need to do is make sure our image is based on a [Universal Base Image (UBI)](add-link-here). UBIs run in both OpenShift and Kubernetes, and can be distributed royalty free.

To accomplish this, we must first determine which UBI to build from by searching the available options from the [Red Hat Container Catalog](https://catalog.redhat.com/software/containers/explore/).

Since we are building a Java app, we will look for an OpenJDK UBI. Doing a search on `openjdk`, we find OpenJDK 1.8 and OpenJDK 11 UBI images.

![ubi8openjdk.png](images/ubi8openjdk.png)

Click on the tile to get release information.

![openjdk11_details.png](images/openjdk11_details.png)

To use the current latest version, we would use the tag `1.3-15`. In our Dockerfile-rhos, we use `FROM registry.access.redhat.com/ubi8/openjdk-11:1.3-15` (where it was using 8-jre-slim-buster).

| Before | After |
| --- | --- |
| `FROM openjdk:8-jre-slim-buster` | `FROM registry.access.redhat.com/ubi8/openjdk-11:1.3-15` |

After this `FROM` line, we needed to add `USER root`. Without this line some of the chmod commands that follow would fail. We will not be running the container as root though. After the permissions are set, a `USER janusgraph` command is used.

When the container runs on OpenShift, it will be assigned a different user ID based on the project. This user will be a member of the root group. Some modifications were made to give access to this user with the root group instead of the janusgraph user and group that was assumed to be available before. Running the database doesn't really require a special user. It was only the file permission setup that needed some adjustments.

We used the name Dockerfile-rhos and docker-entrypoint-rhos.sh so the that original versions are still available. You can see all the modifications by comparing the files.

# Adding charts with oc enable

% oc enable
Looking up pipelines from repository: https://cloud-native-toolkit.github.io/garage-pipelines/
? Which pipeline should be enabled? java-maven
piping stream
**Here's what happened**

1. A listing of available pipelines was retrieved from https://cloud-native-toolkit.github.io/garage-pipelines/
2. You selected the stable/java-maven@latest pipeline
3. We added the following files to your repo:
  - Jenkinsfile
  - chart
  - pipeline.yaml
  - sonar-project.properties

Don't forget to commit the new files
% git status
On branch main
Your branch is up to date with 'origin/main'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	Jenkinsfile
	chart/
	pipeline.yaml
	sonar-project.properties

nothing added to commit but untracked files present (use "git add" to track)
% git add chart
% git status
On branch main
Your branch is up to date with 'origin/main'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	new file:   chart/base/.helmignore
	new file:   chart/base/Chart.yaml
	new file:   chart/base/templates/NOTES.txt
	new file:   chart/base/templates/_helpers.tpl
	new file:   chart/base/templates/deployment.yaml
	new file:   chart/base/templates/ingress.yaml
	new file:   chart/base/templates/route.yaml
	new file:   chart/base/templates/service.yaml
	new file:   chart/base/values.yaml



