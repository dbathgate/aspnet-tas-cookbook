# Assembly Errors

## Overview

- Migration patterns typically involve installing or updating packages
- Assembly issues are a common source of frustration during the migration process.

## Binding Redirects Ignored

Sometimes different versions of assemblies are specified by a project and its dependencies.
Visual Studio will attempt to consolidate the conflicting assemblies down to a single assembly version. 
These version redirects are specified in the `app.config` or `web.config` file.  See
[redirect assembly versions](https://learn.microsoft.com/en-us/dotnet/framework/configure-apps/redirect-assembly-versions#specify-assembly-binding-in-configuration-files).

If ignored, the runtime will report attempting to load a different version of the assembly 
than the the version listed in the binding redirects.  Schema problems in the `web.config` file can 
cause an issue where the binding redirects are silently ignored.

One cause of this can be an old schema namespace being specified on the `configuration` tag, e.g.:

```
<configuration xmlns="http://schemas.microsoft.com/.NetConfiguration/v2.0">
```

Removing the `xmlns` attribute resolves the issue.

```
<configuration>
```

## Package version vs Assembly version

Assembly versions appear as 4 digits, e.g. '4.0.0.2'.  NuGet packages appear as three with an optional suffix, e.g. 
'1.2.3-rc'.  They appear in different contexts but sometimes assemblies have the same number as their package,
though not always.  For more details the following links.

- [Package Versioning](https://learn.microsoft.com/en-us/nuget/concepts/package-versioning)
- [Assembly Versioning](https://learn.microsoft.com/en-us/dotnet/standard/assembly/versioning)
