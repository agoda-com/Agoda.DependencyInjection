# Agoda IoC Extensions 
![GitHub Workflow Status (branch)](https://img.shields.io/github/workflow/status/agoda-com/Agoda.IoC/.NET%20Core%20Build%20and%20Publish/main)
![Nuget](https://img.shields.io/nuget/v/agoda.ioc.netcore)
![Codecov](https://img.shields.io/codecov/c/github/agoda-com/agoda.ioc)

C# IoC extension library, used at Agoda for Registration of classes into IoC container based on Attributes. 

## The Problem?

In some of our larger projects at Agoda, the Dependency injection registration was done in a single or set of "configuration" classes. These large configuration type files are troublesome due to frequency of merge conflicts. Also to look at a normal class and know if it will be run as a singleton or transient you need to dig into these configuration classes.

By declaring the IoC configuration at the top of each class in an attribute it makes it immediately clear to the developer what the class's lifecycle is when running, and avoids large complex configuration classes that are prone to merge conflicts.

## Adding to your project

Install the package, then add to your Startup like below.

```powershell
Install-Package Agoda.IoC.NetCore
```

```csharp
        public void ConfigureServices(IServiceCollection services)
        {
            services.AutoWireAssembly(new[]{typeof(Startup).Assembly}, isMockMode);
        }
```

You need to pass in an array of all the assemblies you want to scan in your project for registration. As well as a boolean indicating if your application is running in Mocked mode or not.

## Usage in your project

The basic usage of this project allows you to use 3 core attributes on your classes (RegisterTransient, RegisterPerRequest, RegisterSingleton) like the following code:

```csharp

    // Simple registration
    public interface IService {}
    [RegisterSingleton] /// replaces services.AddSingleton<IService, Service>();
    public class Service : IService {}

```
Replaces something like this in your startup.
```csharp
services.AddSingleton<IService , Service>();
```

The library will assembly scan your app at start-up and register services in IoC container based on the attribute and its parameters.

Factory options are available for registration with the attributes, as seen below, the factory needs a "Build" method

```csharp
    [RegisterSingleton(Factory = typeof(MySingletonFactory))]
    public class SingletonFromFactory : ISingletonFromFactory
    {
        // implementation here
    }
    public class MySingletonFactory : IComponentFactory<ISingletonFromFactory>
    {
        public ISingletonFromFactory Build(IComponentResolver c)
        {
            return new SingletonFromFactory("test");
        }
    }
```

For services with multiple interfaces the interface can be explicitly declared like below

```csharp
// This class implements 2 interfaces, but we explicitly tell it to register only 1.
    [RegisterTransient(For = typeof(IExplicitlyRegisteredInterface))]
    public class ServiceWithExplicitInterfaceRegistration : IExplicitlyRegisteredInterface, IInterfaceThatShouldNotGetRegistered {}
```
It can also be used to register multiple instances

```csharp

    [RegisterSingleton(For = typeof(IMultipleAttributes1))]
    [RegisterSingleton(For = typeof(IMultipleAttributes2))]
    public class MultipleAttributes : IMultipleAttributes1, IMultipleAttributes2 {}
```

And may more options...

## Mocked Mode?

Mocked mode is used for mocking external dependencies, this should be used on repository type classes that access a database for example. So you can run system tests on your application without the need for external dependencies.

Below example demonstrates using attributes to indicate a mock option for a registration.

```csharp

    // Mocked registration
    public class MockService : IServiceWithMock {}
    public interface IServiceWithMock{}
    [RegisterTransient(Mock = typeof(MockService))]
    public class ServiceWithMock : IServiceWithMock {}

```

## A Unity 3.5 Project?

Some of the old legacy systems at Agoda run an old version of unity, and this library was originally developed against that. These days everything is moving towards net core, but we still decided to publish the original unity library as well.

## This is using reflection, isn't that slow?

Reflection is not slow, it's pretty fast in C# actually. Where you will hit problems with reflection and speed is if you are doing thousands or millions of operations, like in a http request on a busy website, using it at startup like this you are doing very few operations and only when the application starts.

## Dedication
A large amount of the code in this repository was written by Michael Alastair Chamberlain who is no longer with us, so it is with this community contribution we remember him.
