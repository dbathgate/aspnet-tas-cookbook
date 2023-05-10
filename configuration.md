# Configuration

## Overview

* TAS offers features to inject dynamic configuration values into the application
* The benefit to this is enabling engineers to control and swap configurations between environments without having to edit web.config or registry keys
* Use of Registry Keys for configurations will not work on TAS as they cannot be created inside the container

## How

* TAS uses Environment Variables injected at runtime to provide the application context on which configurations to use
* **Services** are available in the TAS marketplace to create and bind to your application
* The two services that enable custom configurations are **User Provided Service** and **CredHub**
* Services are created with configuration values, and bound to an application
* TAS uses an environment variable called `VCAP_SERVICES`, which is a JSON blob containing metadata for each bound service
* The simplest way to consume data in `VCAP_SERVICES` is by adding the nuget package `Steeltoe.Extensions.Configuration.CloudFoundryBase`

## Setup

### Install the following Nuget package

```powershell
PM> Install-Package Steeltoe.Extensions.Configuration.CloudFoundryBase
```


### Create `App_Start\ApplicationConfig.cs`
```csharp
using Microsoft.Extensions.Configuration;
using Steeltoe.Extensions.Configuration.CloudFoundry;

namespace OnboardingApp.App_Start
{
    public class ApplicationConfig
    {
        public static IConfiguration Configuration { get; private set; }

        public static void Configure()
        {
            Configuration = new ConfigurationBuilder()
                .AddCloudFoundry()
                .Build();
        }
    }
}
```

### Update `Global.asax`

```csharp
namespace OnboardingApp
{
    public class Global : HttpApplication
    {
        void Application_Start(object sender, EventArgs e)
        {
            // Code that runs on application startup
            RouteConfig.RegisterRoutes(RouteTable.Routes);
            BundleConfig.RegisterBundles(BundleTable.Bundles);

            // initialize CloudFoundry configuration on start up
            ApplicationConfig.Configure();
        }
    }
}
```

## Usage

### User Provided Service

* User Provided Service is a type of service that allows the user to create a set of key/value configurations
* Any set of string and value can be provided, giving flexibility to add as many configurations as needed
* The below example creates a service containing database credentials (url, username, and password)

#### Creating the User-Provided-Service

```bash
cf create-user-provided-service db-credentials -p '{"url": "db.url", "username": "db-user", "password": "secret123"}'
cf bind-service db-credentials onboardingapp
cf restage onboardingapp
```

#### Accessing User Provided Service values
```csharp
string url = ApplicationConfig.Configuration["vcap:services:user-provided:0:credentials:url"];
string username = ApplicationConfig.Configuration["vcap:services:user-provided:0:credentials:username"];
string password = ApplicationConfig.Configuration["vcap:services:user-provided:0:credentials:password"];
```

#### Updating a User Provided Service
```bash
cf update-user-provided-service db-credentials -p '{"url": "db2.url", "username": "db2-user"}'
cf restage onboardingapp
 ```

### CredHub

* CredHub is similar to a User-Provided-Service in that it stores keys and values
* The difference however is that CredHub configurations are stored at rest encrypted
* CredHub is ideal for sensitive information such as password and encryption keys
* **NOTE**: The one trade off to CredHub is once created you will not be able to see or change the values stored inside
* The below example moves the database password into CredHub

#### Create the CredHub Service

```bash
cf create-service credhub default db-secret -c '{"password": "secret123"}'
cf bind-service onboardingapp db-secret
cf restage onboardingapp
 ```

 #### Access The CredHub values

```csharp
string password = ApplicationConfig.Configuration["vcap:services:credhub:0:credentials:password"];
```

#### Updating the CredHub Service

 ```bash

# first have to unbind and delete the service
cf unbind-service onboardingapp db-secret
cf delete-service db-secret -f

# then recreate the service with the new value and bind it
cf create-service credhub default db-secret -c '{"password": "secret456"}'
cf bind-service onboardingapp db-secret
cf restage onboardingapp
```