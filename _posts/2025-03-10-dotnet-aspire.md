---
layout: post
title: Distributed App Development
subtitle: Aspiring for a better Distributed App Development Experience
author: jjk_charles
categories: programming general
tags: dotnet developer-experience distributed-applications
---

With proliferation of Public Cloud usage across the industry, there is a high change you've already been exposed to Distributed Application Development and the challenges it brings especially during local development and testing. While tools like Docker Compose can help ease the pain, they don't address all the problems involved.

This becomes even more complicated when dealing with disparate technologies. For instance, within my team at work, we handle React, .NET, Java, and Python for various UI/API workloads, along with a mix of MS SQL, Oracle, and Postgres for databases.

My team has been facing challenges with performing a good integration testing in their local system before pushing their code changes into the main branch, which in turn results in bugs leaking into our integration environment causing wasted efforts from Developers and QAs.

## .Net Aspire

One area where .NET Aspire could greatly help is in orchestration. Aspire assists with various tasks, such as orchestration of different components, [Integrations](https://learn.microsoft.com/en-us/dotnet/aspire/get-started/aspire-overview#net-aspire-integrations), [Deployments](https://learn.microsoft.com/en-us/dotnet/aspire/deployment/overview) etc. 

Since I was dealing with multiple technologies, my primary focus for leveraging Aspire was around the below areas,

1. Orchestration
2. Consistent starting point for integration
3. Local logging and Observability
4. Integration Testing

### Orchestration

Here Aspire allows us to programmatically specify how to spin up the different components involved, configure them and wire them up. Don't let the ".net" in its name fool you, its not just meant for orchestrating .Net workloads, but it can work across .Net projects, Containers, and Executables. And with its Extensibility Support, possibilities are endless to extend it to other areas too. For instance, I will cover later about how I am leveraging support for Java's Spring Boot applications.

### Consistent Starting point for integration

While this is not something specific to .Net Aspire, this is one of the major pain points for our team at work, where our QAs had to comb through our Test environment for hours to gather all the test data needed for them to run their test cases. 

The goal is not just to bring up all the necessary components, but also to ensure that every time this happens, all components have a standardized set of data as a starting point. This way, developers don’t have to worry about what data to use for testing during feature development, and integration tests can rely on a consistent set of data that is referenced across all components.

### Local logging and Observability

During the development phase, all developers leverage logging as a tool to help troubleshoot problems, but having Observability in other integration and Production environments is what makes everybody's life so much more easier. 

### Integration Testing

The final goal is to run the automated regression test suite on the developer's local machine without needing to go through tedious data collection (whether automated or manual) every time the tests are run.

## Setup

Before I started trying out our actual production workloads via Aspire, I puttogether a POC using .Net, Java and Postgresql to float this idea within the team (to my surprise, no one at this point in time within my team - all .Net developers, had come across .Net Aspire). This POC setup is what I will be talking about here.

I wanted to get things setup very quickly, so I ended up using the sample Project template .Net Aspire comes with, and plugged in a Java Spring Boot application and a Postgresql Database along with them.

My POC setup had the below components in it,

1. .Net Blazor Web App for UI
2. .Net ASP .Net Core Web API Project for API
3. Java Spring Boot Application for API
4. Postgresql Database that Java API uses

The relationship/dependency between them is as below.

![.Net Aspire POC Setup](/assets/images/posts/dotnet-aspire-001.png)

## Pre-requisites and my challenges

I am not going to be covering the tooling or installation/setup as part of this article, as [Microsoft's documentation](https://learn.microsoft.com/en-us/dotnet/aspire/get-started/aspire-overview) covers it at good lengths. But rather, I will be talking about what challenges I ran into while getting my POC running in our Enterprise setup (On the personal front, it was a breeze)

First of all, one of the key things that makes things easier with Aspire is the ability to run containers as part of the orchestration. This is also how some of the out-of-the-box components like PostgreSQL and Elasticsearch are run by Aspire. 

This is where I started running into issues with my org's dev environment setup - I can’t use Docker Desktop due to licensing restrictions—Challenge #1.

As a logical next step, since Aspire supports Podman, I was eager to try it out. Only to find out that the Dev environment was running an older version of windows which cannot install/support Podman - Challenge #2.

I was trying out ways to get the Windows version updated, but also parallely was researching in my person PC to see if Podman is going to solve my problems. Glad I took this step before moving too much forward with seeking approvals and exceptions to get the windows version upgraded, as I ran into issues while using Podman, which I have not run into while using Docker Desktop.

All problems I have run into seemed to have been around networking, where the containers running inside WSL was not able to communicate with Aspire Proxt, and vice versa.

I assumed this is a problem with Podman itself, and then went on to try running Docker runtime in WSL, and try setting up Docker CLI in windows that can route all the commands to Docker running in WSL. Sure enough, I ran into the same problems as that of when I was running the containers through Podman.

So, this looks like some networking issues in being able to bridge the system's local network with that of the virtual network that WSL uses. Docker Desktop seems to be doing some networking magic, that somehow makes things work seamlessly.

At this point, I pretty much gave up on Aspire+Containers combo, and decided to get things working without any Containers involved.

> Not that anyone reads my blog, but if someone from Microsoft happens to be stumbling into this post - **"Considering the licensing requirements around Docker Desktop, please prioritize addressing the networking issues involved in Aspire+Podman/WSL, and support Podman as a first class citizen to ensure rapid enterprise adoption of Aspire."** In my case, chances of mass procurement of Docker Desktop licenses for the org. and making everyone adopt might not be a feasibility, at least in the near future.

## POC

When it comes to the POC itself, again, I am not planning on covering the .Net pieces as Aspire documentation does a good (enough?) job of covering it.

Rather, I will be focusing on how I got the Java and Postgres instances setup, as well as how I made them all talk to one another.

To refresh, my POC will be focusing on the below connectivity,
1. Java API -> PostgreSQL
2. Java API -> .Net API
3. Blazor UI -> Java API

### Hosting the Java API

My initial plan for running the Java application was to run it using Aspire's Container support. But, now that I would have to do away without Containers, I started looking for alternate option. 

I was already aware that I could run any executable via Aspire, using 

    builder.AddExecutable()

method. But then, I realized that there is a Community built Java support via [CommunityToolkit.Aspire.Hosting.Java](https://www.nuget.org/packages/CommunityToolkit.Aspire.Hosting.Java/9.2.1).

While I could have achieved similar outcome with `AddExecutable()`, `AddSpringApp()` method from the above Nuget package makes things easier. Here's how I approached it:

```csharp
var javaApi = builder.AddSpringApp("javaapi", "../Other Dependencies/sampleapi",
        new JavaAppExecutableResourceOptions() {
            OtelAgentPath = "lib/",
            ApplicationName = "target/sampleapi-0.0.1-SNAPSHOT.jar",
            Port=8090
        })
    .WithMavenBuild()
    .WithEnvironment("DB_URL", $"jdbc:postgresql://{postgresHostname}/{postgresDBName}")
    .WithEnvironment("DB_USERNAME", $"{postgresUsername}")
    .WithEnvironment("DB_PASSWORD", $"{postgresPassword}")
    .WithOtlpExporter()
    .WithHttpEndpoint(8090, name: "javaapi-http", isProxied:false)
    .WithReference(apiService)
    .WithReference(postgresDB)
    .WaitFor(postgresDB);
```

Did you notice, it even allows triggering a Maven Build before launching the instance, which is pretty neat.

I am injecting few Environment variables into the Service, so that I can later refer to it within the Java Application.

```csharp
    .WithEnvironment("DB_URL", $"jdbc:postgresql://{postgresHostname}/{postgresDBName}")
    .WithEnvironment("DB_USERNAME", $"{postgresUsername}")
    .WithEnvironment("DB_PASSWORD", $"{postgresPassword}")
```

Here, `WithReference(postgresDB)` isn't really helpful, as it primarily injects information allowing that service to be configured via connection strings.  Since the service I am running is a Java workload, I don't think they help in any way, but I still wanted to retain it as it explicitly calls out the dependency for anyone looking into that code.

Since Postgresql connection strings are defaulted to what is needed for .Net, I was looking at options to see how to override formatting of the connection strings. But, since I wasn't able to find a way to do it, I ended up pulling the individual pieces of information and framing the URL myself.

#### Managing Dependencies (Java API -> PostgreSQL and .Net API)

As I had mentioned above, for configuring the Postgres instance, I am injecting the connection string, User name and Password as environment variables, so that I can refer to those within the Java Config files.

For my purposes that environment variables used as DB_URL, DB_USERNAME and DB_PASSWORD.

I referenced them within the Java `application.properties` file like this:

```properties
spring.application.name=sampleapi

spring.datasource.url=${DB_URL:}
spring.datasource.username=${DB_USERNAME:}
spring.datasource.password=${DB_PASSWORD:}

api.weather.url=${services__apiservice__http__0}
```

As for the URL of .Net API that I wanted to refer to, Aspire injects it as part of the call to `WithReference(apiService)` as an environment variable. I am merely refering to the same, by following the naming convention it uses.

` services__SERVICENAME__ENDPOINTNAME__EDNDPOINTINDEX

Note: Default endpoint name assigned is "http"/"https".

### Hosting the Database

For the database, I decided to use containers, so I set it up like this:

```csharp
var postgresDB = builder.AddPostgres("localPg", pgUsername, port:60123)
    .WithBindMount("../Data/postgres/seed-data", "/docker-entrypoint-initdb.d")
    .AddDatabase("postgres");
```

This creates a new Postgresql container with "postgres" as the database, User name passed under pgUsername, Port as 60123, and a random password.

Do also note the use of `WithBindMount()` which allows us to execute arbitary SQL commands at the time of initializing the container. I am using this to ensure that the necessary tables are created as well as they are loaded with some initial data.

### Managing Dependencies (.Net -> Java API)

This direction of dependency is pretty easy to Handle with Aspire's Service Discovery capabilities.

All that is needed is for us to tell Aspire that the .Net Project depends on the Java Project, like below,

```csharp
builder.AddProject<Projects.MyTestAspireApp_Web>("webfrontend")
    .WithExternalHttpEndpoints()
    .WithReference(apiService)
    .WithReference(javaApi);
```

and then refer to the Java API by its service name, and let Service Discovery do the trick for us.

```csharp
builder.Services.AddHttpClient<JavaAPIClient>(client =>
{
    client.BaseAddress = new("https+http://javaapi");
});
```

### Observability

Observability is one of the key things that I love about Aspire - it makes it so much easier to achieve within the local dev environment.

For any .Net projects that leverages [ServiceDefaults](https://learn.microsoft.com/en-us/dotnet/aspire/get-started/build-your-first-aspire-app?pivots=dotnet-cli#net-aspire-service-defaults-project), Open Telemetry is autoconfigured by default.

But, for my Java workload I had to take some extra steps (nothing too complex though). To begin with, the [java otel instrumentation agent](https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/) had to be downloaded and placed within AAspire solution folder, which then needs to be passed into `AddSpringApp()`. After that the `WithOtelExporter()` needs to be called to ensure that all environment variables related to OTEL (with URL, Password etc.) is injected into the service.

## End Result

You can refer to the code mentioned above in this [GitHub repository](https://github.com/jjkcharles/CrossLanguageAspireApp).

I am pretty pleased with what we could achieve with Aspire, and how much time it could help save for developers and QAs.

If you haven’t already tried Aspire, do give it a spin right away!