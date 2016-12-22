---
title: Migrating from ASP.NET 5 RC1 to ASP.NET Core 1.0 | Microsoft Docs
author: rick-anderson
description: 
keywords: ASP.NET Core,
ms.author: riande
manager: wpickett
ms.date: 10/14/2016
ms.topic: article
ms.assetid: b09c8cc7-8baa-44bf-97b2-6e160243b3d3
ms.technology: aspnet
ms.prod: aspnet-core
uid: migration/rc1-to-rtm
---
# Migrating from ASP.NET 5 RC1 to ASP.NET Core 1.0

By [Cesar Blum Silveira](https://github.com/cesarbs), [Rachel Appel](http://github.com/rachelappel), and [Rick Anderson](https://twitter.com/RickAndMSFT)

ASP.NET 5 RC1 apps were based on the .NET Execution Environment (DNX) and made use of DNX specific features. ASP.NET Core 1.0 is based on .NET Core, so you must first migrate your application to the new .NET Core project model. See [migrating from DNX to .NET Core CLI](https://docs.microsoft.com/en-us/dotnet/articles/core/migrating-from-dnx) for more information.

See the following resources for a list of some of the most significant changes, announcements and migrations information:

* [ASP.NET Core RC2 significant changes](https://github.com/aspnet/announcements/issues?q=is%3Aopen+is%3Aissue+milestone%3A1.0.0-rc2)

* [ASP.NET Core 1.0 significant changes](https://github.com/aspnet/announcements/issues?q=is%3Aopen+is%3Aissue+milestone%3A1.0.0)

* [Upgrading from Entity Framework RC1 to RTM](https://docs.efproject.net/en/latest/miscellaneous/rc2-rtm-upgrade.html)

* [Migrating from ASP.NET Core RC2 to ASP.NET Core 1.0](rc2-to-rtm.md)

## Update Target Framework Monikers (TFMs)

If your app targeted `dnx451` or  `dnxcore50` in the `frameworks` section of *project.json*, you must make the following changes:

|DNX|.NET Core|
|---|---|
|`dnx451`|`net451`|
|`dnxcore50`|`netcoreapp1.0`|

.NET Core apps must add a dependency to the `Microsoft.NETCore.App` package:

```none
"dependencies": {
     "Microsoft.NETCore.App": {
       "version": "1.0.0",
       "type": "platform"
     },
   ```

## Namespace and package ID changes

* ASP.NET 5 has been renamed to ASP.NET Core 1.0

* ASP.NET MVC and Identity are now part of ASP.NET Core

* ASP.NET MVC 6 is now ASP.NET Core MVC

* ASP.NET Identity 3 is now ASP.NET Core Identity

* ASP.NET Core 1.0 package versions are `1.0.0`

* ASP.NET Core 1.0 tool package versions are `1.0.0-preview2-final`

Namespace and package name changes:

|ASP.NET 5 RC1|ASP.NET Core 1.0|
|---|---|
|`Microsoft.AspNet.*`|`Microsoft.AspNetCore.*`|
|`EntityFramework.*`|`Microsoft.EntityFrameworkCore.*`|
|`Microsoft.Data.Entity.*`|`Microsoft.EntityFrameworkCore.*`|

The `EntityFramework.Commands` package is no longer available. The `ef` command is now available as a tool in the `Microsoft.EntityFrameworkCore.Tools` package.

The following packages have been renamed:

|ASP.NET 5 RC1|ASP.NET Core 1.0|
|---|---|
|`Microsoft.AspNet.*`|`Microsoft.AspNetCore.*`|
|`EntityFramework.*`|`Microsoft.EntityFrameworkCore.*`|
|`Microsoft.Data.Entity.*`|`Microsoft.EntityFrameworkCore.*`|
|`Microsoft.Data.Entity.*`|`Microsoft.EntityFrameworkCore.*`|

## Commands and tools

The `commands` section of  the *project.json* file is no longer supported. Use `dotnet run` or `dotnet <DLL name>` instead.

.NET Core CLI has introduced the concept of tools. *project.json* now supports a `tools` section where packages containing tools can be specified. Some important functionality for ASP.NET Core 1.0 applications has been moved to tools.

See [.NET Core CLI extensibility model](https://dotnet.github.io/docs/core-concepts/core-sdk/cli/extensibility.html) for more information on .NET Core CLI tools.

### Publishing to IIS

IIS publishing is now provided by the `publish-iis` tool in the `Microsoft.AspNetCore.Server.IISIntegration.Tools` package. If you intend to run your app behind IIS, add the `publish-iis` tool to your *project.json*:

```json
{
  "tools": {
    "Microsoft.AspNetCore.Server.IISIntegration.Tools": "1.0.0-preview2-final"
  }
}
```

The `publish-iis` tool is commonly used in the `postpublish` script in *project.json*:

```json
{
  "postpublish": [ "dotnet publish-iis --publish-folder %publish:OutputPath% --framework %publish:FullTargetFramework%" ]
}
```

### Entity Framework commands

The `ef` tool is now provided in the `Microsoft.EntityFrameworkCore.Tools` package:

```json
{
  "tools": {
    "Microsoft.EntityFrameworkCore.Tools": "1.0.0-preview2-final"
  }
}
```

For more information, see [.NET Core CLI](https://docs.efproject.net/en/latest/cli/dotnet.html).

### Razor tools

Razor tooling is now provided in the `Microsoft.AspNetCore.Razor.Tools` package:

```json
{
  "tools": {
    "Microsoft.AspNetCore.Razor.Tools": "1.0.0-preview2-final"
  }
}
```

### SQL cache tool

The `sqlservercache` command, formerly provided by the `Microsoft.Extensions.Caching.SqlConfig` package, has been replaced by the `sql-cache` tool, available through the `Microsoft.Extensions.Caching.SqlConfig.Tools` package:

```json
{
  "tools": {
    "Microsoft.Extensions.Caching.SqlConfig.Tools": "1.0.0-preview2-final"
  }
}
```

### User secrets manager

The `user-secret` command, formerly provided by the `Microsoft.Extensions.SecretManager` package, has been replaced by the `user-secrets` tool, available through the `Microsoft.Extensions.SecretManager.Tools` package:

```json
{
  "tools": {
    "Microsoft.Extensions.SecretManager.Tools": "1.0.0-preview2-final"
  }
}
```

### File watcher

The `watch` command, formerly provided by the `Microsoft.Dnx.Watcher` package, has been replaced by the `watch` tool, available through the `Microsoft.DotNet.Watcher.Tools` package:

```json
{
  "tools": {
    "Microsoft.DotNet.Watcher.Tools": "1.0.0-preview2-final"
  }
}
```

For more information on the file watcher, see **Dotnet watch** in  [Tutorials](../tutorials/index.md).

## Hosting

### Creating the web application host

ASP.NET Core 1.0 apps are console apps; you must define an entry point for your app that sets up a web host and runs it. Below is an example from the startup code for one of the Web Application templates in Visual Studio:

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        var host = new WebHostBuilder()
            .UseKestrel()
            .UseContentRoot(Directory.GetCurrentDirectory())
            .UseIISIntegration()
            .UseStartup<Startup>()
            .Build();

        host.Run();
    }
}
```

You must add the `emitEntryPoint` to the `buildOptions` section of your application's *project.json*:

```json
{
  "buildOptions": {
    "emitEntryPoint": true
  }
}
```

### Class and interface renames

All classes and interfaces prefixed with `WebApplication` have been renamed to start with `WebHost`:

|ASP.NET 5 RC1|ASP.NET Core 1.0|
|---|---|
|IWebApplicationBuilder|IWebHostBuilder|
|WebApplicationBuilder|WebHostBuilder|
|IWebApplication|IWebHost|
|WebApplication|WebHost|
|WebApplicationOptions|WebHostOptions|
|WebApplicationDefaults|WebHostDefaults|
|WebApplicationService|WebHostService|
|WebApplicationConfiguration|WebHostConfiguration|

### Content root and web root

The application base path is now called the content root.

The web root of your application is no longer specified in your *project.json* file. It is defined when setting up the web host and defaults to `wwwroot`. Call the `UseWebRoot` extension method to specify a different web root folder. Alternatively, you can specify the web root folder in configuration and call the `UseConfiguration` extension method.

### Server address binding

The server addresses that your application listens on can be specified using the `UseUrls` extension method or through configuration.

Specifying only a port number as a binding address is no longer supported. The default binding address is http://localhost:5000

### Hosting configuration

The `UseDefaultHostingConfiguration` method is no longer available. The only configuration values read by default by `WebHostBuilder` are those specified in environment variables prefixed with `ASPNETCORE_*`. All other configuration sources must now be added explicitly to an `IConfigurationBuilder` instance. See [Configuration](../fundamentals/configuration.md) for more information.

The environment key is set with the `ASPNETCORE_ENVIRONMENT` environment variable. `ASPNET_ENV` and `Hosting:Environment` are still supported, but generate a deprecated message warning.

### Hosting service changes

Dependency injection code that uses `IApplicationEnvironment` must now use `IHostingEnvironment`. For example, in your `Startup` class, change:

```csharp
public Startup(IApplicationEnvironment applicationEnvironment)
   ```

To:

```csharp
public Startup(IHostingEnvironment hostingEnvironment)
   ```

## Kestrel

Kestrel configuration has changed. [This GitHub announcement](https://github.com/aspnet/Announcements/issues/168) outlines the changes you must make to configure Kestrel if you are not using default settings.

### Controller and action results renamed

The following `Controller` methods have been renamed and moved to `ControllerBase``:

|ASP.NET 5 RC1|ASP.NET Core 1.0|
|---|---|
|HttpUnauthorized|Unauthorized|
|HttpNotFound (and its overloads)|NotFound|
|HttpBadRequest (and its overloads)|BadRequest|

The following action result types have also been renamed:

|ASP.NET 5 RC1|ASP.NET Core 1.0|
|---|---|
|Microsoft.AspNet.Mvc.HttpOkObjectResult|Microsoft.AspNetCore.Mvc.OkObjectResult|
|Microsoft.AspNet.Mvc.HttpOkResult|Microsoft.AspNetCore.Mvc.OkResult|
|Microsoft.AspNet.Mvc.HttpNotFoundObjectResult|Microsoft.AspNetCore.Mvc.NotFoundObjectResult|
|Microsoft.AspNet.Mvc.HttpNotFoundResult|Microsoft.AspNetCore.Mvc.NotFoundResult|

## ASP.NET 5 MVC compile views

To compile views, set the `preserveCompilationContext` option in *project.json* to preserve the compilation context, as shown here:

```json
{
  "buildOptions": {
    "preserveCompilationContext": true
  }
}
```

### Changes in views

Views now support relative paths.

The Validation Summary Tag Helper `asp-validation-summary` attribute value has changed. Change:

```html
<div asp-validation-summary="ValidationSummary.All"></div>
   ```

To:

```html
<div asp-validation-summary="All"></div>
   ```

### Changes in ViewComponents

* The sync APIs have been removed

* `Component.Render()`, `Component.RenderAsync()`, and `Component.Invoke()` have been removed

* To reduce ambiguity in View Component method selection, we've modified the selection to only allow exactly one `Invoke()` or `InvokeAsync()` per View Component

* `InvokeAsync()` now takes an anonymous object instead of separate parameters

* To use a view component, call `@Component.InvokeAsync("Name of view component", <parameters>)` from a view. The parameters will be passed to the `InvokeAsync()` method. The following example demonstrates the `InvokeAsync()` method call with two parameters:

ASP.NET 5 RC1:

```csharp
@Component.InvokeAsync("Test", "MyName", 15)
   ```

ASP.NET Core 1.0:

```csharp
@Component.InvokeAsync("Test", new { name = "MyName", age = 15 })
@Component.InvokeAsync("Test", new Dictionary<string, object> {
                       ["name"] = "MyName", ["age"] = 15 })
@Component.InvokeAsync<TestViewComponent>(new { name = "MyName", age = 15})
```

### Updated controller discovery rules

There are changes that simplify controller discovery:

The new `ControllerAttribute` can be used to mark a class (and it's subclasses) as a controller. A class whose name doesn't end in `Controller` and derives from a base class that ends in `Controller` is no longer considered a controller. In this scenario, `ControllerAttribute` must be applied to the derived class itself or to the base class.

A type is considered a controller if **all** the following conditions are met:

* The type is a public, concrete, non-open generic class

* `NonControllerAttribute` is **not** applied to any type in its hierarchy

* The type name ends with `Controller`, or `ControllerAttribute` is applied to the type or one of its ancestors.

> [!NOTE]
> If `NonControllerAttribute` is applied anywhere in the type hierarchy, the discovery conventions will never consider that type or its descendants to be a controller. In other words, `NonControllerAttribute` takes precedence over `ControllerAttribute`.

## Configuration

The `IConfigurationSource` interface has been introduced to represent the configuration used to build an `IConfigurationProvider`. It is no longer possible to access the provider instances from `IConfigurationBuilder`, only the sources. This is intentional, and may cause loss of functionality as you can no longer do things like call `Load` on the provider instances.

File-based configuration providers support both relative and absolute paths to configuration files. If you want to specify file paths relative to your application's content root, you must call the `SetBasePath` extension method on `IConfigurationBuilder``:

<!-- literal_block {"ids": [], "linenos": false, "xml:space": "preserve", "language": "csharp", "highlight_args": {"hl_lines": [4]}} -->

```csharp
public Startup(IHostingEnvironment env)
{
    var builder = new ConfigurationBuilder()
        .SetBasePath(env.ContentRootPath)
        .AddJsonFile("appsettings.json");
}
```

### Automatic reload on change

The `IConfigurationRoot.ReloadOnChanged` extension method is no longer available. File-based configuration providers now provide extension methods to `IConfigurationBuilder` that allow you to specify whether configuration from those providers should be reloaded when there are changes in their files. See `AddJsonFile`, `AddXmlFile` and
`AddIniFile` for details.

## Logging

`LogLevel.Verbose` has been renamed to `Trace` and is now considered less severe than `Debug`.

The `MinimumLevel` property has been removed from `ILoggerFactory`. Each logging provider now provides extension methods to `ILoggerFactory` that allow specifying a minimum logging level. See `AddConsole`, `AddDebug`, and
`AddEventLog` for details.

## Identity

The signatures for the following methods or properties have changed:

|ASP.NET 5 RC1|ASP.NET Core 1.0|
|---|---|
|ExternalLoginInfo.ExternalPrincipal|ExternalLoginInfo.Principal|
|User.IsSignedIn()|SignInManager.IsSignedIn(User)|
|UserManager.FindByIdAsync(HttpContext.User.GetUserId())|UserManager.GetUserAsync(HttpContext.User)|
|User.GetUserId()|UserManager.GetUserId(User)|

To use Identity in a view, add the following:

```csharp
@using Microsoft.AspNetCore.Identity
@inject SignInManager<TUser> SignInManager
@inject UserManager<TUser> UserManager
```

## Working with IIS

The package `Microsoft.AspNetCore.IISPlatformHandler` has been replaced by `Microsoft.AspNetCore.Server.IISIntegration`.

HttpPlatformHandler has been replaced by the [ASP.NET Core Module (ANCM)](../hosting/aspnet-core-module.md). The *web.config* file created by the *Publish to IIS tool* now configures IIS to the ANCM instead of HttpPlatformHandler to reverse-proxy requests.

The ASP.NET Core Module must be configured in *web.config*:

```xml
<configuration>
  <system.webServer>
    <handlers>
      <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModule" resourceType="Unspecified"/>
    </handlers>
    <aspNetCore processPath="%LAUNCHER_PATH%" arguments="%LAUNCHER_ARGS%"
                stdoutLogEnabled="false" stdoutLogFile=".\logs\stdout"
                forwardWindowsAuthToken="false"/>
  </system.webServer>
</configuration>
```

The *Publish to IIS tool* generates a correct *web.config*. See [Publishing to IIS](../publishing/iis.md) for more details.

IIS integration middleware is now configured when creating the `Microsoft.AspNetCore.Hosting.WebHostBuilder`, and is no longer called in the `Configure` method of the `Startup` class:

```csharp
var host = new WebHostBuilder()
    .UseIISIntegration()
    .Build();
```

### Web Deploy changes

Delete any *{app name} - Web Deploy-publish.ps1* scripts created with Visual Studio web deploy using ASP.NET 5 RC1. The ASP.NET 5 RC1 scripts (which are DNX based) are not compatible with dotnet based scripts. Use Visual Studio to generate new web deploy scripts.

### applicationhost.config changes

An *applicationhost.config* file created with ASP.NET 5 RC1 will point ASP.NET Core to an invalid *content root* location. With such a *applicationhost.config* file, ASP.NET Core will be configured with *content root/web root* as the *content root* folder and therefore look for *web.config* in `Content root/wwwroot`. The *web.config* file must be in the *content root* folder. When configured like this, the app will terminate with an HTTP 500 error.

## Updating Launch Settings in Visual Studio

Update `launchSettings.json` to remove the web target and add the following:

```json
{
  "WebApplication1": {
    "commandName": "Project",
    "launchBrowser": true,
    "launchUrl": "http://localhost:5000",
    "environmentVariables": {
      "ASPNETCORE_ENVIRONMENT": "Development"
    }
  }
}
```

## Server garbage collection (GC)

You must turn on server garbage collection in *project.json* or *app.config* when running ASP.NET projects on the full .NET Framework:

<!-- literal_block {"ids": [], "linenos": false, "xml:space": "preserve", "language": "json", "highlight_args": {"hl_lines": [4]}} -->

```json
 {
   "runtimeOptions": {
     "configProperties": {
       "System.GC.Server": true
     }
   }
 }
```