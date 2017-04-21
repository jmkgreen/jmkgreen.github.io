---
layout: post
title:  "Jenkins, a Jenkinsfile, Maven and Docker"
date:   2017-04-21 16:39:00 +0000
categories: jenkins jenkinsfile maven docker
---

I spent far too long reading sparse documentation so here's a fuller example. What we have is a Jenkins instance,
a software project based on Maven, and the need to push it as a Docker image to a private registry.

Before you go further you will need to set up Jenkins with two things:

1. Ensure you have the `Confile File Provider Plugin` installed, then supply a `settings.xml` file through it that Maven will use and holds the server credentials that your pom refers to for snapshots and releases distribution into the likes of Nexus.
2. A credentials item specifying your private docker registry (in our case `https://docker-registry.my-corp.com:5000`) with username and password

Now, to the code.

First you need to adjust your `pom.xml` to emit a jar file with predictable naming:

```
...
<build>
  <finalName>my-app</finalName>
</build>
...
```

This results in `target/my-app.jar`.

Here's the Dockerfile included in the project root:

```dockerfile
FROM openjdk:8

MAINTAINER James Green

ADD target/my-app.jar /application.jar
ENV JAVA_OPTS=""
ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /application.jar" ]
```

Here's the `Jenkinsfile` - substitute the private registry:

```Jenkinsfile
#!groovy
pipeline {
    agent none

    tools {
        maven 'Maven 3.3.9'
        jdk 'jdk8'
    }

    stages {
        stage('Build') {
            agent {
                label 'docker'
            }
            steps {
                withMaven(jdk: 'jdk8', maven: 'Maven 3.3.9', mavenLocalRepo: '/var/jenkins_home/.m2/repository', mavenSettingsConfig: 'guid-found-in-from-jenkins-config-file-plugin-ui') {
                    sh 'mvn clean deploy'
                }
                script {
                    docker.withRegistry('https://docker-registry.my-corp.com:5000', 'guid-found-in-jenkins-credentials-list') {
                        def app = docker.build("mycorp/${JOB_NAME}:${BUILD_NUMBER}")
                        app.push()
                    }
                }
            }
        }
    }
}
```
