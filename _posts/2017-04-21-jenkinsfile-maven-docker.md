---
layout: post
title:  "Jenkins, a Jenkinsfile, Maven and Docker"
date:   2017-04-21 16:39:00 +0000
categories: jenkins jenkinsfile maven docker
---

I spent far too long reading sparse documentation so here's a fuller example. What we have is a Jenkins instance,
a software project based on Maven, and the need to push it as a Docker image to a private registry.

Here's the Dockerfile include in the project root:

```dockerfile
FROM openjdk:8

MAINTAINER James Green

ADD target/my-app.jar /application.jar
ENV JAVA_OPTS=""
ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /application.jar" ]
```

