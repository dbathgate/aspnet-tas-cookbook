# ASP.NET Website to Web Application Migration

## Overview

* Some legacy archetypes of ASP.NET are "websites" rather than "web applications"
* Website archetypes are compiled on the fly as opposed to being compiled into a DLL like web applications are
* In modernization, websites can pose challenges with installing Nuget packages and referencing other dependencies
* Microsoft as identified that website projects are obsolete and to not continue developing them [Source](https://learn.microsoft.com/en-us/previous-versions/aspnet/dd547590(v=vs.110))

## How do I know I have a website?
* Contains a `.publishproj` rather than a `.csproj` file
* Has an `App_Code` directory
* ASPX Web Forms have no generated `.aspx.designer.cs`