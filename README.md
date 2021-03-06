# grpc-dotnet-validator
Request message validator middleware for [Grpc.AspNetCore](https://github.com/grpc/grpc-dotnet)

![](https://github.com/enif-lee/grpc-dotnet-validator/workflows/Build/badge.svg)
![](https://github.com/enif-lee/grpc-dotnet-validator/workflows/Test/badge.svg)
[![Nuget](https://img.shields.io/nuget/v/GrpcExtensions.AspNetCore.Validation)](https://www.nuget.org/packages/GrpcExtensions.AspNetCore.Validation)


## Feature

- Support async validation for unary, streaming call
- Support IoC LifeStyle scopes and dependency injection
- Profile for validators
- Scan validators and profiles from assembly

## How to use.

This package is integrated with [Fluent Validation](https://github.com/JeremySkinner/FluentValidation). 
If you want to know how build your own validation rules, please checkout [Fluent Validation Docs](https://fluentvalidation.net/start)

#### Add custom message validator

```csharp
// Write own message validator
public class HelloRequestValidator : AbstractValidator<HelloRequest>
{
    public HelloRequestValidator()
    {
        RuleFor(request => request.Name).NotEmpty();
    }
}

public class Startup
{
    // ...
    public void ConfigureServices(IServiceCollection services)
    {
        // 1. Enable message validation feature.
        services.AddGrpc(options => options.EnableMessageValidation());

        // 2. Add custom validators for messages, default scope is scope.
        services.AddValidator(typeof(HelloRequestValidator));
        services.AddValidator<HelloRequestValidator>();
        services.AddValidator<HelloRequestValidator>(LifeStyle.Singleton);
    }
    // ...
}
```

Then, If the message is invalid, Grpc Validator return with `InvalidArgument` code and empty message object.

#### Add inline custom validator

if you don't want to create many validation class for simple validation rule in your project,
you just use below inline validator feature like below example.

Note that, Inline validator always be registered **singleton** in your service collection.
Because, There are no way for using other dependency.

```csharp
public class Startup
{
    // ...
    public void ConfigureServices(IServiceCollection services)
    {
        // 1. Enable message validation feature.
        services.AddGrpc(options => options.EnableMessageValidation());

        // 2. Add inline validators for messages, scope is always singleton
        services.AddInlineValidator<HelloRequest>(rules => rules.RuleFor(request => request.Name).NotEmpty());
    }
    // ...
}
```


#### Profiling validators

If you don't want to make a mess your startup class by registering validators, you implement validator profile and use it.

```cs
public class SampleProfile : ValidatorProfileBase
{
    public SampleProfile()
    {
        CreateInlineValidator<SampleRequest>()
            .RuleFor(r => r.Data).NotNull().NotEmpty()
            .RuleFor(r => r.Name).NotNull();
        AddValidator<SampleRequestValidator>();
        AddValidator<SampleRequestValidator>(ServiceLifetime.Singleton);
    }
}

// Then in your Startup class
public void ConfigureServices(IServiceCollection services)
{
    services.AddGrpc(options => options.EnableMessageValidation());
    services.AddValidatorProfile<SampleProfile>();
}
```

#### Scan and register profiles/validators from assembly.


```cs
// Place somewhere validator or profile class.
public class HelloRequestValidator : AbstractValidator<HelloRequest>
{
    public HelloRequestValidator()
    {
        RuleFor(request => request.Name).NotEmpty();
    }
}

public class SampleProfile : ValidatorProfileBase
{
    public SampleProfile()
    {
        CreateInlineValidator<SampleRequest>()
            .RuleFor(r => r.Data).NotNull().NotEmpty()
            .RuleFor(r => r.Name).NotNull();
        AddValidator<SampleRequestValidator>();
        AddValidator<SampleRequestValidator>(ServiceLifetime.Singleton);
    }
}

// Then in your Startup class
public void ConfigureServices(IServiceCollection services)
{
    services.AddGrpc(options => options.EnableMessageValidation());

    // Scan profile and validators from calling assembly.
    services.AddValidatorsFromAssemblies(); 
    services.AddProfilesFromAssembly();

    // Scan profiles and validators from specific assembly.
    services.AddValidatorsFromAssemblies(typeof(InternalLibarary).Assembly); 
    services.AddProfilesFromAssembly(typeof(InternalLibarary).Assembly);
}
```


#### Customize validation failure message.

If you want to custom validation message handler for using your own error message system,
Just implement IValidatorErrorMessageHandler and put it service collection.

```csharp
public class CustomMessageHandler : IValidatorErrorMessageHandler
{
    public Task<string> HandleAsync(IList<ValidationFailure> failures)
    {
        return Task.FromResult("Validation Error!");
    }
}

public class Startup
{
    // ...
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddGrpc(options => options.EnableMessageValidation());

        // Just put at service collection your own custom message handler that implement IValidatorErrorMessageHnadler.
        // This should be placed before calling AddInlineValidator() or AddValidator();
        services.AddSingleton<IValidatorErrorMessageHanlder>(new CustomMessageHandler())
        services.AddInlineValidator<HelloRequest>(rules => rules.RuleFor(request => request.Name).NotEmpty());
    }
    // ...
}
```

## How to test my validation

If you want to write integration tests. [This test sample](src/Grpc.AspNetCore.FluentValidation.Test/Integration/) may help you.


## Versioning

This pakage`s versioning is following version of [Grpc.AspNetCore](https://github.com/grpc/grpc-dotnet)

 