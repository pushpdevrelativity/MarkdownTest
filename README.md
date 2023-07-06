Migrate Windsor DI: Document Review Service
-------------------------------------------
* [Introduction](#introduction)
* [Functional Description](#functional-description)
* [Windsor DI](#windsor-di)
* [.NET Core API Built-in DI](#net-core-api-built-in-di)

### Introduction

DI is an established design pattern describing how objects acquire their dependencies. This pattern is often facilitated by an Inversion of Control (IoC) container, which is used at runtime to resolve and inject dependencies as objects are instantiated. The Legacy Application Uses Windsor DI, which needs to be Migrated to the .NET Core API built in DI provided by Microsoft, this documentation is a small comparison of the code changes and the steps followed. 

  

### Functional Description

The new .NET Core DI abstraction has proven to be a far from trivial task for most DI Containers. Some Containers are simply not compatible with the way Microsoft built their DI Container and Castle Windsor is a good example.

The Castle Windsor maintainers have tried [for quite some time](https://github.com/castleproject/Windsor/issues/120) to build such an adapter, but even after consulting Microsoft about this, Microsoft [acknowledged](https://twitter.com/davidfowl/status/915712411704758272) ([copy](https://i.stack.imgur.com/sUAM0.png)) Castle Windsor is incompatible to how MS views the world of containers.

Castle isn't the only container that falls into this category. Contains such as Simple Injector, Ninject, Unity and StructureMap have proven to be incompatible with the new .NET Core DI Abstraction as well. Although StructureMap actually has an adapter, their adapter isn't 100% compatible with the abstraction, which might obviously lead to problems when the ASP.NET Core framework, or a third-party library, starts to depend on that incompatible behavior. And even for other containers with an adapter, like Autofac and LightInject, problems with their adapters seem to keep popping up, which proves (IMO) the underlying model is flawed

### Windsor DI

Reference:     

[Windsor/README.md at master · castleproject/Windsor · GitHub](https://github.com/castleproject/Windsor/blob/master/docs/README.md)

Below is the **code snippet for Windsor DI** implementation
```
// application starts...
var container = new WindsorContainer();
 
// adds and configures all components using WindsorInstallers from executing assembly
container.Install(FromAssembly.This());
 
// instantiate and configure root component and all its dependencies and their dependencies and...
var king = container.Resolve<IKing>();
king.RuleTheCastle();
 
// clean up, application exits
container.Dispose();
 
public class RepositoriesInstaller : IWindsorInstaller
{
    public void Install(IWindsorContainer container, IConfigurationStore store)
    {
        container.Register(Classes.FromThisAssembly()
                            .Where(Component.IsInSameNamespaceAs<King>())
                            .WithService.DefaultInterfaces()
                            .LifestyleTransient());
    }
}
```
  

### .NET Core API Built-in DI

Reference:     

[Dependency Injection Example - GitHub](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/fundamentals/dependency-injection.md)

>**_NOTE:_** **Microsoft DI is the replacement for the legacy Castle Windsor DI in the Migrated Code.**

Below is the **code snippet for .NET DI** implementation

```
public static class ServicesInstaller
    {
        // This is called from Startup.cs
        // This is where you can set up DI specific to your service
#pragma warning disable CA1506
        internal static void Install(IServiceCollection services)
        {
#pragma warning restore CA1506
            services.AddScoped<IOpenTelemetryManager, OpenTelemetryManager>();
            services.AddScoped<IOpenTelemetryConfigurationManager, OpenTelemetryConfigurationManager>();
            services.AddTransient<IPersistentHighlightServiceHelper, PersistentHighlightServiceHelper>();
            services.AddTransient<IPersistentHighlightManager, PersistentHighlightManager>();
            services.AddScoped<IPersistentHighlightSetRepository, PersistentHighlightSetRepository>();
            services.AddScoped<IRelativityFieldHelper, RelativityFieldHelper>();
            services.AddScoped<IRelativityCodeHelper, RelativityCodeHelper>();
}
}
```

### SERVICE LIFETIMES ###
--------------------------
>You can use three lifetimes with the default dependency injection framework. These lifetimes affect how the service is resolved and disposed of by the service provider.

* [Transient](#by-context-or-functionality): A new service instance is created each time a service is requested from the service provider. If the service is disposable, the service scope will monitor all instances of the service and destroy every instance of the service created in that scope when the service scope is disposed.
* [Singleton](#by-context-or-functionality): Only one instance of the service is created if it’s not already registered as an instance. Single instance services are tracked by the root scope if they are created by the framework. This means that single instance services will not be disposed until the root scope is disposed which is usually occur when the application exits. Please note that if your singleton service is disposable and you didn’t register it with its implemented type or service provider factory and registered it as an instance, framework does not track and dispose it. In this case you should manually dispose it after the service container is disposed.
* [Scoped](#by-context-or-functionality): A new instance of a service is created in each scope. It will act as if it is singleton within that scope. If the service is disposable it will be disposed when service scope is disposed.

#### CHOOSING SERVICE LIFETIME FOR YOUR SERVICES ####

##### BY CONTEXT OR FUNCTIONALITY #####
> Use Singleton:
  * _If your service has a shared state such as cache service_. Singleton services with mutable state should consider using a locking mechanism for thread safety.
  * _If your service is stateless_. If your service implementation is very lightweight and infrequently used, you might also consider registering it as transient.

> Use Scoped:
  * _If your service should act as a singleton within the scope of the request_. In Asp.Net Core each request has in own service scope. Database and repository services are often registered as scoped services. Default registration of DbContext in EntityFramework Core is also scoped. Scoped lifetime ensures that all the services created within the request shares the same DbContext.

> Use Transient:
  * _If your service holds a private (non-shared) state for the execution context._
  * _If your service will be used by multiple threads concurrently and it is not thread safe._
  * _If your service has a transient dependency, such as HttpClient, that has a short intended lifetime._
##### BY DEPENDENCIES #####
> Singleton:
  * Singleton services can inject other singleton service. Singleton service can also inject transient services but be aware that transient service will be alive as long as the singleton service which is usually the lifetime of the application.
  * Singleton services should not inject scoped services as a best practice. Because scoped services will act as a singleton, which usually is not the intended way by design for scoped services.
> Scoped:
  * Scoped services can inject other scoped services and singleton services. Scoped services can also use transient services but you should review your design and see if you can register your transient service as scoped service.
> Transient:
  * Transient services can inject all type of services. They are usually intended as short-lived services.
  * If you need to use transient services in singleton or scoped services you should consider service factory approach for transient services. If your transient service implementation is a disposable class, the service factory approach is recommended.
 
