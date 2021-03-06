---
title: ASP.NET Core fundamentals
author: rick-anderson
description: Learn the foundational concepts for building ASP.NET Core apps.
ms.author: riande
ms.custom: mvc
ms.date: 02/14/2019
uid: fundamentals/index
---
# ASP.NET Core fundamentals

This article is an overview of key topics for understanding how to develop ASP.NET Core apps.

## The Startup class

The `Startup` class is where:

* Any services required by the app are configured.
* The request handling pipeline is defined.

* Code to configure (or *register*) services is added to the `Startup.ConfigureServices` method. *Services* are components that are used by the app. For example, an Entity Framework Core context object is a service.
* Code to configure the request handling pipeline is added to the `Startup.Configure` method. The pipeline is composed as a series of *middleware* components. For example, a middleware might handle requests for static files or redirect HTTP requests to HTTPS. Each middleware performs asynchronous operations on an `HttpContext` and then either invokes the next middleware in the pipeline or terminates the request.

::: moniker range=">= aspnetcore-2.0"

Here's a sample `Startup` class:

[!code-csharp[](index/snapshots/2.x/Startup1.cs?highlight=3,12)]

::: moniker-end

For more information, see [App startup](xref:fundamentals/startup).

## Dependency injection (services)

ASP.NET Core has a built-in dependency injection (DI) framework that makes configured services available to an app's classes. One way to get an instance of a service in a class is to create a constructor with a parameter of the required type. The parameter can be the service type or an interface. The DI system provides the service at runtime.

::: moniker range=">= aspnetcore-2.0"

Here's a class that uses DI to get an Entity Framework Core context object. The highlighted line is an example of constructor injection:

[!code-csharp[](index/snapshots/2.x/Index.cshtml.cs?highlight=5)]

::: moniker-end

While DI is built in, it's designed to let you plug in a third-party Inversion of Control (IoC) container if you prefer.

For more information, see [Dependency injection](xref:fundamentals/dependency-injection).

## Middleware

The request handling pipeline is composed as a series of middleware components. Each component performs asynchronous operations on an `HttpContext` and then either invokes the next middleware in the pipeline or terminates the request.

By convention, a middleware component is added to the pipeline by invoking its `Use...` extension method in the `Startup.Configure` method. For example, to enable rendering of static files, call `UseStaticFiles`.

::: moniker range=">= aspnetcore-2.0"

The highlighted code in the following example configures the request handling pipeline:

[!code-csharp[](index/snapshots/2.x/Startup1.cs?highlight=14-16)]

::: moniker-end

ASP.NET Core includes a rich set of built-in middleware, and you can write custom middleware.

For more information, see [Middleware](xref:fundamentals/middleware/index).

<a id="host"/>

## The host

An ASP.NET Core app builds a *host* on startup. The host is an object that encapsulates all of the the app's resources, such as:

* An HTTP server implementation
* Middleware components
* Logging
* DI
* Configuration

The main reason for including all of the app's interdependent resources in one object is lifetime management: control over app startup and graceful shutdown.

