# Oracle Driver

## Overview

* Using `Oracle.DataAccess` driver requires installing an Oracle Client
* Any custom software like the Oracle Client is not going to be available in the root container filesystem
* Oracle offers a portable driver called `Oracle.ManagedDataAccess` or commonly referred to as the "managed oracle driver"
* Using this driver increases the portability of the app and therefore simplifies the installation process

## Install the following package

```
PM> Install-Package Oracle.ManagedDataAccess
```

## Replace all `using` statements

### Do a Find+Replace

* Find
```csharp
using Oracle.DataAccess;
```

* Replace With
```csharp
using Oracle.ManagedDataAccess;
```

## Remove the `Oracle.DataAccess` reference