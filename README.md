Migrate Windsor DI: Document Review Service
-------------------------------------------
* [Introduction](#introduction)
* [Functional Description](#functional-description)
* [Windsor DI](#windsor-di)
* [.NET Core API Built-in DI](#net-core-api-built-in-di)

### Introduction<a name="introduction"></a>

DI is an established design pattern describing how objects acquire their dependencies. This pattern is often facilitated by an Inversion of Control (IoC) container, which is used at runtime to resolve and inject dependencies as objects are instantiated. The Legacy Application Uses Windsor DI, which needs to be Migrated to the .NET Core API built in DI provided by Microsoft, this documentation is a small comparison of the code changes and the steps followed. 

  

### Functional Description<a name="functional-description"></a>

The new .NET Core DI abstraction has proven to be a far from trivial task for most DI Containers. Some Containers are simply not compatible with the way Microsoft built their DI Container and Castle Windsor is a good example.

The Castle Windsor maintainers have tried [for quite some time](https://github.com/castleproject/Windsor/issues/120) to build such an adapter, but even after consulting Microsoft about this, Microsoft [acknowledged](https://twitter.com/davidfowl/status/915712411704758272) ([copy](https://i.stack.imgur.com/sUAM0.png)) Castle Windsor is incompatible to how MS views the world of containers.

Castle isn't the only container that falls into this category. Contains such as Simple Injector, Ninject, Unity and StructureMap have proven to be incompatible with the new .NET Core DI Abstraction as well. Although StructureMap actually has an adapter, their adapter isn't 100% compatible with the abstraction, which might obviously lead to problems when the ASP.NET Core framework, or a third-party library, starts to depend on that incompatible behavior. And even for other containers with an adapter, like Autofac and LightInject, problems with their adapters seem to keep popping up, which proves (IMO) the underlying model is flawed

### Windsor DI<a name="windsor-di"></a>

Reference:     

[Windsor/README.md at master · castleproject/Windsor · GitHub](https://github.com/castleproject/Windsor/blob/master/docs/README.md)

|**Windsor DI - Code snippet**                                                                                                                  |
|-----------------------------------------------------------------------------------------------------------------------------------------------|
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
  

### .NET Core API Built-in DI<a name="net-core-api-built-in-di"></a>

|**.NET DI - Code snippet**                                                                                                                     |
|-----------------------------------------------------------------------------------------------------------------------------------------------|
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
