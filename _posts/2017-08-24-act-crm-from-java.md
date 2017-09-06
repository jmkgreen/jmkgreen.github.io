---
layout: default
title:  "Connect to Act! CRM from Java"
date:   2017-08-24 10:00:00 +0000
categories: act-crm java
---

Like many businesses my employer holds data across a number of software packages. One of these is _Act! CRM_ and in recent times an official web service interface has been added for us programmers to finally gain access to the structured data within it.

The interface Act supply is HTTP based and uses Swagger so the barrier to entry has fallen in-line with many other projects software developers now face. This article points you at a GitHub repository where an automatically-generated Java client exists and provides a small fragment of code to help you authenticate before making use of this client.

This article does not:

1. Cover the installation of the Act HTTP server
2. Explain the technologies in use - you should make yourself familiar with them as needed

Technology Notes:

1. We're pretty used to using OpenFeign as our HTTP client of choice so that's what we're using here
2. The Act client was auto-generated using Swagger-CodeGen at https://github.com/swagger-api/swagger-codegen - see below for how
3. I'm going to assume you can build a skeleton software project with Maven

OK let's get stuck in.

##Part One - Authentication

This is needed if you have a username and password pair and need a bearer token. If your software starts off with a bearer token clearly you can skip this step.

Add OpenFeign and create an interface called `ActAuthentication`. Fill it with this:

```java
import feign.Headers;
import feign.Param;
import feign.RequestLine;
import feign.Response;

public interface ActAuthentication {

    /**
     * Switch on HTTP Basic authentication to use this call
     *
     * @param databaseName
     * @return Bearer token value on success
     */
    @RequestLine("GET /authorize")
    @Headers({"Act-Database-Name: {databaseName}"})
    Response authenticate(@Param("databaseName") String databaseName);
}

```

Next create a class that will include the authentication routine. Here's a fragment:

```java
import feign.Feign;
import feign.Response;
import feign.auth.BasicAuthRequestInterceptor;
import feign.jackson.JacksonDecoder;
import feign.jackson.JacksonEncoder;

public class ActClient {
    public String authenticate(String username, String password, String actDatabaseName) {
        ActAuthentication actAuthentication = Feign.builder()
                .decoder(new JacksonDecoder())
                .encoder(new JacksonEncoder())
                .requestInterceptor(new BasicAuthRequestInterceptor(username, password))
                .target(ActAuthentication.class, "https://act.my-company.com/Act.Web.API");
        Response response = actAuthentication.authenticate(actDatabaseName);
        String bearer = new BufferedReader(new InputStreamReader(response.body().asInputStream()))
                .lines().collect(Collectors.joining("\n"));

        logger.info("I have a bearer token of {}", bearer);
        return bearer;
    }
}

```

Now you are free to hand the bearer token (which will need to be replaced over time) into the Act Java client.

##Part Two - Act Client

The Java client can be found in a GitHub repo: https://github.com/jmkgreen/act-java

Build the client locally - I have not yet published it to Maven central. Next, depend on the resulting artifact.

Now let's take the bearer token and use it:

```java
public class ActClient {
    public void useClient(String bearer) {
        ApiClient apiClient = new ApiClient();
        ApiKeyAuth apiKeyAuth = new ApiKeyAuth("header", "Authorization");
        apiKeyAuth.setApiKey("Bearer " + bearer);
        apiClient.addAuthorization("http", apiKeyAuth);
        apiClient.setBasePath("http://act.my-company.com/Act.Web.API");
        
        // Now use in anger
        ProductsApi productsApi = apiClient.buildClient(ProductsApi.class);
        System.out.println("Product 0 has name: " + productsApi.productsGet().get(0).getName());

    }
}

```

##Part Three - (Re)Building the Act Client - Optional

This is how https://github.com/jmkgreen/act-java came to be.

Start by creating a folder, say `act-java`. Within this create a `config.json` with the following content:

```json
{
  "groupId":"com.act",
  "artifactId":"act-client",
  "artifactVersion":"1.0.0",
  "library":"feign",
  "apiPackage":"com.act.client.swagger.api",
  "modelPackage":"com.act.client.swagger.model"
}
```

Next, generate the client. I did this using docker:

```bash
docker run --rm -v ${PWD}/act-java:/local swaggerapi/swagger-codegen-cli generate -i https://actforweb.actops.com/Act.Web.API/swagger/docs/v1 -l java -c /local/config.json -o /local/java
```

Now check your `act-java/java` folder and see a new software project exists. This was imported in to my repository.

Finally I had to make the following changes:

1. Locate the `SystemObject` class and add a `String value` field with a corresponding constructor

