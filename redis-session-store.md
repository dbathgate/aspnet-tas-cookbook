
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
PM> Install-Package Steeltoe.CloudFoundry.ConnectorBase -Version 2.5.5
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