The code to create a host is in `Program.Main` and follows the [builder pattern](https://wikipedia.org/wiki/Builder_pattern). Methods are called to configure each resource that is part of the host. A builder method is called to pull it all together and instantiate the host object.

::: moniker range="<= aspnetcore-2.2"

ASP.NET Core 2.x uses Web Host (the `WebHost` class) for web apps. The framework provides `CreateDefaultBuilder` extension methods that set up a host with commonly used options, such as the following:

* Use [Kestrel](#servers) as the web server and enable IIS integration.
* Load configuration from *appsettings.json*, environment variables, command line arguments, and other sources.
* Send logging output to the console and debug providers.

::: moniker-end

::: moniker range=">= aspnetcore-2.0 <= aspnetcore-2.2"

Here's sample code that builds a host:

[!code-csharp[](index/snapshots/2.x/Program1.cs?highlight=9)]

For more information, see [Web Host](xref:fundamentals/host/web-host).

::: moniker-end

::: moniker range="> aspnetcore-2.2"

In ASP.NET Core 3.0, Web Host (`WebHost` class) or Generic Host (`Host` class) can be used in a web app. Generic Host is recommended, and Web Host is available for backwards compatibility.

The framework provides `CreateDefaultBuilder` and `ConfigureWebHostDefaults` extension methods that set up a host with commonly used options, such as the following:

* Use [Kestrel](#servers) as the web server and enable IIS integration.
* Load configuration from *appsettings.json*, *appsettings.[EnvironmentName].json*, environment variables, and command line arguments.
* Send logging output to the console and debug providers.

Here's sample code that builds a host. The extension methods that set up the host with commonly used options are highlighted.

[!code-csharp[](index/snapshots/3.x/Program1.cs?highlight=9-10)]

For more information, see [Generic Host](xref:fundamentals/host/generic-host) and [Web Host](xref:fundamentals/host/web-host)

::: moniker-end

### Advanced host scenarios

::: moniker range=">= aspnetcore-2.1 <= aspnetcore-2.2"

Web Host is designed to include an HTTP server implementation, which isn't needed for other kinds of .NET apps. Starting in 2.1, Generic Host (`Host` class) is available for any .NET Core app to use&mdash;not just ASP.NET Core apps. Generic Host lets you use cross-cutting features such as logging, DI, configuration, and app lifetime management in other types of apps. For more information, see [Generic Host](xref:fundamentals/host/generic-host).

::: moniker-end

::: moniker range="> aspnetcore-2.2"

Generic Host is available for any .NET Core app to use&mdash;not just ASP.NET Core apps. Generic Host lets you use cross-cutting features such as logging, DI, configuration, and app lifetime management in other types of apps. For more information, see [Generic Host](xref:fundamentals/host/generic-host).

::: moniker-end

You can also use the host to run background tasks. For more information, see [Background tasks](xref:fundamentals/host/hosted-services).

## Servers

An ASP.NET Core app uses an HTTP server implementation to listen for HTTP requests. The server surfaces requests to the app as a set of [request features](xref:fundamentals/request-features) composed into an `HttpContext`.

::: moniker range=">= aspnetcore-2.2"

# [Windows](#tab/windows)

ASP.NET Core provides the following server implementations:

* *Kestrel* is a cross-platform web server. Kestrel is often run in a reverse proxy configuration using [IIS](https://www.iis.net/). In ASP.NET Core 2.0 or later, Kestrel can be run as a public-facing edge server exposed directly to the Internet.
* *IIS HTTP Server* is a server for windows that uses IIS. With this server, the ASP.NET Core app and IIS run in the same process.
* *HTTP.sys* is a server for Windows that isn't used with IIS.

# [macOS](#tab/macos)

ASP.NET Core provides the *Kestrel* cross-platform server implementation. In ASP.NET Core 2.0 or later, Kestrel can be run as a public-facing edge server exposed directly to the Internet. Kestrel is often run in a reverse proxy configuration with [Nginx](https://nginx.org) or [Apache](https://httpd.apache.org/).

# [Linux](#tab/linux)

ASP.NET Core provides the *Kestrel* cross-platform server implementation. In ASP.NET Core 2.0 or later, Kestrel can be run as a public-facing edge server exposed directly to the Internet. Kestrel is often run in a reverse proxy configuration with [Nginx](https://nginx.org) or [Apache](https://httpd.apache.org/).

---

::: moniker-end

::: moniker range="< aspnetcore-2.2"

# [Windows](#tab/windows)

ASP.NET Core provides the following server implementations:

* *Kestrel* is a cross-platform web server. Kestrel is often run in a reverse proxy configuration using [IIS](https://www.iis.net/). In ASP.NET Core 2.0 or later, Kestrel can be run as a public-facing edge server exposed directly to the Internet.
* *HTTP.sys* is a server for Windows that isn't used with IIS.

# [macOS](#tab/macos)

ASP.NET Core provides the *Kestrel* cross-platform server implementation. In ASP.NET Core 2.0 or later, Kestrel can be run as a public-facing edge server exposed directly to the Internet. Kestrel is often run in a reverse proxy configuration with [Nginx](https://nginx.org) or [Apache](https://httpd.apache.org/).

# [Linux](#tab/linux)

ASP.NET Core provides the *Kestrel* cross-platform server implementation. In ASP.NET Core 2.0 or later, Kestrel can be run as a public-facing edge server exposed directly to the Internet. Kestrel is often run in a reverse proxy configuration with [Nginx](http://nginx.org) or [Apache](https://httpd.apache.org/).

---

::: moniker-end

For more information, see [Servers](xref:fundamentals/servers/index).

## Configuration

ASP.NET Core provides a configuration framework that gets settings as name-value pairs from an ordered set of configuration providers. There are built-in configuration providers for a variety of sources, such as *.json* files, *.xml* files, environment variables, and command-line arguments. You can also write custom configuration providers.

For example, you could specify that configuration comes from *appsettings.json* and environment variables. Then when the value of *ConnectionString* is requested, the framework looks first in the *appsettings.json* file. If the value is found there but also in an environment variable, the value from the environment variable would take precedence.

For managing confidential configuration data such as passwords, ASP.NET Core provides a [Secret Manager tool](xref:security/app-secrets). For production secrets, we recommend [Azure Key Vault](/aspnet/core/security/key-vault-configuration).

For more information, see [Configuration](xref:fundamentals/configuration/index).

## Options

Where possible, ASP.NET Core follows the *options pattern* for storing and retrieving configuration values. The options pattern uses classes to represent groups of related settings.

For example, the following code sets WebSockets options:

```csharp
var options = new WebSocketOptions  
{  
   KeepAliveInterval = TimeSpan.FromSeconds(120),  
   ReceiveBufferSize = 4096
};  
app.UseWebSockets(options);
```

For more information, see [Options](xref:fundamentals/configuration/options).

## Environments

Execution environments, such as *Development*, *Staging*, and *Production*, are a first-class notion in ASP.NET Core. You can specify the environment an app is running in by setting the `ASPNETCORE_ENVIRONMENT` environment variable. ASP.NET Core reads that environment variable at app startup and stores the value in an `IHostingEnvironment` implementation. The environment object is available anywhere in the app via DI.

::: moniker range=">= aspnetcore-2.0"

The following sample code from the `Startup` class configures the app to provide detailed error information only when it runs in development:

[!code-csharp[](index/snapshots/2.x/Startup2.cs?highlight=3-6)]

::: moniker-end

For more information, see [Environments](xref:fundamentals/environments).

## Logging

ASP.NET Core supports a logging API that works with a variety of built-in and third-party logging providers. Available providers include the following:

* Console
* Debug
* Event Tracing on Windows
* Windows Event Log
* TraceSource
* Azure App Service
* Azure Application Insights

Write logs from anywhere in an app's code by getting an `ILogger` object from DI and calling log methods.

::: moniker range=">= aspnetcore-2.0"

Here's sample code that uses an `ILogger` object, with constructor injection and the logging method calls highlighted.

[!code-csharp[](index/snapshots/2.x/TodoController.cs?highlight=5,13,17)]

::: moniker-end

The `ILogger` interface lets you pass any number of fields to the logging provider. The fields are commonly used to construct a message string, but the provider can also send them as separate fields to a data store. This feature makes it possible for logging providers to implement [semantic logging, also known as structured logging](https://softwareengineering.stackexchange.com/questions/312197/benefits-of-structured-logging-vs-basic-logging).

For more information, see [Logging](xref:fundamentals/logging/index).

## Routing

A *route* is a URL pattern that is mapped to a handler. The handler is typically a Razor page, an action method in an MVC controller, or a middleware. ASP.NET Core routing gives you control over the URLs used by your app.

For more information, see [Routing](xref:fundamentals/routing).

## Error handling

ASP.NET Core has built-in features for handling errors, such as:

* A developer exception page
* Custom error pages
* Static status code pages
* Startup exception handling

For more information, see [Error handling](xref:fundamentals/error-handling).

::: moniker range=">= aspnetcore-2.1"

## Make HTTP requests

An implementation of `IHttpClientFactory` is available for creating `HttpClient` instances. The factory:

* Provides a central location for naming and configuring logical `HttpClient` instances. For example, a *github* client can be registered and configured to access GitHub. A default client can be registered for other purposes.
* Supports registration and chaining of multiple delegating handlers to build an outgoing request middleware pipeline. This pattern is similar to the inbound middleware pipeline in ASP.NET Core. The pattern provides a mechanism to manage cross-cutting concerns around HTTP requests, including caching, error handling, serialization, and logging.
* Integrates with *Polly*, a popular third-party library for transient fault handling.
* Manages the pooling and lifetime of underlying `HttpClientMessageHandler` instances to avoid common DNS problems that occur when manually managing `HttpClient` lifetimes.
* Adds a configurable logging experience (via *ILogger*) for all requests sent through clients created by the factory.

For more information, see [Make HTTP requests](xref:fundamentals/http-requests).

::: moniker-end

## Content root

The content root is the base path to any private content used by the app, such as its Razor files. By default, the content root is the base path for the executable hosting the app. An alternative location can be specified when [building the host](#host).

::: moniker range="<= aspnetcore-2.2"

For more information, see [Content root](xref:fundamentals/host/web-host#content-root).

::: moniker-end

::: moniker range="> aspnetcore-2.2"

For more information, see [Content root](xref:fundamentals/host/generic-host#content-root).

::: moniker-end

## Web root

The web root (also known as *webroot*) is the base path to public, static resources, such as CSS, JavaScript, and image files. The static files middleware will only serve files from the web root directory (and sub-directories) by default. The web root path defaults to *\<content root>/wwwroot*, but a different location can be specified when [building the host](#host).

In Razor (*.cshtml*) files, the tilde-slash `~/` points to the web root. Paths beginning with `~/` are referred to as virtual paths.

For more information, see [Static files](xref:fundamentals/static-files).
