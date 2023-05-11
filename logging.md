# Logging

## Overview

* Tanzu Application Service (TAS) consumes and displays all logs written to STDOUT
* The simplest way to write to STDOUT in C# is `Console.WriteLine(..)`
* Leveraging a logging library like `Microsoft.Extensions.Logging` further expands log customizations

## Issue with Event Logs

* TAS cannot display event logs
* Configuring event logs requires write access to the Windows Registry, which is not possible via TAS
* Attempting to write to event logs on TAS will likely fail

### Simple Solution

#### Replace
```csharp
EventLog.WriteEntry("MyAppSource", "My Log Message");
```

#### With

```csharp
Console.WriteLine("My Log Message");
```

### Using a Logging Library

* The logging library `Microsoft.Extensions.Logging` offers the flexibility to write logs to multiple sources
* Sources can include: Console (STDOUT), Event Log, DEBUG (System.Diagnostics.Debug)
* Log message formats can also be customized globally to add more context to logs such as time and thread info

#### Install the following packages
* Base logging library
```
PM> Install-Package Microsoft.Extensions.Logging
```

* Adds ability to write to Console (STDOUT)
```
PM> Install-Package Microsoft.Extensions.Logging.Console
```

* Adds ability to write tto `System.Diagnostics.Debug` to assist with local debugging
```
PM> Install-Package Microsoft.Extensions.Logging.Debug
```
* **Optional**: Adds ability to write to Event Logs for backwards compatibility
```
PM> Install-Package Microsoft.Extensions.Logging.EventLog
```

#### Create `App_Start\LoggingConfig.cs`

```csharp
using Microsoft.Extensions.Logging;
using System.Configuration;

namespace OnboardingApp.App_Start
{
    public class LoggingConfig
    {
        public static ILoggerFactory LoggingFactory { get; private set; }

        public static void Configure()
        {
            LoggingFactory = LoggerFactory.Create(builder =>
            {
                builder.AddConsole();
                builder.AddDebug();
                builder.SetMinimumLevel(LogLevel.Debug);

            });
        }
    }
}
```

#### Add to `Global.asax`
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

            // initialize logging config on app startup
            LoggingConfig.Configure();
        }
    }
}
```


#### **Optional**: Add Event Logging
```csharp
// to feature toggle event logging if needed
if (ConfigurationManager.AppSettings["useEventLog"] == "true")
{
    builder.AddEventLog(b =>
    {
        b.SourceName = "My Source Name";
        b.LogName = "My App Name";
    });
}
```

#### Using the Logger in your code
```csharp
namespace OnboardingApp
{
    public partial class _Default : Page
    {
        protected void Page_Load(object sender, EventArgs e)
        {
            ILogger<_Default> logger = LoggingConfig.LoggingFactory.CreateLogger<_Default>();


            logger.LogInformation("Default Page Loaded");

            try
            {

            } catch(Exception ex)
            {
                logger.LogError(ex, "Something went wrong");
            }
        }
    }
}
```

#### Adding a "Catch-All" log statement to `Global.asax`

* Uncaught exceptions go unlogged and becomes difficult to debug issues in TAS
* Consider added a "catch-all" log statement in `Global.asax` to log any uncaught errors

```csharp
void Application_Error(object sender, EventArgs e)
{
    ILogger<Global> logger = LoggingConfig.LoggingFactory.CreateLogger<Global>();

    logger.LogError(Server.GetLastError(), "Unhandled exception");
}
```