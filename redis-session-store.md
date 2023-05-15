
# Redis Session Server

## Overview

* Having in-memory session state in your application is not a good cloud-native practice
* One of the benefits of a cloud environment like TAS is elasticity if your applications
* This includes the ability to scale up and down, deploy and upgrade without user disruption
* With applications more frequently being taken in and out of service, any in-memory state would be lost
* The cloud-native best practice is to use an external datastore for session and state management
* TAS allows teams the ability to create an on-demand instance of Redis, which can be used as a session state provider

## Prerequisite

* Completion of the **Setup** step in [Configuration](configuration.md) is required for this guide

## Install Nuget Packages

```powershell
# Microsoft provided session state provider for Redis
PM> Install-Package Microsoft.Web.RedisSessionStateProvider -Version 4.0.1

# Connection helper for CloudFoundry to discover and bind Redis to the app
PM> Install-Package Steeltoe.Connector.CloudFoundry

#
```

## Add sessionState config to `Web.Release.config`

* You may want to consider putting in `Web.Release.config` so that you can continue to develop locally without needing Redis

```xml
<sessionState mode="Custom" customProvider="MySessionStateStore" xdt:Transform="Insert">
    <providers>
        <add name="MySessionStateStore" type="Microsoft.Web.Redis.RedisSessionStateProvider" settingsClassName="RedisConnectionHelper" settingsMethodName="GetConnectionString" />
    </providers>
</sessionState>
```

## Create `RedisConnectionHelper.cs`

* **NOTE**: This class name and method name is reference in the `Web.config` step above. Changing the name/location needs to be reflected there as well
```csharp
using OnboardingApp.App_Start;
using Steeltoe.CloudFoundry.Connector.Redis;

namespace OnboardingApp
{
    public class RedisConnectionHelper
    {
        public static string GetConnectionString()
        {
            var redisConnectionFactory = ApplicationConfig.Configuration.CreateRedisServiceConnectorFactory();

            return redisConnectionFactory.GetConnectionString();
        }
    }
}
```

## Create an On-Demand Redis

```bash
cf create-service p.redis on-demand-cache onboarding-redis
```

## Wait for redis to be ready
* Use `cf services` to check status of the service creation
* Redis is ready when **last operation** changes from `create in progress` to `create succeeded`
```
cf services
Getting services in org sandbox / space test as admin...

name               service         plan              bound apps      last operation       broker           upgrade available
onboarding-redis   p.redis         on-demand-cache                   create in progress   redis-odb        no
```

```
cf services
Getting services in org sandbox / space test as admin...

name               service         plan              bound apps      last operation     broker           upgrade available
onboarding-redis   p.redis         on-demand-cache                   create succeeded   redis-odb        no
```

## Bind the Redis service and Restage

```bash
cf bind-service onboardingapp onboarding-redis
cf restage onboardingapp
```

## Using Session Store
* The Session store works as it did prior to Redis by Calling `Session["key"]` from an aspx page or using `HttpContext.Current.Session`

```csharp
// setting session values
Session["str1"] = "Hello";

// retrieving session values
string str1 = Session["str1"];
```

## Session Store with complex objects

* If migrating from In-Memory (InProc) session store, the app may not consider if session stored objects are serializable
* In order for any object to be stored in an external session store, it must be easily converted (serialized) to binary and back to an object
* Otherwise it won't be able to store the object

### Primitives serialize easily
* Primitives such as strings, booleans, and number types easily convert to binary, even Lists
* The following should work:

```csharp
Session["str1"] = "Hello";
Session["bool1"] = true;
Session["int1"] = 3;
Session["double2"] = 1.3;
Session["intArray"] = new int[] {1, 2, 3};
Session["intList"] = new List<int> { 1, 2, 3 };
```

* The following however will not work because `Book` is an object and not serializable:
```csharp
Session["Started"] = new Book { Id = 1, Name = "Using Redis", Author = "Developer" };
```

* If `Book` were to be annotated with `Serializable` and contained only serializable fields then it would work
* The below works because Book is annotated with `Serializable` and contains only primitives

```csharp
using System;

namespace OnboardingApp
{
    [Serializable]
    public class Book
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public string Author { get; set; }
    }
}
```

* If `Book` were to contain a `genres` field that is an object of type `Genres` then `Genres` would also need to be annotated with `Serializable`

 ```csharp
 using System;

namespace OnboardingApp
{
    [Serializable]
    public class Book
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public string Author { get; set; }
        public Genres genres { get; set; }
    }

    [Serializable]
    public class Genres
    {
        public string[] values { get; set; }
    }
}