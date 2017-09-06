---
layout: default
title:  "Connect to KeyCloak from Java for Administration"
date:   2017-09-05 10:00:00 +0000
categories: keycloak java
---

I have recently been playing around internally with a Keycloak installation. This post is limited to how to get your Java code to interact with Keycloak for administrative operations - *not* for authentication purposes.

It turns out that Keycloak produce a `keycloak-admin-client` Maven dependency. Trouble is integrating this successfully with your own application since many of the existing examples are not up to date. This post is for Spring Boot 1.x applications working with Keycloak 3.2.


###Maven Dependencies

You will need:

```xml
	<properties>
		<keycloak.version>3.2.1.Final</keycloak.version>
		<resteasy.version>3.0.14.Final</resteasy.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.keycloak</groupId>
			<artifactId>keycloak-admin-client</artifactId>
			<version>${keycloak.version}</version>
		</dependency>
		<dependency>
			<groupId>org.keycloak</groupId>
			<artifactId>keycloak-core</artifactId>
			<version>${keycloak.version}</version>
		</dependency>

		<dependency>
			<!-- for keycloak -->
			<groupId>org.jboss.resteasy</groupId>
			<artifactId>resteasy-jaxrs</artifactId>
			<version>${resteasy.version}</version>
		</dependency>
		<dependency>
			<!-- for keycloak -->
			<groupId>org.jboss.resteasy</groupId>
			<artifactId>resteasy-client</artifactId>
			<version>${resteasy.version}</version>
		</dependency>
		<dependency>
			<groupId>org.jboss.resteasy</groupId>
			<artifactId>resteasy-jackson2-provider</artifactId>
			<version>${resteasy.version}</version>
		</dependency>
    </dependencies>
```

### Keycloak Configuration

```java
package com.foo;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

@Configuration
@ConfigurationProperties(prefix = "keycloak")
public class KeycloakConfiguration {

    private String serverUrl;
    private String realm;
    private String username;
    private String password;
    private String clientId;
    private String clientSecret;

    public String getServerUrl() {
        return serverUrl;
    }

    public void setServerUrl(String serverUrl) {
        this.serverUrl = serverUrl;
    }

    public String getRealm() {
        return realm;
    }

    public void setRealm(String realm) {
        this.realm = realm;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getClientId() {
        return clientId;
    }

    public void setClientId(String clientId) {
        this.clientId = clientId;
    }

    public String getClientSecret() {
        return clientSecret;
    }

    public void setClientSecret(String clientSecret) {
        this.clientSecret = clientSecret;
    }
}

```

### Client Use

Start with some set-up:

```java
        Keycloak keycloak = KeycloakBuilder.builder()
                .serverUrl(keycloakConfiguration.getServerUrl())
                .realm(keycloakConfiguration.getRealm())
                .username(keycloakConfiguration.getUsername())
                .password(keycloakConfiguration.getPassword())
                .clientId(keycloakConfiguration.getClientId())
                .clientSecret(keycloakConfiguration.getClientSecret())
                .resteasyClient(new ResteasyClientBuilder().connectionPoolSize(10).build())
                .build();

```

Now the above does *not* contact the Keycloak server by itself so if the above is configured incorrectly an error will *not* yet be exposed - we need to try an operation for that.

Let's try an operation:

```java
        RealmResource realmResource = keycloak.realm(keycloakConfiguration.getRealm());
        log.debug("I can find {} users inside the realm!", realmResource.users().count());
        UserRepresentation userRepresentation = new UserRepresentation();
        userRepresentation.setUsername(email);
        userRepresentation.setEmail(email);
        Response response = realmResource.users().create(userRepresentation);
        log.info("Creation of email={} resulted in keycloakResponseCode=={} keycloakResponseText={}", email, response.getStatus(), response.getStatusInfo().getReasonPhrase());

```

OK so this grabs the number of users inside the realm (which is a basic operation to prove things work), then does something slightly more substantial in the creation of a really basic user. Note I found an internal server error coming from Keycloak if the user did not have a username set, this may be fixed in later versions to return 400 Bad Request.

###What About Swagger?

Yeah that would be nice. See [KEYCLOAK-4474](https://issues.jboss.org/browse/KEYCLOAK-4474) for this and related issues.
