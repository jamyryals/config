# Config.Net ![](https://aloneguid.visualstudio.com/_apis/public/build/definitions/323c5f4c-c814-452d-9eaf-1006c83fd44c/8/badge) [![NuGet](https://img.shields.io/nuget/v/Config.Net.svg?maxAge=2592000?style=flat-square)](https://www.nuget.org/packages/Config.Net/)

A comprehensive easy to use and powerful .NET configuration library, fully covered with unit tests and tested in the wild on thousands of servers and applications.

## Purpose

Config.Net addresses the following problems:

* Configuration in multiple places
* Converting types between different providers
* Hardcoded configuration keys accross the solution
* Abstracts specific configuration source implementation 

Config.Net accomplishes this by exposing an abstract configuration interface and provides most common implementations for configuration sources like app.config, environment variables etc.

> **Note:** Current version (v2) is not compatible with v1. If you need to come back to v1 documentation please follow [this link.](https://github.com/aloneguid/config/blob/master/README.v1.markdown)

## Quick Start

Usually developers will hardcode reading cofiguration values from different sources like app.config, local json file etc. For instance, consider this code example:

```csharp
var clientId = ConfigurationManager.AppSettings["AuthClientId"];
var clientSecret = ConfigurationManager.AppSettings["AuthClientSecret"];

```

You would guess that this code is trying to read a configuration setting from the local app.config file by name and that might be true, however there are numerous problems with this approach:

* Settings are referenced by a hardcoded string name which is prone to typos and therefore crashes in runtime.
* There is no easy way to find out where a particular setting is used in code, except for performing a fulltext search (provided that the string was not mistyped)
* If you decide to store configuration in a different place the code must be rewritten.

Welcome to Config.Net which solves most of those problems. Let's rewrite this abomination using the Config.Net approach. First, we need to define a configuration container which describes which settings are used in your application or a library:


### Declare Settings Container

```csharp
using Config.Net;

public class AllSettings : SettingsContainer
{
    public readonly Option<string> AuthClientId;

    public readonly Option<string> AuthClientSecret;

    protected override void OnConfigure(IConfigConfiguration configuration)
    {
         configuration.UseAppConfig();
    }
}
```

Let's go through this code snippet:
* We have declared `AllSettings` class which will store configuration for your application. All configuration classes must derive from `SettingsContainer`.
* Two strong-typed configuration options were declared. Note they are both `readonly` which is another plus towards code quality.
* `Option<T>` is a configuration option definition in Config.Net where the generic parameter specifies the type.
* `OnConfigure` method implementation specifies that app.config should be used as a configuration store.

### Use Settings

Once a container has been defined, start using the settings. For instance:

```csharp
var c = new AllSettings();

string clientId = c.AuthClientId;
string clientSecret = c.AuthClientSecret;
```

Two things worth noting in this snippet:
* An instance of `AllSettings` container was created. Normally you would create an instance of a settings container per application instance for performance reasons.
* The settings were read from the settings container. Note the syntax and that for example `AuthClientId` is defined as an `Option<string>` but casted to `string`. This is because `Option<T>` class has implicit casting operator to `T` defined.


### Using Multiple Sources

`OnConfigure` method is used to prepare settings container for use. You can use it to add multiple configuration sources. To get the list of sources use IntelliSense (type dot-Use after `configuration`). For instance: 

```csharp
protected override void OnConfigure(IConfigConfiguration configuration)
{
    configuration.UseAppConfig();
    configuration.UseEnvironmentVariables();
}

```

This method implementation causes the container to use both app.config and environment variables as configuration source.

The order in which sources are added is important - Config.Net will try to read the source in the configured order and return the value from the first store where it exists.

### Writing Settings

Some configuration stores support writing values. You can write the value back by calling the `.Write()` method on an option definition:

```csharp
c.AuthClientId.Write("new value");
```

Config.Net will write the value to the first store which supports writing. If none of the stores support writing the call will be ignored.


## Caching

By default Config.Net caches configuration values for 1 hour. After that it will read it again from the list of configured stores. If you want to change it to something else set the variable in the `OnConfigure` method:

```csharp
protected override void OnConfigure(IConfigConfiguration configuration)
{
    configuration.CacheTimeout = TimeSpan.FromMinutes(1);	//sets caching to 1 minute

    configuration.UseAppConfig();
    configuration.UseEnvironmentVariables();
}
```

Setting `CacheTimeout` to `TimeSpan.Zero` disables caching completely.


# Available Stores

The list of available built-in and external stores is maintained on this page:


| Name                 | Readable | Writeable | Package  | Purpose                  |
|----------------------|----------|-----------|----------|--------------------------|
| [AppConfig](doc/Stores_AppConfig.md)            | v        | x         | internal | .NET app.config files    |
| [EnvironmentVariables](doc/Stores_EnvironmentVariables.md) | v        | v         | internal | OS environment variables |
| [IniFile](doc/Stores_IniFile.md)              | v        | v         | internal | INI files |
| [InMemory](doc/Stores_InMemory.md)             | v        | v         | internal | In-memory storage |
| [Azure](doc/Stores_Azure.md)                | v        | x         | [Config.Net.Azure](https://www.nuget.org/packages/Config.Net.Azure) | Azure's ConfigurationManager |
| [AzureTable](doc/Stores_AzureTable.md)           | v        | v         | [Config.Net.Azure](https://www.nuget.org/packages/Config.Net.Azure) | Azure Table Storage |
| [AzureKeyVault](doc/Stores_AzureKeyVault.md)        | v        | v          | [Config.Net.Azure](https://www.nuget.org/packages/Config.Net.Azure) | Azure Key Vault Secrets |
