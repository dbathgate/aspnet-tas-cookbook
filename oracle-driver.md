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

## Troubleshooting

### `ORA-01882: timezone region not found`

* This error seems to occur when there is a compatibility between the `Oracle.ManagedDataAccess` version and the Oracle database version.
* Some Internet posts suggest setting [UseHourOffsetForUnsupportedTimezone](https://docs.oracle.com/en/database/oracle/oracle-database/21/odpnt/ConnectionUseHourOffsetForUnsupportedTimezone.html) to `true`. This however resulted in Connection timeouts.
* If that doesn't work, consider lowering the `Oracle.ManagedDataAccess` version down until the issue is resolved. Version `19.18.0` seems to have resolved this in at least one case.